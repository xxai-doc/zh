# 用户收藏系统 设计笔记

## 技术方案

把用户收藏、取消收藏的行为数据存到 CeresDB , CeresDB 建表的时候注意设置 [enable_ttl=false](https://docs.ceresdb.io/cn/sql/engine_options.html) 避免自动删除数据 。

表结构设计：

```
CREATE TABLE fav (
  ts TIMESTAMP NOT NULL, // 插入的时间戳
  id uint64 NOT NULL,  // 用户行为的时间戳(毫秒)
  uid uint64 NOT NULL,  // 用户 id
  cid uint8 NOT NULL, // 用户行为 : 收藏、取消收藏
  rid uint64 NOT NULL, // 收藏对象的 id
  TIMESTAMP KEY(ts),
  PRIMARY KEY(id)
) ENGINE=Analytic WITH (
  compression='ZSTD',
  enable_ttl=false
);
```

表中会记录用户收藏、取消收藏行为的全量操作日志，类似数据库的 [WAL](https://www.taosdata.com/engineering/6062.html)。

然后前端用 [indexdb](https://www.ruanyifeng.com/blog/2018/07/indexeddb.html) 同步当前这个用户的全量收藏日志，类似 [firebase](https://firebase.google.com/products/realtime-database?hl=zh-cn)、[rxdb](https://rxdb.info)。

然后，在前端基于此数据库判断图片是否已经收藏、展示收藏列表等等。

## 客户端数据的同步

当标签页被选举为领导者的时候（什么是选举见下面），会创建一个 SSE 并同步数据。

同步数据的逻辑是，会记录下上次同步到的最后一条的 client.ts，再次访问，会把 ts 之后本地新增的记录都传给服务器。

如果服务器先查询当前此用户收藏表中 `ts>client.ts` 中的内容，然后再执行插入，同时返回更新内容给这个客户端。

为了避免出现记录的缺失，会返回此用户的总行为行数，如果同步出现行为数不一致的情况，客户端会从 0 开始全量重建缓存，重建缓存，不会删除原来的记录，只会插入新纪录。重建的步骤是 1 同步行为日志，2 清空并重建收藏表。

当某设备产生收藏事件的时候，会通过 SSE ([Server Sent Events](https://caniuse.com/eventsource)) 广播给该用户的其他设备。

广播的时候同样会附带总行数，不一致就重建。

> ceresdb 目前走的方向偏向列存分析 查询的方式主要是 scan 加剪枝 目前没有 tp 型数据库的那些索引 但是底层会建立一些帮助剪枝的索引。

我感觉我还是应该再前面加一个每个用户总行数的内存缓存（用 [dragonfly](https://github.com/dragonflydb/dragonfly) 限制内存上限做个缓存 ）， 缓存过期时间设置为 3 天，过期了就 `select count(1) from fav where uid=123` 来重建。

### 为什么是 SSE 而不是 websocket ？

不选用 websocket 的原因是，cloudflare 代理 websocket 可能比较贵。

cloudflare 代理的 SSE 可能会因长时间没通讯超时，所以客户端注意自动重连。 ( [Are Server-sent events SSE supported, or will they trigger HTTP 524 timeouts](https://community.cloudflare.com/t/are-server-sent-events-sse-supported-or-will-they-trigger-http-524-timeouts/499621/6) )

### 服务器端实现

服务器端用 [nchan](https://www.nginx.com/resources/wiki/modules/Nchan/) 实现 SSE 推送。

会在 redis 中记录下 `user_id - client_id - timestamp` 关系的 zset 映射，基于这个给用户设备实时广播。

[Cloudflare's default connection timeout is 100 seconds](https://support.nine.ch/articles/cloudflare-faqs)

客户端每 90 秒会断开重连一次， 所以超过 100 没更新时间戳的客户端就自动从 zset 中删除掉。

## 浏览器多标签页选举

为了避免每个标签页都搞一个 SSE 长连接浪费服务器资源。

多标签页会进行选举 [tab-election](https://github.com/dabblewriter/tab-election)。

选举成为领导者，才会创建 SSE 链接。

领导者收到了 SSE 的消息，会更加消息类型进行在对应的 [Broadcast Channel](https://developer.mozilla.org/zh-CN/docs/Web/API/Broadcast_Channel_API) 上分发。

消息类型分为：全局消息 (不同类型的全局消息都自己的频道，比如：收藏)、标签页消息（可以用于搜索框的自动补全，每个标签页有自己的专属频道）。

## 设计意图

将来考虑用 [Tauri](https://tauri.app) 封装手机客户端，以及做桌面壁纸客户端，都希望可以支持离线使用。

类似抖音，离线的时候也可以收藏，划过视频等等。（比如地铁上的手机信号不好，也不影响使用）。

所以，设计网站方案的时候，也是都以离线客户端的逻辑来设计开发，未来可以复用代码逻辑。

### 可能进一步的优化

可以做一个前端离线的推荐系统，比如，基于[wonnx](https://github.com/webonnx/wonnx) 。

参考 [快手 - 移动端实时短视频推荐](https://cloud.tencent.com/developer/article/2203171)。

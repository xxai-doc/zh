[‼️]: ✏️README.mdt

<p align="center"><a href="https://xxai.art"><img src="https://cdn.jsdelivr.net/gh/xxai-art/doc/logo.svg"/></a><br/><a href="https://xxai.art"><img src="https://cdn.jsdelivr.net/gh/xxai-art/doc/xxai.svg"/></a></p><p align="center"><a href="https://github.com/xxai-art/doc#readme"><img alt="I18N" src="https://cdn.jsdelivr.net/gh/wactax/img/t.svg"/></a>　<a href="https://groups.google.com/u/0/g/xxai-art"><img alt="Google Groups" src="https://cdn.jsdelivr.net/gh/wactax/img/g-groups.svg"/></a></p>

# xxAI.art

网站代码部分开源，欢迎帮忙优化翻译。

## 序 : xxAI.Art - 献给《黑客与画家》

这是一个变革的时代。

曾经，编程是一门艺术，绘画是另一门艺术。

只有当你站得足够高，才能看到黑客与画家他们的相似之处 -- 内心有股无法抑制的创作冲动。

今天，黑客与画家的灵魂终于可以合二为一，编程和绘画不再是两种截然不同的艺术，她们就是同一种艺术。

科技解放了生产力，打破了人类创作的枷锁，开启了全新的人工智能纪元。

你可以否定人工智能，颂扬抑或是诋毁它。但是唯独不能漠视它。

因为人工智能改变了时代。

于是，我创造了 [xxAI.Art](https://xxAI.Art)，一个分享计算生成艺术的平台 , 献给黑客与画家们，献给这个伟大的时代。

## 前端代码

* [前端代码](https://github.com/xxai-art/web)
* [网站整体的语言包](https://github.com/xxai-art/web/tree/main/i18n)
* [登录模块的语言包](https://github.com/wacpkg/user/tree/main/ui.i18n)
* [网站多语言文档](https://github.com/xxai-doc)

前端的编程语言是 [@w5/coffee_plus](http://npmjs.com/@w5/coffee_plus) ，在 coffeescript 语法的基础上增加了一些特性，参见 [./coffee_plus.md](./coffee_plus.md)。

## 网站、文档国际化

基于以下 3 个项目构建

* [@w5/mdt](https://www.npmjs.com/package/@w5/mdt)

  后缀为 `.mdt`, 可以用类似 `<+ ./coffee_plus/import.js>` 的语法引用外部文件，生成后缀为 `.md` 的 markdown。

* [@w5/trmd](https://www.npmjs.com/package/@w5/trmd)

  markdown 翻译，不会翻译代码和链接，会缓存翻译过的句子，如果修改了译文但没有修改原文，再次执行不会覆盖对译文的修改。

* [@w5/i18n](https://www.npmjs.com/package/@w5/i18n)

  用于翻译 `yaml` 生成网站的语言文件。

### 文档翻译自动化说明

请参见代码库 [xxai-art/doc](https://github.com/xxai-art/doc)

建议先安装 nodejs, [direnv](https://direnv.net), [bun](https://github.com/oven-sh/bun)，然后进入目录后 `direnv allow` ( 进入目录后会自动执行 [.envrc](https://github.com/xxai-art/doc/blob/main/.envrc) )。

为了避免翻译成几百种语言的代码库过大，我把每种语言单独创建了一个代码代码库，并创立一个组织来存放这个代码库

设置环境变量 `GITHUB_ACCESS_TOKEN` 然后运行 [create.github.coffee](https://github.com/xxai-art/doc/blob/main/create.github.coffee) 会自动创建代码库。

当然你也可以放到一个代码库中。

翻译脚本参考 [run.sh](https://github.com/xxai-art/doc/blob/main/run.sh)

脚本代码解读如下：

[bunx](https://bun.sh/docs/cli/bunx) 是 npx 的替代品，更快，当然如果你没有安装 bun，用 `npx` 替代也可以。

`bunx mdt zh` 把 zh 目录下的 `.mdt` 渲染为 `.md`，参见下面 2 个链接的文件

* [coffee_plus.mdt](https://github.com/xxai-doc/zh/blob/main/coffee_plus.mdt)
* [coffee_plus.md](https://github.com/xxai-doc/zh/blob/main/coffee_plus.md)

`bunx i18n` 是翻译的核心代码（如果你只安装了 `nodejs`，没有安装 `bun` 和 `direnv`，也可以运行 `npx i18n` 来翻译）。

它会解析 [i18n.yml](https://github.com/xxai-art/doc/blob/main/i18n.yml) ，本文档的 `i18n.yml` 配置如下：

```
en:
zh: ja ko en
```

含义是 : 中文翻译为日文、韩文、英文，英文翻译为其他所有语种。如果你只想支持中文、英文，可以只写 `zh: en`。

最后是 [gen.README.coffee](https://github.com/xxai-art/doc/blob/main/gen.README.coffee)，它是提取每个语种 `README.md` 大标题 和 第一个子标题 之间的内容，来生成一个入口的 `README.md` 。代码很简单，可以自己看一下。

用到了谷歌 API 来免费翻译。如不能访问谷歌，请配置并设置代理，比如 :

```
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

翻译脚本将在 `.i18n` 目录下生成翻译的缓存，请用 `git status` 查看并将其添加到代码代码库，避免重复翻译。

每次修改译文之后请运行 `bunx i18n`，更新缓存。

如果同时修改了原文和译文，会造成缓存错乱，所以如果要修改，只能修改一个，然后运行 `bunx i18n` 更新缓存。

[‼️]: ✏️README.mdt

<p align="center"><a href="https://xxai.art"><img src="https://cdn.jsdelivr.net/gh/xxai-art/doc/logo.svg"/></a><br/><a href="https://xxai.art"><img src="https://cdn.jsdelivr.net/gh/xxai-art/doc/xxai.svg"/></a></p><p align="center"><a href="https://github.com/xxai-art/doc#readme"><img alt="I18N" src="https://cdn.jsdelivr.net/gh/wactax/img/t.svg"/></a>　<a href="https://groups.google.com/u/0/g/xxai-art"><img alt="Google Groups" src="https://cdn.jsdelivr.net/gh/wactax/img/g-groups.svg"/></a></p>

# xxAI.art

网站代码部分开源，欢迎帮忙优化翻译。

## 前端代码

* [前端代码](https://github.com/xxai-art/web)
* [网站整体的语言包](https://github.com/xxai-art/web/tree/main/i18n)
* [登录模块的语言包](https://github.com/wacpkg/user/tree/main/ui.i18n)
* [网站多语言文档](https://github.com/xxai-doc)

前端的编程语言是 [@w5/coffee_plus](http://npmjs.com/@w5/coffee_plus) ，在 coffeescript 语法的基础上增加了一些特性，参见 [./coffee_plus.md](./coffee_plus.md)。

## 网站、文档国际化

基于以下 3 个项目构建

### [@w5/mdt](https://www.npmjs.com/package/@w5/mdt)

markdown 模板，后缀为 `.mdt`, 可以用类似 `<+ ./coffee_plus/import.js>` 的语法引用外部文件。

[@w5/trmd](https://www.npmjs.com/package/@w5/trmd)

markdown 翻译，不会翻译代码和链接，会缓存翻译过的句子，如果修改了译文但没有修改原文，再次执行不会覆盖对译文的修改。

[@w5/i18n](https://www.npmjs.com/package/@w5/i18n)

用于翻译 `yaml` 生成网站的语言文件。
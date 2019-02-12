# 贡献指南

[Switch to English](CONTRIBUTING.md)

非常感谢您有兴趣为这个翻译项目做贡献，我深感荣幸。在提交您的宝贵意见之前，请务必花点时间阅读以下指南：

- [Code of Conduct](CODE_OF_CONDUCT.md)
- [问题提交指南](#问题提交指南)
- [Pull Request 指南](#Pull Request 指南)
- [格式](#格式)

## 问题提交指南

- 使用模版来创建新的问题。

- 如果添加一些问题讨论或建议
    - 请使用 `Discussion_or_Suggestion` 模版。

- 如果发现了错误或认为译文很有必要改进
    - 请使用 `Bug_or_Improvement` 模版。

## Pull Request 指南

- `master` 分支用于保存最新版本。所有的操作都应该在专门的分支中进行。**不要将任何 PR 提交到 `master` 分支。**

- 目前已经分离出一个主题分支，如：`dev`，所有的操作都在该分支进行，并对该分支进行合并。

- 修复错误或添加任何改进都应该描述理由和依据。理想情况下，您应该先创建一个问题，如果您希望亲自处理，则通过 pr 引用该问题。

- 如果您的 pr 用于解决列表中提出的一个或多个问题，请引用该问题 `(closes #xxxx[,#xxx])`，添加问题的序号，例如：`update Item 19 (closes #1899)`

- 所有内容的更改应符合 [格式](#格式) 的要求。

## 格式
以下内容引用自：[中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)

## 空格

### 中英文之间需要增加空格

正确：

> 在 LeanCloud 上，数据存储是围绕 `AVObject` 进行的。

错误：

> 在LeanCloud上，数据存储是围绕`AVObject`进行的。
>
> 在 LeanCloud上，数据存储是围绕`AVObject` 进行的。

完整的正确用法：

> 在 LeanCloud 上，数据存储是围绕 `AVObject` 进行的。 每个 `AVObject` 都包含了与 JSON 兼容的 key-value 对应的数据。 数据是 schema-free 的，你不需要在每个 `AVObject` 上提前指定存在哪些键，只要直接设定对应的 key-value 即可。

例外：「豆瓣FM」等产品名词，按照官方所定义的格式书写。

### 中文与数字之间需要增加空格

正确：

> 今天出去买菜花了 5000 元。

错误：

> 今天出去买菜花了 5000元。
>
> 今天出去买菜花了5000元。

### 数字与单位之间需要增加空格

正确：

> 我家的光纤入屋宽频有 10 Gbps，SSD 一共有 20 TB

错误：

> 我家的光纤入屋宽频有 10Gbps，SSD 一共有 20TB

例外：度 / 百分比与数字之间不需要增加空格：

正确：

> 今天是 233° 的高温。
>
> 新 MacBook Pro 有 15% 的 CPU 性能提升。

错误：

> 今天是 233 ° 的高温。
>
> 新 MacBook Pro 有 15 % 的 CPU 性能提升。

### 全角标点与其他字符之间不加空格

正确：

> 刚刚买了一部 iPhone，好开心！

错误：

> 刚刚买了一部 iPhone ，好开心！
>
> 刚刚买了一部 iPhone， 好开心！

## 标点符号

### 不重复使用标点符号

正确：

> 德国队竟然战胜了巴西队！
>
> 她竟然对你说「喵」？！

错误：

> 德国队竟然战胜了巴西队！！
>
> 德国队竟然战胜了巴西队！！！！！！！！
>
> 她竟然对你说「喵」？？！！
>
> 她竟然对你说「喵」？！？！？？！！

## 全角和半角

不明白什么是全角（全形）与半角（半形）符号？请查看维基百科词条『[全形和半形](http://zh.wikipedia.org/wiki/%E5%85%A8%E5%BD%A2%E5%92%8C%E5%8D%8A%E5%BD%A2)』。

### 使用全角中文标点

正确：

> 嗨！你知道嘛？今天前台的小妹跟我说「喵」了哎！
>
> 核磁共振成像（NMRI）是什么原理都不知道？JFGI！

错误：

> 嗨! 你知道嘛? 今天前台的小妹跟我说 "喵" 了哎！
>
> 嗨!你知道嘛?今天前台的小妹跟我说"喵"了哎！
>
> 核磁共振成像 (NMRI) 是什么原理都不知道? JFGI!
>
> 核磁共振成像(NMRI)是什么原理都不知道?JFGI!

### 数字使用半角字符

正确：

> 这件蛋糕只卖 1000 元。

错误：

> 这件蛋糕只卖 １０００ 元。

例外：在设计稿、宣传海报中如出现极少量数字的情形时，为方便文字对齐，是可以使用全形数字的。

### 遇到完整的英文整句、特殊名词，其内容使用半角标点

正确：

> 贾伯斯那句话是怎么说的？「Stay hungry, stay foolish.」
>
> 推荐你阅读《Hackers & Painters: Big Ideas from the Computer Age》，非常的有趣。

错误：

> 贾伯斯那句话是怎么说的？「Stay hungry，stay foolish。」
>
> 推荐你阅读《Hackers＆Painters：Big Ideas from the Computer Age》，非常的有趣。

## 名词

### 专有名词使用正确的大小写

大小写相关用法原属于英文书写范畴，不属于本 wiki 讨论内容，在这里只对部分易错用法进行简述。

正确：

> 使用 GitHub 登录
>
> 我们的客户有 GitHub、Foursquare、Microsoft Corporation、Google、Facebook, Inc.。

错误：

> 使用 github 登录
>
> 使用 GITHUB 登录
>
> 使用 Github 登录
>
> 使用 gitHub 登录
>
> 使用 gｲんĤЦ8 登录
>
> 我们的客户有 github、foursquare、microsoft corporation、google、facebook, inc.。
>
> 我们的客户有 GITHUB、FOURSQUARE、MICROSOFT CORPORATION、GOOGLE、FACEBOOK, INC.。
>
> 我们的客户有 Github、FourSquare、MicroSoft Corporation、Google、FaceBook, Inc.。
>
> 我们的客户有 gitHub、fourSquare、microSoft Corporation、google、faceBook, Inc.。
>
> 我们的客户有 gｲんĤЦ8、ｷouЯƧquﾑгє、๓เςг๏ร๏Ŧt ς๏гק๏гคtเ๏ภn、900913、ƒ4ᄃëв๏๏к, IПᄃ.。

注意：当网页中需要配合整体视觉风格而出现全部大写／小写的情形，HTML 中请使用标淮的大小写规范进行书写；并通过 `text-transform: uppercase;`／`text-transform: lowercase;` 对表现形式进行定义。

### 不要使用不地道的缩写

正确：

> 我们需要一位熟悉 JavaScript、HTML5，至少理解一种框架（如 Backbone.js、AngularJS、React 等）的前端开发者。

错误：

> 我们需要一位熟悉 Js、h5，至少理解一种框架（如 backbone、angular、RJS 等）的 FED。

### 链接之间增加空格

正确：

> 请 [提交一个 issue](#) 并分配给相关同事。
>
> 访问我们网站的最新动态，请 [点击这里](#) 进行订阅！

错误：

> 请[提交一个 issue](#)并分配给相关同事。
>
> 访问我们网站的最新动态，请[点击这里](#)进行订阅！

### 简体中文使用直角引号

正确：

> 「老师，『有条不紊』的『紊』是什么意思？」

错误：

> “老师，‘有条不紊’的‘紊’是什么意思？”

## 其他注意事项

### 泛型的正确标注
由于 markdown 支持 html 标签，泛型的尖括号如果直接使用，会误判为 html 标签，导致解析为正文时无法显示。因此，应将泛型放到单行代码（\`  \`）中。正确的使用，如：`Comparable<CaseInsensitiveString>`

### 代码或方法的正确标注
原文在正文中提及的代码在译文中保持半角标点符号，并放到单行代码中。如：`sgn(x.compareTo(y)) == -sgn(y. compareTo(x))`、`BigDecimal("1.0")`，等等。

### 代码缩进
代码块缩进采用 4 个空白。

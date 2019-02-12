# Contributing Guide

[切换到简体中文](CONTRIBUTING_zh.md)

I’m really excited that you are interested in contributing to this project. Before submitting your contribution though, please make sure to take a moment and read through the following guidelines.

- [Code of Conduct](CODE_OF_CONDUCT.md)
- [Issue Reporting Guidelines](#issue-reporting-guidelines)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Format Guidelines](#format-guidelines)

## Issue Reporting Guidelines

- Always use an template to create new issues.

- If adding new discussion or suggestion:
    - PLS use the `Discussion_or_Suggestion` template.

- If finding a bug or adding new improvement:
    - PLS use the `Bug_or_Improvement` template.

## Pull Request Guidelines

- The `master` branch is basically just a snapshot of the latest release. All operation should be done in dedicated branches. **Do not submit PRs against the `master` branch.**

- Checkout a topic branch from the relevant branch, e.g. `dev`, and merge back against that branch.

- Provide convincing reason to fixing a bug or add this improvement. Ideally you should open a issue first and have it greenlighted before working on it.

- If you are resolving a issue, add `(closes #xxxx[,#xxx])` (#xxxx is the issue id) in your PR title for a better release log, e.g. `update Item 19 (closes #1899)`.

- All updated content should be formatted in accordance with [Format Guidelines](#format-guidelines) requirements.

## Format Guidelines
The following is a quote from: [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)

## Spacing

### Place one space before/after English words

Good:

> 在 LeanCloud 上，数据存储是围绕 `AVObject` 进行的。

Bad:

> 在LeanCloud上，数据存储是围绕`AVObject`进行的。
>
> 在 LeanCloud上，数据存储是围绕`AVObject` 进行的。

An example of complete and correct usage:

> 在 LeanCloud 上，数据存储是围绕 `AVObject` 进行的。 每个 `AVObject` 都包含了与 JSON 兼容的 key-value 对应的数据。 数据是 schema-free 的，你不需要在每个 `AVObject` 上提前指定存在哪些键，只要直接设定对应的 key-value 即可。

Exceptions: For product and brand names, please refer to the writing format of the official definition. For example, use “豆瓣FM” instead of “豆瓣 FM”.

### Place one space before/after numbers

Good:

> 今天出去买菜花了 5000 元。

Bad:

> 今天出去买菜花了 5000元。
>
> 今天出去买菜花了5000元。

### Place one space between numbers and units

Good:

> 我家的光纤入屋宽频有 10 Gbps，SSD 一共有 20 TB

Bad:

> 我家的光纤入屋宽频有 10Gbps，SSD 一共有 20TB

Exceptions: There should not be any spacing between numbers and degrees/percentages.

Good:

> 今天是 233° 的高温。
>
> 新 MacBook Pro 有 15% 的 CPU 性能提升。

Bad:

> 今天是 233 ° 的高温。
>
> 新 MacBook Pro 有 15 % 的 CPU 性能提升。

### No additional spaces before/after punctuation in fullwidth form

Good:

> 刚刚买了一部 iPhone，好开心！

Bad:

> 刚刚买了一部 iPhone ，好开心！
>
> 刚刚买了一部 iPhone， 好开心！

## Punctuation

### Avoid duplicate punctuation

Good:

> 德国队竟然战胜了巴西队！
>
> 她竟然对你说「喵」？！

Bad:

> 德国队竟然战胜了巴西队！！
>
> 德国队竟然战胜了巴西队！！！！！！！！
>
> 她竟然对你说「喵」？？！！
>
> 她竟然对你说「喵」？！？！？？！！

## Fullwidth and halfwidth

If you’re not familiar with fullwidth and halfwidth forms please refer to [Halfwidth and fullwidth](http://zh.wikipedia.org/wiki/%E5%85%A8%E5%BD%A2%E5%92%8C%E5%8D%8A%E5%BD%A2) forms on Wikipedia.

### Use punctuation in fullwidth form

Good:

> 嗨！你知道嘛？今天前台的小妹跟我说「喵」了哎！
>
> 核磁共振成像（NMRI）是什么原理都不知道？JFGI！

Bad:

> 嗨! 你知道嘛? 今天前台的小妹跟我说 "喵" 了哎！
>
> 嗨!你知道嘛?今天前台的小妹跟我说"喵"了哎！
>
> 核磁共振成像 (NMRI) 是什么原理都不知道? JFGI!
>
> 核磁共振成像(NMRI)是什么原理都不知道?JFGI!

### Use numbers in halfwidth form

Good:

> 这件蛋糕只卖 1000 元。

Bad:

> 这件蛋糕只卖 １０００ 元。

Exceptions: fullwidth numbers are acceptable for better visual alignment in graphic design.

### Use punctuation in halfwidth form for English sentences

Good:

> 贾伯斯那句话是怎么说的？「Stay hungry, stay foolish.」
>
> 推荐你阅读《Hackers & Painters: Big Ideas from the Computer Age》，非常的有趣。

Bad:

> 贾伯斯那句话是怎么说的？「Stay hungry，stay foolish。」
>
> 推荐你阅读《Hackers＆Painters：Big Ideas from the Computer Age》，非常的有趣。

## Nouns

### Proper nouns are properly capitalized

Uppercase and lowercase are used in English and are not discussed in this wiki. Here is a brief overview of the error-prone usage.

Good:

> 使用 GitHub 登录
>
> 我们的客户有 GitHub、Foursquare、Microsoft Corporation、Google、Facebook, Inc.。

Bad:

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

### Avoid jargons

Good:

> 我们需要一位熟悉 JavaScript、HTML5，至少理解一种框架（如 Backbone.js、AngularJS、React 等）的前端开发者。

Bad:

> 我们需要一位熟悉 Js、h5，至少理解一种框架（如 backbone、angular、RJS 等）的 FED。

### Add extra spaces before/after links

Good:

> 请 [提交一个 issue](#) 并分配给相关同事。
>
> 访问我们网站的最新动态，请 [点击这里](#) 进行订阅！

Bad:

> 请[提交一个 issue](#)并分配给相关同事。
>
> 访问我们网站的最新动态，请[点击这里](#)进行订阅！

### Use corner brackets for Chinese Simplified

Good:

> 「老师，『有条不紊』的『紊』是什么意思？」

Bad:

> “老师，‘有条不紊’的‘紊’是什么意思？”

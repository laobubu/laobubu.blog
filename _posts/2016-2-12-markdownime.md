---
layout: post
title:  "MarkdownIME 一个酷炫的 web 富文本编辑器"
date:   2016-02-12
categories: 
 - 发布
 - 前端
permalink: /archives/markdownime
published: true
excerpt_separator: <!--more-->
---

![MarkdownIME](https://laobubu.github.io/MarkdownIME/demo.gif)

[Markdown](https://zh.wikipedia.org/wiki/Markdown) 是一种轻量级标记语言。使用它写的文档即使不经过渲染，也具有极强的可读性，而且语法极其简单，输入起来也特别方便。现有的的大多数 Markdown 编辑器需要手动转换，抑或是将半边屏幕用于多余的预览！

作为一个酷炫的 Markdown 式富文本编辑器，[MarkdownIME](http://laobubu.github.io/MarkdownIME) 将这两个步骤合并成了一个，你的输入会**实时地**转换为你所想要的渲染结果！

<!--more-->

## 感受新世界

该程序运行于浏览器，无任何依赖，只需要一个 js 文件即可。

 - **在线演示 & 文档**： <http://laobubu.github.io/MarkdownIME>
 - **源代码**： <https://github.com/laobubu/MarkdownIME>
 - **问题反馈**： [submit issue](https://github.com/laobubu/MarkdownIME/issues/new)

## 使用说明

 - 没有任何按钮，要插入链接、粗体字、列表等玩意儿时不需要动鼠标。
 - 直接输入 Markdown 内容即可。
 - 多按两次回车就能结束列表、引用文本段、代码块、表格等块状元素。

## 碎碎念

### idea 与造轮子？

这个 idea 是在使用 OneNote 记录东西时突然想到的，因为微软那鬼畜的界面要点来点去实在太烦人了。

与 Typora 的关系只是功能相似而已。

此作品的[雏形](https://github.com/laobubu/MarkdownIME/tree/draft)最早发布于 V2EX，地址为 <http://www.v2ex.com/t/241963>，那个时候 Typora 还没有出 Windows 版本，因此我没有什么机会去感受，不知道你们这些使用 Mac 的城里人是怎么玩儿的。反正就是，没有任何山寨的意思！

有意思的是后来 Typora 还是推出了 Windows 版本，我测试了一下，效果还不错，但是有点儿 bug，而且感觉启动速度很慢（没有 SSD 的我能为各位水深火热中的学生党带盐）。

### 反正没人看

本来还想使用 Electron 之类的玩意儿，把这个 MarkdownIME 做成电脑软件的，但是懒得动了……

反正估计没有人会看代码的，于是就写得乱七八糟了。你可以看到乱七八糟的 `Utils.ts`，山寨安卓的 `Toast.makeToast(str, ele).show()`，以及一大堆会被吐槽死的鬼畜风格。如果有活人看了代码想吐嘈，没关系， [submit issue](https://github.com/laobubu/MarkdownIME/issues/new)!

说到没人看，好像还真的没几个人看这个 project 呢……我不懂怎么让更多人看到这个项目。希望各位看了以后回去宣传宣传，到时候这里就人山人海了，这是坠吼的！

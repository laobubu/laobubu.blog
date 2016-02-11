---
layout: post
title:  "Typecho插件 RewriteRule"
date:   2014-6-24 19:14:00
categories: ["发布","程序实验室"]
permalink: /archives/typecho-rewriterule/
excerpt_separator: <!--more-->
---

## RewriteRule for Typecho 是啥

本着有新东西就瞎鼓捣鼓捣的坏习惯，拿 Typecho 的 HelloWorld 插件的代码改了一个 RewriteRule 出来。至于功能嘛：

> **在遇到 404 的时候按照你设定的匹配规则，跳转到其他的地址上。**
采用正则表达式来匹配 URI，并可以在目标 URI 里使用正则表达式里匹配到的项。
如果懒得动 .htaccess 文件来设定跳转的话，快用这个插件，妥妥的！

## 下载它

从 GitHub 下载，戳这里： https://github.com/laobubu/Typecho_rewriterule/archive/master.zip

<!--more-->

## Typecho 和超廉价 VPS

在超廉价玩具 VPS 上玩了一下 Typecho，有一种清爽淡雅的感觉，而且里面结构真的很有意思（[GitHub](https://github.com/typecho/typecho)）。受不了 WordPress 臃肿的现状的可以考虑看看。这个博客系统1MB不到，但是除了一个博客必须的功能外也支持插件和主题等，而且支持 sqlite 等格式数据库，也支持各种国内 BAE、SAE 等云平台。[插件库][1]里东西虽然还不多，但是相信这个优秀的平台（和里面很赞的编码）一定会吸引来大量开发者。

话说这个VPS六块钱一个月，ping 值在人品不好的时候 400+ms，不过基本凑合了（主要是送多个IP）。听 @OwOCat 同学讲，临近网关的地方这玩意儿速度飞快……

就是不知道是否稳定，反正那个工具我都丢上面了。

  [1]: http://docs.typecho.org/plugins/download "Typecho Wiki &raquo; 插件列表及下载"
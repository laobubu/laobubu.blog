---
layout: post
title:  "博客来到 Jekyll 了"
date:   2014-12-31 01:10:47
categories: 
permalink: /archivers/hello
excerpt_separator: <!--more-->
---

好吧，正如你所见，我的老博客的东西都被糊到这里丢着了。数据还在同步，等过后完全同步结束就可以正式上线了。

博客将挂在 GitHub 上，因此文本什么的都会是开源的。

主题是自己基于 Jekyll 修改的，此外也对 Jekyll 做了简单的修改，譬如

<!--more-->

* 支持 GFM 标记语言特色（比如代码块、删除线之类的）
* 增加翻页
* 将英文修改为中文
* 将页面底部的社交网络图标进行重排
* 添加留言板功能
 * 在 `_includes/comment.html` 文件里面修改评论插件
 * 如果要关闭评论，在文章头部加上 `nocomments: true` 即可
* ……

如果要快速搭建一个的话，可以 clone 我的模板Repo，具体见： https://github.com/laobubu/jekyll-theme-EasyBook

还有就是从 WordPress 导入数据，我用的是自己编写的一个 NodeJS 程序，可以在这里拿到代码： https://gist.github.com/laobubu/d48aeb6d35913ccf1fce

天色已晚，该睡觉了。晚安。下周还有考试呢。
---
layout: post
title:  "使 CSS 自动适配不同设备"
date:   2014-7-2 22:29:00
categories: ["拿程序说话", "前端"]
permalink: /archives/css-device-adapt/
---

我做网页一般就是在 Dreamweaver 里面拿它的所见即所得特性做，而且还经常拿表格来布局。不得不说感觉还是很赞呢，在大屏幕上可以较好的随着窗口变化而变化，问题遇到手机就坑了。一个页面要左滑右滑很痛苦有木有？因此，我需要特别的适配技巧，不是去匹配 User-Agent。

趁着还没有看到 more 处，先把基础的两行代码送上来……如果看到这里秒懂或基本够用的话，就不必点开看全文了吧。

```css
@viewport {
  zoom: 1.0;
  width: extend-to-zoom 100%;
}
@media screen and (max-width: 800px){ 
  #main { }
}
```

<!--more-->

## 视口（viewport）

就是很多网站里 `<meta name="viewport" content="initial-scale=1.0, width=device-width" />` 标签里那个玩意儿。

viewport 就是浏览器窗口中内容显示那一块区域（含右边滚动条）。首先是网络上已经玩滥了的这个表：

| 属性 | 说明 | 值
|:-----|:-----|:-----
|width|viewport的宽度。|纯数字，或者 `device-width`
|height|类似上面，指定高度。|纯数字，或者 `device-height`
|initial-scale|初始缩放比例。| 类似 `1.0`
|maximum-scale|允许用户缩放到的最大比例。| 同上
|minimum-scale|允许用户缩放到的最小比例。| 同上
|user-scalable|用户是否可以手动缩放。|`yes`/`no`/`0`/`1`

如果你单独指定 `width` 为某个数值的话（例如200），浏览器会放大显示的内容，使得 viewport 宽度刚好就是200px。

如果你的设计就是基于视口宽度为固定值的情况，那么你会发现，适配从未如此轻松……浏览器把内容直接放大（或缩小），使得视口宽度为你设定的值。就像下面多个“移动设备”，虽然屏幕宽度不一样，但是内容都被缩放得刚好塞满了屏幕…

![various device example](http://i671.photobucket.com/albums/vv73/laobubu/pics%20for%20blog/css-device-adapt-fig1.jpg)



## 在CSS里风骚地拿下 viewport

当然风骚的 CSS 一定不会把这种样式设计的活儿忘记。在 W3.org 上就有这么一篇鸟语文章[^w3adapt]，提到了 `@viewport` 这种玩意儿，也就是本文开头那个玩意。当然，用的就不是同一套了：

属性 |值	|默认值
:---|:---|:---
height  |(viewport长度) {1,2}|
width   |(viewport长度) {1,2}|
min-height|(viewport长度)   |auto
min-width|(viewport长度)   |auto
max-height|(viewport长度)   |auto
max-width|(viewport长度)   |auto
zoom|auto，数字或百分比|auto
max-zoom|auto，数字或百分比|auto
min-zoom|auto，数字或百分比|auto
orientation|auto，`portrait`，`landscape`   |auto
user-zoom|`zoom`，`fixed`|`zoom`

### viewport长度、width与height

上表中提到的 `viewport长度` 可以…

* 用 `auto` 敷衍过去
* 像平时那样写（例如`800px`、`100vw`和`100vh`，后两个表示viewport宽度或高度的100%）
* 使用百分比（参照的是viewport初始状态的宽度或高度）

至于表中 width 和 height 提到的 `{1,2}` ，意思是有两种写法：

* 一个长度值，例如 `100vw`。这将**同时**改变 min-xxxxx 和 max-xxxxx 。
* 两个长度值，例如 `800px 1400px`。这将**分别**赋予 min-xxxxx 和 max-xxxxx 。

## 媒体查询

好的！万恶之源来了！这个玩意儿有 w3.org 中文版维基文档[^w3query]，所以本文为了省流量就简单说说我个人认为常用的东西。

媒体查询能自动在不同情况下使用不同的样式。首先它在 CSS 里可以像这样出现…

* `@import url(laobubu_net.css) screen and (color);`
* `@media screen and (color), projection and (max-width: 800px) { ... }`

然后你可能用得到的部分媒体特性，完整版可以访问 [CSS3媒体查询&raquo;媒体特性](http://www.w3.org/html/ig/zh/wiki/CSS3%E5%AA%92%E4%BD%93%E6%9F%A5%E8%AF%A2#.E5.AA.92.E4.BD.93.E7.89.B9.E6.80.A7) 查阅。

| 名字 | 内容 | 注释
|:--   |:--   |:--
|device-width 和 device-height | 屏幕尺寸（或纸张尺寸） | 支持 min- 和 max- 前缀
|width 和 height | 视口尺寸（含滚动条） | 同上
|device-aspect-ratio| 设备屏幕宽高比（例如`16/9`）  | 同上
|aspect-ratio| 视口宽高比  | 同上
|orientation| `portrait` 或 `landscape` | 只要视口height大于width，就取`portrait`

至于屏幕宽度啥的，如果你用 Chrome 浏览器，可以使用“审查元素”里的Emulation（就是左上角那个手机图标），具体自己感受吧。

顺带补充一个，貌似 WP7 上的 IE 对媒体查询支持性真心蛋疼，所以你可以考虑……

```html
<!--[if IEMobile 7]>
   <link rel="stylesheet" type="text/css" href="images/wp.css" media="screen">
<![endif]-->
```

然后接下来的活计就看你自己了。


[^w3adapt]: **CSS Device Adaptation** http://dev.w3.org/csswg/css-device-adapt

[^w3query]: **CSS3媒体查询** http://www.w3.org/html/ig/zh/wiki/CSS3%E5%AA%92%E4%BD%93%E6%9F%A5%E8%AF%A2
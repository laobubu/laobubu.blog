---
layout: post
title:  "旧 Internet Explorer 的私人订制"
date:   2014-7-7 14:52:00
categories: ["拿程序说话", "前端"]
permalink: /archives/old-ie-adapt/
---

好吧，这是一个悲伤的问题。兼容不同浏览器，尤其 IE 一直都是一个叫人痛苦的活计。本文介绍若干针对IE老版本特别优化的事情，原本是[《使 CSS 自动适配不同设备》](/archives/css-device-adapt/)里的一部分，但是感觉太长了，所以单独拿出。本文也不完整，如果我又遇到一些奇葩问题会再添加的。此外，感谢某学长的帮助。

## DOCTYPE，专治盒模型和各种疑难杂症

<center>![W3C盒模型和IE盒模型](http://i671.photobucket.com/albums/vv73/laobubu/pics%20for%20blog/Box_module.jpg)</center>

大量用到 `padding` 和 `margin` 的时候必备此代码，否则在 IE 里渲染会有点奇怪（如上图，早期乃至近期 IE 都有盒模型缺陷[^ieboxbug]）。

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

使用这一行代码可以解决 IE 的坑爹 Quirks 模式的诸多问题，例如上面的盒模型，还有 `:hover` 之类的高端玩意儿。如果不爽那个 doctype 声明的类型，[这里还有一些丰富的选择](http://www.w3schools.com/tags/tag_doctype.asp)……

当然，如果你要偷懒，可以简单粗暴地试试 `<!doctype html>`

## CSS Hack

利用 CSS Hack 能在某些属性值上区分对待 IE（例如遇到不支持 `position: fixed` 的情况）。具体用法就是在属性名字前面加一个奇怪的符号，就像这样：

```css
    .ie6    {_color:#FF0000}
    .ie6_7  {+color:#FF0000}
    .ie6_7_8{.color:#FF0000}
```

虽然会造成不同浏览器体验不一样，但总比毁成一片好多了。（注意，如果加了DOCTYPE，可能这一招就失效了，具体看下面这个测试）

<iframe width="100%" height="100" src="http://fiddle.jshell.net/t3h2V/show/light/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## 条件注释与 comment 标签

comment 标签是微软特色，而且从 IE8 以后微软也把它抛弃了<comment>，**而且你现在用的这个浏览器也不支持**</comment>。具体效果就是被`<comment>`包围的内容不会有效而已。（试试换个浏览器看本文）

这玩意儿我真没有用过，因为它实际上就是套了个马甲的注释：

> 
&lt;!-- [if IE]>
&lt;p>laobubu.net 不支持 IE 浏览器。请考虑换一个。&lt;/p>
&lt;![endif]-->
> 
&lt;!-- [if lt IE 9]>
&lt;p>我的天啊，至少升级成 IE9 或更高版本再来吧。&lt;/p>
&lt;![endif]-->

## 强制最新渲染引擎，或者 Chrome Frame

如果你运气比较好，遇到了带有最新渲染引擎的 IE，或者那个 IE 用户有 Chrome Frame，那么加上这一句无疑会让你爽到飞起来。

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
```

当然这也是一句标准的 HTTP 响应头，如果你用的是 Apache 服务器，可以考虑编辑 `.htaccess` 文件：

```
<IfModule mod_setenvif.c>
  <IfModule mod_headers.c>
    BrowserMatch MSIE ie
    Header set X-UA-Compatible "IE=Edge" env=ie
    BrowserMatch chromeframe gcf
    Header append X-UA-Compatible "chrome=1" env=gcf
  </IfModule>
</IfModule>
```

### gov 网站独享：强制使用老版本渲染引擎

我知道这个有点奇葩，但是很多 gov 网站在新版 IE 渲染引擎下各种掉渣。因此，如果你遇到一个豆腐渣网站，可以考虑加上这句代码。

```html
<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE6" />
```


[^ieboxbug]: **IE盒模型缺陷** http://zh.wikipedia.org/wiki/IE%E7%9B%92%E6%A8%A1%E5%9E%8B%E7%BC%BA%E9%99%B7
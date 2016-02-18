---
layout: post
title:  "Web 富文本编辑器与空行的事"
date:   2016-02-18
categories: 
 - 前端
 - 拿程序说话
permalink: /archivers/web-richedit-empty-line
excerpt_separator: <!--more-->
---

本文就简单记录一点儿在开发 [MarkdownIME] 时遇到的有关空行的坑。

在现在的浏览器中，空行还必须需要加一点儿东西才能渲染出来，比如一个空格 `&nbsp;` 或者一个 `<br>`。但是，在 IE 10 以及以前的版本中，`<p>` 标签不需要包含任何内容就可以创建一个空行；包含了 `<br>` 反而会产生两个空行。

在浏览器的“contentEditable”容器里创建空行时，会自动产生一个 `<br>` 元素，然后在接着输入文字的时候，会把那个元素给自动地删除。然而，自己实现移动光标（cursor, aka. caret）到空行时，可能会产生奇怪的现象。

<!--more-->

## 创建空行

虽然 `<p></p>` 这种写法没有错误，但是不被推荐，浏览器甚至可能会无视它[^1]。

在 TinyMCE 中，空行会自带一个 `<br data-mce-bogus="true">` 标签；在 [MarkdownIME] 中则是 `<br data-mdime-bogus="true">`。

如果要手动创建空行，同时照顾 IE9 和 IE10，记得判断以后再插入那个占位玩意儿。

```javascript
if (/MSIE (9|10)\./.test(navigator.appVersion)) {
  insertBr(newLine);
}
```

## 移动光标到空行中

在通过 [Selection API] 移动光标到空行时，光标需要移动到那个 `<br>` 前面，否则那个玩意儿不会被删除，新的文字节点会跟在它后面。也就是说，使用 [Range.collapse] 设定的新坐标来开始输入，可能会神奇地出现一个空行。

**方案一**：我行我素，直接使用 [Range.collapse] 产生新光标。这是最偷懒的方法，选中整个空行的内容，然后收起选区，就得到一个光标了。Chrome 这样产生的新光标，在用户输入的时候不会产生问题。但是在 Firefox 中，那个占位用的 `<br>` 标签不会消失。

**方案二**：拒绝 `<br>`。你可以使用其他的东西，例如一个空格，来使得一个空行看得见。但是浏览器自动创建的空行都是使用 `<br>` 的。

**方案三**：定位到那一只 `<br>` 上，把光标挪到它的前面。这样在多个浏览器下，那个多余的玩意儿会被自动删除。然而这个要实现起来会比较麻烦。

结局就是选择了方案三，也就是产生了 [MarkdownIME@958e61ea] 中的那一段代码。

[MarkdownIME]: https://github.com/laobubu/MarkdownIME
[MarkdownIME@958e61ea]: https://github.com/laobubu/MarkdownIME/commit/958e61ea
[Selection API]: https://www.w3.org/TR/selection-api/
[Range.collapse]: https://developer.mozilla.org/en-US/docs/Web/API/Range/collapse

[^1]: **Using empty P elements is discouraged.** <https://www.w3.org/TR/REC-html40/struct/text.html#edef-P>

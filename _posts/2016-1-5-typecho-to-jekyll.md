---
layout: post
title:  "从 Typecho 搬迁数据到 Jekyll 和多说"
date:   2016-01-05
categories: 
permalink: /archives/typecho-to-jekyll
---

我发现虽然使用了 Typecho 可以相对自由地管理很多东西，但是实际上并没有太大的意义——根本没什么要修改的，就连去服务器管理文件都要鼓捣半天（[我的 Typecho 是挂载在 OpenShift 上的](/archives/move-to-openshift/)）。所以为了节约世界资源，我把 Typecho 的帖子和回复都移动到了 GitHub Pages 里。


![Jekyll](http://jekyllrb.com/img/logo-2x.png)

[搬迁用的代码](https://gist.github.com/laobubu/a8efd66e4ec7076e7f26)是用 Node.js 写的，使用方法为：

1. 安装依赖 `npm install dateformat` 。
2. 将数据库里的表 `typecho_comments` 和 `typecho_contents` 分别导出为 json 文件。可以使用 PHPMysqlAdmin 来做，点两下鼠标就好。
3. 编辑我的这个脚本，尤其是第 15 - 20 行。
4. 运行这个脚本 `node t2jd.js` 。

其输出的一个多说 JSON 文件可以[在多说后台导入](http://laobubu.duoshuo.com/admin/tools/import/)。如果不了解多说的 ThreadKey 怎么改，建议配合[我的这个 Jekyll 主题](https://github.com/laobubu/jekyll-theme-EasyBook)直接使用。

2016年1月5日：这个版本还不能把文章分类给一起导出来，如果谁有兴趣的话欢迎 Fork 和修改。

{% gist a8efd66e4ec7076e7f26 %}
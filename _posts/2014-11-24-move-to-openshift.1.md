---
layout: post
title:  "记一次搬迁到 OpenShift 并搭建 PHP5.5 环境等"
date:   2014-11-24
categories: ["后端"]
permalink: /archives/move-to-openshift/
---

十一月，忙碌到飞起来的二十多天中，我使用的廉价VPS主机商 Incero 没钱，任性，跑路了，接着我的网站直接挂彩。本来打算使用 DigitalOcean 的学生优惠去购买VPS，谁知他们不接受中国的邮箱后缀。无奈之下我又滚回了经典的 OpenShift。

OpenShift 是由红帽公司提供的 PaaS，当然可大致视为虚拟主机使用，只是一个账号只能创建三个 app （可以当做站点），而且官方提供的 PHP 版本截至目前最高是 5.4。对于我这种可能需要多来几个子域名，而且想体验最新版本的人来说有点遗憾。

不过 OpenShift 还很厚道地提供 DIY Cartridge，也就是提供一个类似 VPS 的存在，可以用来编译和运行自己的网页服务器程序。网络上有PHP 5.5 + Nginx 的 DIY Cartridge 模板[^opn]，但是我还是想自己试着闹腾一个 PHP 5.5 + OpenShift httpd (Apache2.2) 的版本……

我的 DIY Cartridge 模板已经放到 GitHub 上了：[laobubu/openshift-php5.5-cgi-apache](https://github.com/laobubu/openshift-php5.5-cgi-apache)，不定期更新。
使用方法和前面所提的那个类似，上传到服务器，在服务器上运行编译脚本，等它编译一小时，完成。
由于以前也没有接触过这种服务器方面的操作，使用的是简陋的 CGI 方式，但是能够摆脱官方的限制，可以自定义 `httpd.conf` 和 `php.ini` 也是很不错的。

<!--more-->

## New OpenShift DIY Cartridge

官方介绍就是 Do-It-Yourself ，允许自己闹腾服务器程序，只要你的 web 服务器程序监听的地址是 `${OPENSHIFT_DIY_IP}:${OPENSHIFT_DIY_PORT}`，爱怎么玩就怎么玩。

刚刚创建一个基于 DIY Cartridge 的 App 时，官方会在里面提供一行的 Ruby 简易服务器示例，但是可以删除。

### 操作钩子（action hooks）

有一点很重要的就是修改 `.openshift/action_hooks` 里面的脚本，从而让 OpenShift 知道怎么启动和停止服务器程序。官方也提供了[一些说明](http://openshift.github.io/documentation/oo_user_guide.html#action-hooks)。我就将里面的 `start` 和 `stop` 修改成了启动和停止 httpd 的脚本，感觉就像 init.d 一样萌萌哒。

### 文件和程序管理

一个 app 的程序是保存在 git 上的。通过 `git push/pull` 来控制代码，一大股 PaaS 范儿。

除此之外也可以通过 SSH 直接连接到服务器，也支持 SFTP 管理文件。当前版本的程序就保存在 `~/app-root/runtime/repo` 里面，可以随便修改，而且立即见效（顺带吐槽一下 AppFog 那诡异的服务器，随便一修改，刷新就没了）。问题就是这个不是长久之计，因为只要一 `git push` ，所有的改动都会被 git 洗去。

当然也有不受 git 支配的文件夹，那就是 `~/app-root/runtime/data` 甚至 `~/app-root/runtime`。可以把那些需要持久的数据放在这些文件夹。

说了这么多，大概可以这么想：将 PHP5.5 编译出来并放在 `~/app-root/runtime` 里面，然后 httpd 的配置文件就用 git 来控制，挺顺溜的。

## 编译 PHP 和准备 httpd

这个基本就是参考那个 PHP+Nginx[^opn] 的做法，糊了一大堆的编译参数，同时指定好 prefix 使之输出到那个 runtime 文件夹下面。

然后是 httpd 的准备。摆脱了官方的限制，可以自己编写操作钩子，指定配置文件地启动 httpd。而 httpd.conf 文件还得从头开始，不过天下 Apache 基本也是那个样子，找一个例子，按需修改即可。

哦对了别忘记 `Listen ${OPENSHIFT_DIY_IP}:${OPENSHIFT_DIY_PORT}`

## 搬家

这个过程比较简单了。在 OpenShift 后台给这个 app 绑定了域名之后，修改 httpd.conf 添加 VirtualHost 即可实现多站点。把文件上传上去，一切都进行的很顺溜。

## 为 Typecho 更换一个 Markdown 解析引擎

不知道咋的，Typecho 1.0 的 Markdown 解析引擎不支持脚注、表格等功能了，这使我感到纠结。之前看 QQ 群他们也吐槽过这个问题，但是都手工更换引擎了，不知道官方怎么想的……

大概方法就是：

下载 [ParseDown](https://github.com/erusev/parsedown) 和 [ParseDown-Extra](https://github.com/erusev/parsedown-extra) 的解析引擎程序，放到 `var` 文件夹内。然后修改 `var/Markdown.php` 文件，将里面的 `Markdown::convert($text)` 函数代码修改掉：

```php
    public static function convert($text)
    {
        static $docParser;

        if (empty($docParser)) {
            $docParser = new ParsedownExtra();
        }
        
        return $docParser->text($text);
    }
```

测试发现 ParseDown-Extra 对 HTML 标签围绕的 Markdown 是不解析的，所以需要修改程序。

找到 `getAttribute('markdown')` 这一行，将下面的 `!= '1'` 修改为 `== '0'` 即可。

完成。如果你看到这篇文章有一个角标，那么恭喜我，成功了。

## 后面的事情

这个月还有一大串电子设计的事情，等后面闲下来再在博客扯淡吧。

[^opn]: **OpenShift PHP5.5 Nginx**: https://github.com/rexdf/openshift-nginx-php55/
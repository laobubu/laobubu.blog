---
layout: post
title:  "Google 语音识别 API (Part 2)"
date:   2016-03-11
categories: 
 - 互联网资源
published: true
excerpt_separator: <!--more-->
---

在 2011 年的时候发过一篇[《\[API\]Google的语音识别API，支持各种语言》](/546)，然而现在 Google 已经把那个 API 给关闭了。昨天收到读者邮件询问，便再提一下那个 API 现在的情况吧。

一句话概括：现在仅限 Chrome 内部私自使用了。如果可以的话，考虑使用其他家（比如科大讯飞）的吧。

<!--more-->

## 熟悉的一次性识别 API

当时的 API 地址为 `http://www.google.com/speech-api/v1/recognize?xjerr=1&client=chromium`，有趣的是现在仍然存在一个类似的 API：

```
https://www.google.com/speech-api/v2/recognize?lang=en-us&key=APIKEY
```

### 请求方式

还是和以前一样的，这个 API 使用 `POST` 方式请求。在设置了 `Content-Type` 请求头之后，将音频文件的文件内容作为 Request Body 即可。而其他的参数，还是直接加到网址上的。

| 参数 | 说明 | 例子 |
|:-----|:-----|:-----|
|key   | **(必填)** APIKEY，见下文|  |
|lang  | **(必填)** 识别的语言|`zh-CN`|
|app   |请求方，但是这个参数没啥用|`app`、`chromium`等|
|xhw   |硬件配置情况，好像也没啥用|  |
|maxresults| 最多返回多少条识别结果 | `10` |

一个请求的例子

```
POST /speech-api/v2/recognize?key=你的APIKEY内容&lang=en-US HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.87 Safari/537.36
Content-Type: audio/x-flac; rate=16000;

(flac 文件的内容)
```

一个返回的例子

```json
{
  "result":
  [
    {
      "alternative":
      [
        {"transcript":"hello world","confidence":0.96691877},
        {"transcript":"helloworld"}
      ],
      "final":true
    }
  ],
  "result_index":0
}
```

还是类似以前，会返回一个 JSON，内容如上。通过读取这个对象的 `result[0].alternative` 数组即可得到识别的结果。

然而不知道为什么，有时候它会返回类似如下这样的多行结果。此时只需要选一行有效的 JSON 来解析即可。

```
{"result":[]}
{"result":[{"alternative":[{"transcript":"hello world","confidence":0.96691877},{"transcript":"helloworld"}],"final":true}],"result_index":0}

```

### APIKEY

在上面的网址中的 `APIKEY` 即为你访问这个 API 所需要的“钥匙”。Chromium 的项目网站中 [Chromium API Keys] 页面就提到了这个。以下为免费申请的方法：

1. 成为 Chromium 开发者，[订阅 chromium-dev 这个论坛](https://groups.google.com/a/chromium.org/forum/?fromgroups#!forum/chromium-dev)。
2. 进入 <https://cloud.google.com/console>，在 API Console 里启用 [Speech API](https://console.developers.google.com/apis/api/speech/overview) 。
3. 去 [API 凭据](https://console.developers.google.com/apis/credentials) 里自己创建 API Key 使用。

#### 使用次数限制

要注意的是，这个 API 每天只有 50 次使用权限。毕竟，这个 API Key 只是给你自娱自乐以及自己编译专属 Chromium 时候用的。

> Speech API:
>
> **It is NOT possible to get additional quota for the Speech API.**
> 需要提高每日配额？**想都不要想**！

[Chromium API Keys]: (http://www.chromium.org/developers/how-tos/api-keys)

#### 来自 Chrome 中的 APIKEY

在 [Chromium API Keys] 中提到了，可以在编译的时候指定一个 APIKEY，这也正是 Google 在开源的浏览器里使用私有的服务的一种手段；语音识别功能仅限他们自己发布的 Chrome （而不是 Chromium ）可用。

有趣的是，通过抓包（或者逆向其可执行文件）可以发现其 APIKEY。比如说，我的 49.0.2623.87 m (Windows 64-bit) 版本中，APIKEY 就是 `AIzaSyBOti4mM-6x9WDnZIjIeyEU21OpBXqWBgw`。

曾经 Google 的第一代 API 是没有要求 Key 的，后来玩坏的人多了，也便有了这种限制使用量的方法。当一个 APIKEY 被玩坏时，在下一个版本中再换一个就可以了。

### 请求的内容

和以前相似，采样率不是很高的 flac 都支持。只需要在请求头里指定 `Content-Type: audio/x-flac; rate=16000;`

要生成支持的 flac 文件有许多的方法，比如使用[以前那篇博客](/546)提到的 `flac` 工具，或者使用[万能的 ffmpeg](https://ffmpeg.org/download.html)。在命令行运行 `ffmpeg -i 原始文件 -ar 16000 输出.flac`

此外，还[支持原始的音频数据](https://github.com/gillesdemey/google-speech-v2)。比如 16 位单通道有符号型的数据，对应请求头 `Content-Type: audio/l16; rate=16000;`

## 流式(Stream)识别

顾名思义，就是边说边识别。之前那个 API 只支持十秒以内的音频，如果拆成一大堆地识别会有点蛋疼，所以有了这个 API。

这个功能要使用起来还挺繁琐的，这里就简单提一下，具体的实现参考 Chromium 源代码中的 [google_streaming_remote_engine.cc](https://code.google.com/p/chromium/codesearch#chromium/src/content/browser/speech/google_streaming_remote_engine.cc) 文件。

### API 接口

#### 下行接口

```
GET https://www.google.com/speech-api/full-duplex/v1/down?key=XXXXX&pair=XXXXX&output=pb
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.87 Safari/537.36
```

用于拉取识别结果。需要以下的全部网址参数：

| 参数 | 说明 | 例子 |
|:-----|:-----|:-----|
|key   | APIKEY|  |
|pair  | 请求编号，见下文| |
|output| 输出格式，目前仅支持一个 | `pb` |

#### 上行接口

```
POST https://www.google.com/speech-api/full-duplex/v1/up?key=XXXXX&pair=XXXXX&output=pb&lang=en-US&pFilter=2&maxAlternatives=1&app=chromium&continuous&interim
Content-Type: audio/x-flac; rate=16000
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.87 Safari/537.36

(flac文件内容)
```

用于发送语音文件内容。需要以下网址参数：

| 参数 | 说明 | 例子 |
|:-----|:-----|:-----|
|key   | APIKEY|  |
|pair  | 请求编号，见下文| |
|output| 输出格式，目前仅支持一个 | `pb` |
|lang  | 语言 | `en-US` |
|pFilter| 是否启用脏话过滤器 | `2` 表示启用， `0` 表示禁用 |
|maxAlternatives| （可选）最多可选结果数量 | `1` |
|app | | `chromium` |
|continuous 或者 endpoint| 一般只需要一个 `continuous` 即可 | `1` |
|interim|||

### 识别过程

在启动的时候，会先生成一个 `pair`（请求编号），然后创建两个HTTP连接到那俩 API 接口。一个连接发送语音内容，另外一个接收识别结果。

请求编号只要是一个类似 `ECC7EB46ED4FF017` 这种的十六进制随机内容即可。Chromium 内部的生成方法就是产生一个 64 位的随机数来用的。

在上行端口发送的数据被识别出来的时候，下行端口就会收到识别结果。具体格式懒得去探索了，先这样吧。参考：

 - 源代码 [google_streaming_remote_engine.cc](https://code.google.com/p/chromium/codesearch#chromium/src/content/browser/speech/google_streaming_remote_engine.cc)
 - 一个基于 PHP 的实现 <http://mikepultz.com/2013/07/google-speech-api-full-duplex-php-version/>

## 后记

由于 Google 删除了老的 API 以及我的那个 App Engine 实例，所以以前那个自动转换格式的 API （<http://glab.laobubu.net/stt/>）已经不提供服务了。将来也不会再提供服务。

现在的 Google 已经收紧了许多的东西，反正感觉没有以前开放了。也或者是因为现在的 Web 已经不如以前干净了吧。不管怎样，有些东西该改变的还是必然会改变。

本文只是一个回复（顺便刷存在感）而已，如果要做语音识别，建议使用其他家公司的服务。

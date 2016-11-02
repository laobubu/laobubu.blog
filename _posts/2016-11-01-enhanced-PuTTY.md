---
layout: post
title:  "能自动选串口，能拖拽传文件，魔改版 PuTTY"
date:   2016-11-01
categories: 
 - 发布
permalink: /archives/enhanced-PuTTY
published: true
excerpt_separator: <!--more-->
---

[![putty-enh1.jpg](https://ooo.0o0.ooo/2016/11/01/5818650e03742.jpg)](https://laobubu.net/PuTTY)

最近在玩某个神奇的全志 H3 板子（类似树莓派的玩意儿），用串口直接去~~调试~~玩。
然而我的廉价串口每次插入电脑，显示的 COM 号都不一样，导致每次使用前都要开设备管理器，甚是不爽。
于是就有了这个玩意儿…

[项目网站](https://laobubu.net/PuTTY)  /  [立即下载](https://github.com/laobubu/PuTTY/releases/)  /  [GitHub点个赞](https://github.com/laobubu/PuTTY)

<!--more-->

## 功能列表

### 串口识别 (feature-serial-port-list)

- 列出所有可用的串口。
- (Windows) 直接连接 `COM`（不带串口编号）时，PuTTY 会自动搜索可用的串口。

### 拖拽上传文件 (feature-drag-n-drop)

**未完全实现！** 此功能未正式发布，你可以[点击这里下载尝鲜版](https://ci.appveyor.com/project/laobubu/putty/build/1.0.12/artifacts)。

截至 2016-11-1，仅实现了以下功能：

- 可拖拽上传文件。
- 提供两种模式上传文件：
  - 使用 ZModem 上传文件（需自备 `win32-lrzsz` 的 `sz.exe` ）
  - 使用 Shell + base64 上传文件（默认选项。技术细节见下文，速度很慢，但是几乎支持全部 Unix 目标）
  - 在选项窗口中可选择

以下功能即将到来：

- 使用 ZModem 接收文件
- 发送文件前/后执行远程命令  （目前设置窗口的这一项仅仅是摆设）
- Linux 下可用   （没错，PuTTY 也能编译成 Linux 版本）

## 技术细节

### 使用 Shell + xxd 上传文件

不知道为什么 Windows 的 lrzsz 在我电脑上一直用不顺，总是会卡死，或者说 CRC ERROR。
于是有了这个很奇葩的文件传输方式。

这个方式不需要任何额外安装软件，它只需要系统自带的 `xxd` 和 Shell 的一些指令。
即使是 `sh` 这种上古玩意儿也支持。

假设文件名是 `something`，在传输开始前，会自动输入并执行如下指令：

```bash
stty -echo            # 隐藏输入回显，否则发送的数据会被 Shell 传回来，浪费流量
AZps1=$PS1; PS1='.';  # 备份一些环境变量，下同
AZhc1=$HISTCONTROL; HISTCONTROL=ignorespace;

rm -rf something

AZupl () {           # 定义一个函数，接受1个参数（Cycle tag）。
  printf "\r@@@";    # 告诉 PuTTY 可以开始接受输入
  read a;            # 此时 PuTTY 会模拟输入 0e3f8c250933 类似的字符串
  (echo "$a" |xxd -r -p >> something);  # 将输入的Hex转换，并追加写入到文件
  echo -n "~~~$1";   # 告诉 PuTTY 这一轮（Cycle）数据接收完毕
}
```

接下来就是开始发送数据了。
Cycle tag 表示这是第几波数据，这个 tag 是可打印的字符（字母数字符号等）。

如果开了回显，会看到 PuTTY 一直在像这样子模拟用户的输入。
实际上就是调用之前那个函数，输入数据并追加到文件末尾。

```bash
AZupl ' '
E4BDA0E5B185E784B6E69C89E5BF83E68385E8A7A3E7A081E
AZupl '!'
79C8BE8BF99E4B8AAE69687E4BBB6EFBC8CE5A5BDE58E89E5
AZupl '"'
AEB3EFBC810D0A0D0AE4BD86E698AFE5A5BDE5838FE6B2A1E
AZupl '#'
595A5E794A8E6ACB8E38082E6B5B7E79B97E79A84E5AE9DE7
AZupl '$'
AEB1E4B88DE59CA8E8BF99E9878CE380820D0A0D0A2D2D20E
AZupl '%'
5908CE6A0B7E8A2ABE9AA97E79A84206C616F627562750D0A
```

发送文件结束后，执行以下指令，恢复现场

```bash
PS1=$AZps1
HISTCONTROL=$AZhc1
printf "\r"
stty echo
```

至于为什么不使用 `xxd -r -p >something <<EOF` 这种简单的方式…

因为不知道咋的，会爆，无论是本地的发送缓冲区，还是对面的 Shell。

PuTTY 是单线程的，要实现这个功能还额外加了一个线程，于是各种多线程的坑就出现了，还额外实现了一个 subthd 模块。

### 使用 Base64 上传

使用 Base64 的话，冗余更低，速度更快。只是需要不停地摇晃鼠标，促使 mainloop 发送文件，晚些时候修正。 -- 2016年11月2日

---
title: Linux 与 Android 同步剪贴板的通用方案
date: '2019/01/12 12:01:23'
updated: '2019/01/12 12:01:23'
categories: 实验室
tags:
  - Clipboard
  - 剪贴板
thumbnail: 'https://ae01.alicdn.com/kf/HTB1fShdaUvrK1RjSspc762zSXXaE.png'
abbrlink: d691e748
---

按照惯例，还是在每篇文章的开头扯几句不相关的：元旦前就从学校回家了，在家的十几天过得很是舒坦：闲的时候，上午玩两小时游戏或者看看直播，下午就看会 Coursera，晚上有时间就看部电影，没整段的空余时间就找朋友聊聊天、刷刷 V2EX；当然在不闲的时候也是做了一些事的，就比如本文将要说到的——让移动平台和桌面平台同步剪贴板的方案。

<!-- more -->

我为什么想到做这个呢：两周前，我把用了一年半的 Manjaro 格式化了（忍受不了 KDE 巨多的 Bug，还有硬盘都被我用完了），装上了 Ubuntu，桌面环境选择了 Budgie——一个新出的 DE。比 GNOME 漂亮，比 KDE 稳定，稍加配置即可满足我这个强迫症的审美需求：

![桌面截图](https://ae01.alicdn.com/kf/HTB15xNmaOzxK1RjSspj763S.pXaS.png)

从 KDE 转成 Budgie 之后，最让我不习惯的就是手机和电脑再也不能愉快地共享剪贴板和文件了，当然我也在网上找了一些现成的解决方案，但体验都不佳，于是我便考虑自己造一个轮子。

## 设计思路

由于移动端（客户端）、桌面端（服务端）二者需要进行数据的双向传输，HTTP 协议肯定是无法做到了：于是选用了 WebSocket 作为底层的传输协议。同时，为了让适用性更广，桌面端开发选择了 Golang；移动端则使用 React Native 开发。但这两门高级的语言（Golang、JavaScript）都无法为剪贴板添加监听事件，于是我只好自己用轮询的方式对比当前剪贴板的内容与上一次内容的差异，再考虑是否发送数据。

## 桌面端

桌面端的开发语言选择 Golang 其实我是有些不情愿的：

1. 太过高级：无法提供系统底层的 API（比如监控剪贴板）；
2. 语法太过丑陋：我想把 log 信息封装一下，结果用 struct 封装了半天，代码反而看起来更「💩」了，还不如用 Switch，可 Switch 导致暴露出的接口又不够简洁 . . .

其它语言我也找了个遍：Python 虽可监听剪贴板的变化（通过 gi 这个库），但这个库并没有办法跨平台，且我 Windows 没有安装编程环境，我也不想安装。

总之，能监听剪贴板的无法跨平台，能跨平台的无法监听剪贴板。

So . . . 哪怕 Go 有万般不是，但在跨平台这一点的易用性上也足以让我抛弃其它的所有（纯静态链接库 + 交叉编译）。

于是乎，

「真香」。

## 移动端

其实我本想用 Flutter 来开发的，但遇到了一些问题：

1. 在我电脑上无法热加载，这在修改样式的时候可太难受了；
2. Dart 似乎无法在已有的 WebSocket 连接上绑定一个新的连接，即使原来的连接已经失效了（不知是 Bug 还是咋样）。

于是我选择了 React Native。有了 React 的基础，上手确实很快，并且 JavaScript 写起来感觉还是挺爽的。

之所以会有刚刚提到的（重新绑定 WebSocket 的）需求，是因为在不同的网络环境中，电脑的 IP 可能有所改变，而我暂时想不到一个方法让手机自动识别同一网络的哪一台电脑使用了共享剪贴板的工具（逐一扫描 IP？那也太丑陋了）。

所以在初始化连接时需要手动输入一次电脑的 IP 地址，随后地址信息会被保存，之后就不用再输入了。

## 使用

只需保持这两个软件在后台，会自动监控剪贴板的内容并发送。

![桌面端](https://ae01.alicdn.com/kf/HTB1nepnaIfrK1RkSnb4760HRFXaf.png)

![移动端](https://ae01.alicdn.com/kf/HTB1dj0maN_rK1RkHFqDq6yJAFXam.jpg)

## 下载

由于代码写的有点不满意，功能也不太完善，等以后有空重构之后加上文件共享功能，会开源的。这里先放出各平台的可执行程序：[戳我👈](https://bit.ly/clipboard_shared)。

---

愿能有所帮助。
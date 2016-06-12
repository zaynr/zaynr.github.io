---
title: Android Imapge DPI
date: 2016-05-29 15:53:09
tags:
---
## 安卓 ImageView 的默认 DPI 问题
因为最近在弄 Excited BBS，在设置开始画面的时候发现渲染的图像分辨率超过 OpenGl 的上限(2400*4800), 搜索一番之后发现是图片的默认 DPI 问题，在
```bash
@drawable
```
里面，有 HDPI, XHDPI 等等子文件夹，里面就是不同的 DPI 设置，此时我们就把所需要的图片放在
```bash
NODPI
```
里面，这样 ImageView 渲染的 DPI 就不会乱跑了。
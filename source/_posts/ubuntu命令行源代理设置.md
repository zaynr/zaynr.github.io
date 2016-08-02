---
title: ubuntu命令行源代理设置
date: 2016-08-02 22:09:53
tags:
---
使用科学上网工具时,因为在 linux 下不能直接在 shadowsocks 的 gui 里直接设置全局代理,需要在浏览器中添加代理插件走 localhost:1080,这样有点麻烦.在 Ubuntu 里的 Terminal 里如果要用代理来翻墙 apt-get, 可以使用:
```bash
http_proxy=http://10.1.3.1:8080 apt-get update
```
来用代理进行下载.
在文件
> /etc/environment

中,是存储系统变量的地方,可以在这里设置其他的东西.[ubuntu 环境变量设置](https://help.ubuntu.com/community/EnvironmentVariables)之后即可.
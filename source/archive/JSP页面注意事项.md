---
title: JSP页面注意事项
date: 2017-07-11 17:32:05
tags:
---
在开发网站时，可以用jsp来生成需要进行交互的页面，此时需要在JSP开头添加以下两行代码：
```java
<%@page isELIgnored="false"%>
<%@page pageEncoding="UTF-8"%>
```
第一行表示启用 EL 表达式，比如在 Spring 中利用 model.addAttribute 来让 Java 与 JSP 交互，JSP 内的 ${name} 就会被作为一个表达式对待，如果不加上第一行就会被当成文本。第二行是解决由于 JSP 默认编码导致的网页中文乱码。
另外，如果需要使用 JavaScript 则需要将 js 文件放在 webapp 目录下而不是 WEB-INF 下，例如当前页面 test.jsp 的文件层次为 /webapp/WEB-INF/views/test.jsp，需要引用在 /webapp/js/ 下的 JavaScript 文件，则如此引用：`<script type="text/javascript" src="../../jsfile/jquery-3.2.1.js" />`。

---
title: servlet学习笔记
date: 2016-08-11 10:44:19
tags: [java, web, servlet]
category: [学习笔记]
---
现在想开始上手网页开发，才发现之前都没接触过那些基础的东西。没办法，从头开始吧。
<!--more-->
## 什么是 servlet
---
servlet 是一个 Java 的类, 继承自 HttpServlet 类, 在服务端运行用以处理客户端的请求. 其实就是一个 server 和 database 的中间层.
## servlet 生命周期
---
servlet 的一个生命周期实例:
> 1. 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。
> 2. Servlet 容器在调用 service() 方法之前加载 Servlet。
> 3. 然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。

所以, servlet 大体的生命周期为:
> 1. 调用 init() 进行初始化.
> 2. 调用 service() 方法来处理客户端请求.
> 3. 调用 destroy() 结束.
> 4. 由 JVM 进行垃圾回收.

### init() 方法
该方法被设计为只能调用一次, 用于一次性的初始化. servlet 创建于用户第一次调用对应于该 servlet 对应的 URL 时, 用户可自定义让 servlet 在服务器启动时即初始化. 用户第一次调用 servlet 时就会创建一个实例, 每一个用户请求都会产生一个新的线程, 适当的时候移交给 doGet 或 doPost 方法. init 方法简单地创建并初始化一些数据, 存在于整个 servlet 的生命周期.
```java
public void init() throws ServletException {
  // 初始化代码...
}
```
### service() 方法
service() 方法是执行实际任务的主要方法. Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求, 并把格式化的响应写回给客户端. 
每次服务器接收到一个 Servlet 请求时, 服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型(GET、POST、PUT、DELETE 等), 并在适当的时候调用 doGet, doPost, doPut, doDelete 等方法。
下面是该方法的特征
```java
public void service(ServletRequest request, 
                    ServletResponse response) 
      throws ServletException, IOException{
}
```
service() 方法由容器调用, service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法. 所以不用对 service() 方法做任何动作, 只需要根据来自客户端的请求类型来重载 doGet() 或 doPost() 即可.
doGet() 和 doPost() 方法是每次服务请求中最常用的方法. 下面是这两种方法的特征.
#### doGet() 方法
GET 请求来自于一个 URL 的正常请求, 或者来自于一个未指定 METHOD 的 HTML 表单, 它由 doGet() 方法处理.
```java
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```
#### doPost() 方法
POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单, 它由 doPost() 方法处理.
```java
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```
### destroy() 方法
destroy() 方法只会被调用一次, 在 Servlet 生命周期结束时被调用. destroy() 方法可以让 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动.
在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。destroy 方法定义如下所示：
```java
  public void destroy() {
    // 终止化代码...
  }
```
## 实例
---
### Hello World!
servlet 的 HelloWorld.java
```java
// 导入必需的 java 库
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;

// 扩展 HttpServlet 类
public class HelloWorld extends HttpServlet {
 
  private String message;

  public void init() throws ServletException
  {
      // 执行必需的初始化
      message = "Hello World";
  }

  public void doGet(HttpServletRequest request,
                    HttpServletResponse response)
            throws ServletException, IOException
  {
      // 设置响应内容类型
      response.setContentType("text/html");

      // 实际的逻辑是在这里
      PrintWriter out = response.getWriter();
      out.println("<h1>" + message + "</h1>");
  }
  
  public void destroy()
  {
      // 什么也不做
  }
}
```
生成 HelloWorld.class 后放入 webapps/ROOT/WEB-INF/classes. 
并在 WEB-INF 目录下的 web.xml 添加如下内容:
```xml
<web-app>
    ...
    <servlet>
        <servlet-name>HelloWorld</servlet-name>
        <servlet-class>HelloWorld</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloWorld</servlet-name>
        <url-pattern>/HelloWorld</url-pattern>
    </servlet-mapping>
    ...
</web-app>
```
即可运行该 demo.
### POST & GET & TABLES
//TODO
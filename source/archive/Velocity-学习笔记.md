---
title: Velocity 学习笔记
date: 2016-08-22 19:32:20
tags: [Velocity, Java]
category: [学习笔记]
---
Velocity 是一个类似于 Jsp 的语言, 可以在布局文件中嵌入 HTML 标签来绘制前端. 并且还能执行一些简单的脚本. 文件类型为 `*.vm` .
<!--more-->
## 注释
---
Velocity 的注释分为两种, 单行注释以 `##` 打头, 多行注释为 `#* blablabla *#`, 也就是用 `#` 代替了 `/`.
## 引用
---
Velocity 的引用分为三种: 变量, 属性, 方法. 有关引用的所有参数都被处理为了 String. 而显式的引用方式为加大括号, 例如 ${foo}. 还有一个引用方式为 "安静引用", 即让变量初始值为空, 例子为 `$!foo` 或 `$!{foo}`. 而如果碰到需要使用和变量名相同的字符串时, 可以用反斜杠来转义, 例如 `\foo`.
### 变量
Velocity 的变量使用其实就是引用, 从而将动态内容嵌入页面中. VTL, 即为"Velocity 模板语言".
```php
#set( $foo = "Velocity" )
```
这就是一条典型的 VTL 语句. 对于 Velocity 来说, 所有的变量类型都为 String. 此处变量名为 foo, 而值被赋予之后, 即可使用了:
```php
<html>
<body>
#set( $foo = "Velocity" )
Hello $foo World!
</body>
<html>
```
有一点要注意的是, VTL 的变量名必须以字母开头.
### 属性
因为 VTL 是与 Java 交互的, 很显然, 既然变量是引用对象, 自然就有属性这一说法, 例如 `$Employee.name`. 属性包括对象里的数据和方法.
### 方法
也就是函数啦, 下面是一些实例:
```php
$customer.getAddress()
$purchase.getTotal()
$page.setTitle( "My Home Page" )
$person.setAttributes( ["Strange", "Weird", "Excited"] )
```
传参方式有点像 Json.
## 与 Java 交互
---
VTL 使用的数据类型也是 JavaBean, 下面是一个实例:
```php
$foo
 
$foo.getBar()
## is the same as
$foo.Bar
 
$data.getUser("jon")
## is the same as
$data.User("jon")
 
$data.getRequest().getServerName()
## is the same as
$data.Request.ServerName
## is the same as
${data.Request.ServerName}
```
而从 Java 向 Velocity 传参时, 则使用 ModelMap:
```java
@RequestMapping("/test/test")
	public void test(ModelMap modelMap){
		test = new ConnectTestDTO();
		test.setRemark("test");
		modelMap.put("foo", test);
	}
```
此时则向 model 中放入了键为 foo, 值为 test 的 ConnectTestDTO 对象.
## 指令
---
VTL 里的指令也就相当于是内置函数了, 以下是几个常见指令:
### #set
没什么可说的, 也就是给变量赋值, 下面是一些实例:
```php
#set( $monkey = $bill ) ## variable reference
#set( $monkey.Friend = "monica" ) ## string literal
#set( $monkey.Blame = $whitehouse.Leak ) ## property reference
#set( $monkey.Plan = $spindoctor.weave($web) ) ## method reference
#set( $monkey.Number = 123 ) ##number literal
#set( $monkey.Say = ["Not", $my, "fault"] ) ## ArrayList

#set( $value = $foo + 1 )
#set( $value = $bar - 1 )
#set( $value = $foo * $bar )
#set( $value = $foo / $bar )
```
有一个很坑的地方是, 如果赋值语句的右值为空(null), 而左值之前已经被赋了其他值, 此时赋值语句并不会让左值的数据改变, 依然是原来的值(妈的智障). 当右值被双引号包围时, 会解析里面的变量名, 而单引号则直接引用, 如下:
```php
#set( $directoryRoot = "www" )
#set( $templateName = "index.vm" )
#set( $template = "$directoryRoot/$templateName" )
$template
#set( $foo = "bar" )
$foo
#set( $blargh = '$foo' )
$blargh

output:
## template
www/index.vm
## foo 
Bar
## blargh
$foo
```
### #if, #foreach
就跟字面意思一样, 一个是条件判断, 一个是循环. 和 #set 不同, 这俩都需要用 `#end` 来标示代码段的结束. 当然, 在条件判断处也支持逻辑符号使用.
```php
#set( $criteria = ["name", "address"] )
 
#foreach( $criterion in $criteria )
 
    #set( $result = false )
    #set( $result = $query.criteria($criterion) )
 
    #if( some_condition && other_condition || !some_condition )
        Query was successful
    #elseif( some_else_condition ) ## don't mind, this is just a demo, lol.
        blablabla
    #else
        blablabla
    #end
 
#end
```
### #include, #parse, #stop
#include 就是来引入文件的啦, 可以引入多个文件, like this: `#include( "greetings.txt", $seasonalstock )`, 而 #parse 则只能有一个参数, like this: `#parse( "me.vm" )`. #parse 的用处嘛, 就是宏展开啊.
```php
Count down.
#set( $count = 8 )
#parse( "parsefoo.vm" )
All done with dofoo.vm!
```
parsefoo.vm:
```php
$count
#set( $count = $count - 1 )
#if( $count > 0 )
    #parse( "parsefoo.vm" )
#else
    All done with parsefoo.vm!
#end
```
#stop 则是调试用的(Oooo~, 在这停顿!).
### #macro
类似于汇编的宏, 跟 #parse 差不多, 一个是引用文件外的, 一个是引用文件内的. 代码段结尾需要 #end 标示.
```php
#macro( d )
<tr><td></td></tr>
#end

#macro( tablerows $color $somelist )
#foreach( $something in $somelist )
    <tr><td bgcolor=$color>$something</td></tr>
#end
#end

## call d:
#d()

## call tablerows:
#set( $greatlakes = ["Superior","Michigan","Huron","Erie","Ontario"] )
#set( $color = "blue" )
<table>
    #tablerows( $color $greatlakes )
</table>

```

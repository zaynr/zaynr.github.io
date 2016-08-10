---
title: Spring MVC 学习笔记
date: 2016-08-09 20:20:38
tags: [web, java]
---
其实我现在学习的 Spring MVC 并不是原生的，而是公司魔改版本的 JRES 系列框架。里面的坑之多简直丧心病狂。首先就是配置工程。公司里使用的 IDE 为 Eclipse LUNA，JDK 版本为 1.7。
<!--more-->
## 初始化工程
工程的初始化选择为 Maven Project，不过在创立工程前应进入 Window->Preference->Maven->User Setting 里面更改 Maven 的 repo 下载地址。
```xml
<settings>
	<localRepository>C:\Users\zengzy19585\.m2\repository</localRepository>
  <mirrors>
    <mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://repos.hundsun.com:6060/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>hundsun-central-repos</id>
          <url>http://repos.hundsun.com:6060/nexus/content/groups/public</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>hundsun-central-repos</id>
          <url>http://repos.hundsun.com:6060/nexus/content/groups/public</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
      </profile>
  </profiles>
  <activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
  <servers>
     <server>
       <id>nexus</id>
       <username>readOnly</username>
       <password>nexus@0831</password>
     </server>
    <server> 
      <id>hsrisk-ftp</id> 
      <username>mvn</username> 
      <password>123456</password> 
    </server> 
   </servers>
   <!--
   <proxies>
     <proxy>
       <id>my-proxy</id>
       <active>true</active>
       <protocol>http</protocol>
       <host>192.168.190.126</host>
       <port>3128</port>
     </proxy>
   </proxies>
   -->
</settings>
```
如此设置好 Maven 的设置之后才能从公司的 repo 获取魔改版本的 Spring 框架。之后需要设置 Maven 工程的依赖，即 pom.xml 设置为：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>zt.demo</groupId>
  <artifactId>demo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
	<properties>
		<jresplus-remoting.version>1.1.15</jresplus-remoting.version>
		<jresplus-mvc.version>1.0.10</jresplus-mvc.version>
		<jresplus-ui.version>1.0.28</jresplus-ui.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<artifactId>slf4j-api</artifactId>
			<groupId>org.slf4j</groupId>
			<version>1.5.6</version>
		</dependency>
		<dependency>
			<groupId>com.hundsun.jresplus</groupId>
			<artifactId>jresplus-mvc</artifactId>
			<version>${jresplus-mvc.version}</version>
			<exclusions>
				<exclusion>
					<artifactId>fastjson</artifactId>
					<groupId>com.alibaba</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.hundsun.jresplus</groupId>
			<artifactId>jresplus-remoting</artifactId>
			<version>${jresplus-remoting.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.10.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>3.10-FINAL</version>
		</dependency>

		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.4.4</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>2.4.4</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-annotations</artifactId>
			<version>2.4.4</version>
		</dependency>
		<!--拼音处理 -->
		<dependency>
			<groupId>com.belerweb</groupId>
			<artifactId>pinyin4j</artifactId>
			<version>2.5.0</version>
		</dependency>

		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>

		<dependency>
			<groupId>commons-configuration</groupId>
			<artifactId>commons-configuration</artifactId>
			<version>1.10</version>
		</dependency>
		<dependency>
			<groupId>org.jdom</groupId>
			<artifactId>jdom</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
		    <groupId>com.alibaba</groupId>
		    <artifactId>fastjson</artifactId>
		    <version>1.2.11</version>
		</dependency>
		<dependency>
		    <groupId>org.apache.commons</groupId>
		    <artifactId>commons-compress</artifactId>
		    <version>1.12</version>
		</dependency>
		<dependency>
		    <groupId>org.apache.commons</groupId>
		    <artifactId>commons-io</artifactId>
		    <version>1.3.2</version>
		</dependency>
  </dependencies>
  
  <build>
    <sourceDirectory>src</sourceDirectory>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.3</version>
        <configuration>
          <source>1.7</source>
          <target>1.7</target>
        </configuration>
      </plugin>
      <plugin>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.6</version>
        <configuration>
          <warSourceDirectory>WebContent</warSourceDirectory>
          <failOnMissingWebXml>false</failOnMissingWebXml>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```
如此设置之后，需要在 webapp/WEB-INF 目录手动建立以下文件夹：
> WEB-INF/conf
WEB-INF/conf/spring
WEB-INF/views/layout
WEB-INF/views/screen

并在 WEB-INF/conf/spring 文件夹下建立 *-beans.xml 文件来进行 Spring 框架的配置。以下为 service-beans.xml 的文件内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"  
    xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
            http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
            http://www.springframework.org/schema/context   
            http://www.springframework.org/schema/context/spring-context-3.0.xsd  
            http://www.springframework.org/schema/aop   
            http://www.springframework.org/schema/aop/spring-aop-3.0.xsd  
            http://www.springframework.org/schema/tx   
            http://www.springframework.org/schema/tx/spring-tx-3.0.xsd  
            http://www.springframework.org/schema/mvc   
            http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd  
            http://www.springframework.org/schema/context   
            http://www.springframework.org/schema/context/spring-context-3.0.xsd"
        default-autowire="byName">

	
	<bean id="messageSource"
		class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
		<property name="defaultEncoding" value="UTF-8" />
		<property name="cacheSeconds" value="5"/>
		<property name="useCodeAsDefaultMessage" value="false"/>
	</bean>

</beans>
```
还应在 WEB-INF/conf 下建立 log4j.properties、server.properties、vm-toolbox.xml 配置文件。内容依次为
```xml
log4j.properties

log4j.rootLogger=INFO,stdout,errorfile

#standout log appender #
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n

#error log appender #
log4j.appender.errorfile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.errorfile.File=${webapp.root}/logs/error.log
log4j.appender.errorfile.Threshold=ERROR
log4j.appender.errorfile.append=true
log4j.appender.errorfile.ImmediateFlush=true
log4j.appender.errorfile.encoding=UTF-8
log4j.appender.errorfile.layout=org.apache.log4j.PatternLayout
log4j.appender.errorfile.layout.ConversionPattern=[%p] %d %l %m %n
```
```xml
server.properties

system.dev.mode=true
app.server.host=localhost
#app.server.host=192.168.224.41
app.server.port=8080

jres.action.scan=zengzy.test
```
```xml
vm-toolbox.xml

<?xml version="1.0"?>
<toolbox>
  <xhtml>true</xhtml>
   <tool>
     <key>stringUtils</key>
     <scope>application</scope>
     <class>org.apache.commons.lang.StringUtils</class>    
   </tool>
   <tool>
     <key>dateUtil</key>
     <scope>application</scope>
     <class>com.hundsun.jresplus.common.util.DateUtil</class>    
   </tool>
</toolbox>
```
至此，基本的框架差不多就搭建完成了。需要注意的是，如果在中文环境下进行程序编写，Eclipse 有可能把字符集选成 gbk，这时候需要到 Window->Preference->Workspace 里进行默认字符集的设置，改为 UTF-8。
## Hello World
因为 Spring 是 MVC 框架（Module，View，Control），而 V 所在的位置是 View/screen/folder，即生成网站之后，访问的视图位于 localhost:8080/Project-Name/folder 之下。对于视图的 *.vm 文件，其实就是类似于 html 语言，比如本次创建的 Hello World 工程，layout 下，创建的 default.vm 内容为：
```jsp
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>Demo</title>
</head>
<body>
    <h3>Layout Test</h3>
	-------------------------------------------------
	<br/>
	$screen_content
	<br/>
	-------------------------------------------------
</body>
</html>
```
而之后要部署到 tomcat 上的文件则位于 screen/folder 之下，创建 test.vm 文件。
```jsp
<h4>中文字符测试</h4>
<form method="POST" action="$appServer.get('/test/test.htm')">
    <fieldset>
        <label>test</label>
        <input name="data"/>
        <input type="submit"/>
    </fieldset>
</form>
```
在工程的 src 下创建 TestAction.java 文件，扮演 MVC 中 C 的角色。
```java
package zengzy.test;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class TestAction {
	@RequestMapping("/test/test.htm")
	public void test() {
	}
}
```
至此，还差最后一步，就是 WEB-INF/ 下的 server.properties 编辑。添加
```xml
jres.action.scan=zengzy.test`
```
来指定控制器的位置。然后右键 Server 标签页下的 Tomcat，首先 Clean，之后点击 Run，"Hello World！"。
## Nginx 与 Tomcat 的混合使用
TODO：
目前只知道将编写好之后的 Web 项目 Export 为 *.war 包，然后扔到服务器上 Tomcat 目录下的 webapps 文件夹中，访问方式为：domain:8080/war-name/folder/test.htm。folder 即为 WEB-INF/screen/folder。
混合 Nginx 与 Tomcat，让 Nginx 进行反向代理仍需学习。
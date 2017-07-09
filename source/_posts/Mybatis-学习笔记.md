---
title: Mybatis 学习笔记
date: 2017-07-08 23:02:27
tags: [Mybatis, MySql, Java]
category: [学习笔记]
---
MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

<!--more-->

# 配置 xml 文件
Mybatis 可以通过 XML 文件快捷地创建 SqlSessionFactory，所以我们先配置 XML 文件可以方便之后对于数据库的操作，下面是一个 config.xml 文件的模板。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTDConfig 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties>
        <property name="username" value="USERNAME"/>
        <property name="password" value="PASSWORD"/>
    </properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://myserver.com/users"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="main/data/user-mapping.xml"/>
    </mappers>
</configuration>
```
当中“driver”需要在 Maven 中配置为：
```xml
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.6</version>
    </dependency>
```
为了对数据库进行远程连接，需要对服务端的数据库进行用户权限设置。我在服务端使用的是 MySql，所有 IP(%) 的远程访问都允许，以下是设置代码：
```sql
GRANT ALL PRIVILEGES ON users.* TO 'USERNAME'@'%'
    IDENTIFIED BY 'PASSWORD' 
    WITH GRANT OPTION;
```
完成设置之后，便可以对远程服务端的数据库进行访问。

另外，Mybatis 还提供 SQL 语句从 XML 到 JAVA 的映射，在 mybatis-config.xml，也就是上面的 XML 文件内，`<mapper resource="main/data/user-mapping.xml"/>`已经指出了所引用的映射定义文件。映射定义 XML 文件大致结构如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
    <select id="selectDriver" resultType="com.example.beans.customer">
    select car_plate,mobile_number,name from driver
    </select>
</mapper>
```
当中的 `resultType` 是 JavaBean，需要预先定义好。Mybatis 对字段名是全匹配处理，例如表内有字段名为“serial_number”的话，在 JavaBean 内也需要将变量名设置为一模一样的“serial_number”，什么“Serial_Number”，“serialNumber”都不可以，会无法将结果集写进去。

在 Java 代码内对从 SqlSessionFactory.openSession() 获取的 SQL Session 进行操作，这样就完成了最基本的从数据库获取数据的任务。
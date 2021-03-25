---
title: database-and-instance
date: 2016-08-17 14:30:28
tags: [database]
category: [学习笔记] 
---
## 完整的数据库
所谓完整的数据库, 包括着数据库(database)和实例(instance). 其中 database 为物理上的数据集, 即磁盘或文件. 而 instance 为一组进程或线程与一个共享内存区, 并且内存区可以与系统内其他程序共享. 也即, instance 就是扮演操作 database 的角色. 而这二者的关系就是, 一个数据库可以被多个实例操作, 而一个实例的生命周期内只能装载和打开一个数据库.
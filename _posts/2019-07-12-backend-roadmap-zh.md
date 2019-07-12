---
layout: post
title:  "后端之路"
date:   2019-07-12 00:53:49
categories: Thinking
tags: 阅读笔记
---

后端之路导图
```

1.选择一门编程语言

脚本语言
    Python
    Ruby
    PHP
    Node.JS(typescript)

函数语言
    Elixir
    scala
    Erlang
    Clojure
    Haskell
    java
    .Net

其他
    Go
    Rust

对于初学者更建议选择脚本语言，简单并且快
推荐是Nodejs或php.如果是已经做过一些后端开发
使用过一些脚本语言推荐选择Go、Rust或者是Clojure
因为能给你一些新的视野

2.根据选择的语言进行练习

实验并且用你选择的语言写一些
命令行应用

推荐实现一个ls命令。
使用命令获取reddit 上/r/programming的帖子
命令行输出一个JSON 的结构
或者构建一个自动化工具

3.学习包管理

学习选择编程语言对应的包管理工具
eg:php的composer、NodeJS的NPM和yarn
Python的Pip、Ruby的gems、Golang的Go mod等

包管理可以帮助管理外部依赖和
帮忙分发自己的包

4.标准和最佳实践

每一门编程语言都有自己的标准
和最佳实践，学习它们。例如php有PHP-FIG和PSRs
NodeJs有很多不同包和框架等

确保阅读了相关编程安全的最佳实践
例如OWASP指南和理解不同的安全问题并且
如何在选择的编程语言中避免它

5.构建并且发布一些包/库

现阶段就可以写一个包并且发布它
提供给其他的人去使用，让自己能巩固标准
和最佳实践。
另外，可以给开源项目做贡献。在github上
给开源项目提pull request，譬如你可以：
利用最佳实践重构或实现issue、尝试修复bug、
添加新功能

6.学习如何测试

测试的方法有很多中，不过主要学习
单元测试和集成测试，理解不同的测试术语。譬如
mocks、stubs等

不同的编程语言有不同的方式和库，只要选择适合自己的
就可以了，可以通过Google。譬如
PHP：PHPUnit、PHPSpec、Codeception
NodeJS：Mocha、chai、sinon、Mockery、Ava、Jasmine
对于其他的语言也一样，很多。

7.实践上面的选择，并且写测试

给自己之前写的包/库写单元测试

学习如何计算测试覆盖率

8.学习关系数据库

选择项很多，如果你已经学习过了。
譬如MySQL，可以选择一门其他的数据库
进行对比不同点和使用:Oracle、MySQL、
MariaDB、PostgreSQL、MSSQL

9.实践时间

创建一个简单应用，充分的利用前面所学的知识
例如做一个Web应用，实现注册和登录、CRUD、创建博客
例如：任何人都可以注册获得一个公共的个人主页
创建、更新、删除博客。并且在公共主页展示所有的博客

要记得写单元测试、应用之前学习的标准和
最佳实践。对于数据库要添加索引、选择适当的存储引擎
并且在开发自己应用的时候分析查询

10.学习一个框架

选择一个开发框架进行学习

不同的编程语言，选择不一样。譬如
php：Laravel、Symfony、Slim、Lumen的微服务框架
NodeJS：Express、Koa、Hapi.Js
Golang: Echo，不过我觉得不需要框架

11.实践时间

使用框架实现步骤9开发的应用

12.学习一个NoSQL数据库

理解什么是NoSQL、它和关系型数据库有什么不同
为什么需要、它们的不同。推荐选择MongoDB

学习过MongoDB可以比较下其他的NoSQL
譬如：RethinkDB、Cassandra、Couchbase

13.缓存

学习如何使用Redis或Memcached实现一个应用级别
的缓存

利用所学在步骤11中实现的应用中加上缓存

14.创建RESTFul API

理解REST和学习如何构建RESTFul API
这里可以阅读Roy Fielding的论文

15.认证和授权

学习两者的不同和实现

OAuth、Basic认证、Token认证、JWT、openID

16.消息系统

学习消息系统，理解为什么需要和选择一个适合
自己的。比较推荐RabbitMQ或者Kafka。学习如何使用
消息队列

17.学习搜索引擎

随着应用的增长、简单的DB查询已经无法满足需求。
这是就需要搜索引擎，搜索引擎有很多可选方案。
比较他们的不同

ElasticSearch、Solr、Sphinx

18.学习如何使用docker

19.Web服务器的知识

Web服务器也有很多可选，理解它们的不同点和缺点

Apache、Nginx、Caddy、MS IIS

20.学习如何使用Websocket

21.学习GraphQL

非必须，只是这个号称是新的REST

22.看看图数据库

不是必须，但是至少要理解它们能做什么
能提供什么

23.还有以上没有提到的。

性能调优、静态分析、DDD、SOAP等

保持阅读
```


![后端之路.png](./attachment/20190711/后端之路.png)

- [后端之路.xmind](./attachment/20190711/后端之路.xmind)

- [backend](https://roadmap.sh/backend)
---
layout: post
title:  "如何阅读技术文档"
date:   2020-02-02 23:29:49
categories: reading
tags: 消息队列 阅读笔记 Golang
---

## 如何阅读技术文档

> 在技术领域经常会接触众多的文档，譬如技术文档、需求文档等。在这里主要讨论的是如何阅读技术文档。它是我们学习新技术除了书籍以外最直接的方法

在某项目中，需要使用到消息队列。经过调研后，选择RabbitMQ作为消息队列的基础服务。服务端语言选择Golang。今天以RabbitMQ为例，来分析如何阅读文档

**第一步：先跑起来**

一个成熟的项目都会有快速指引。进入官网，找到类似`Quick start`或`Get started`的文档。作者会提到如何快速的运行起来一个项目。MQ 项目中是[Get Started](https://www.rabbitmq.com/#getstarted)。它提供安装和教程两部分。

![image-1](/attachment/20200202/2.jpg)

在这里我们只需要使用一行命令，利用现有的容器化运行起来

```bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

**第二步：hello world**

`hello world`是我们开启新世界的钥匙，环境配置好后就需要写第一段代码。因为是消息队列，以发送成功一条`hello world`消息为入口。官方提供了Golang操作MQ发送一条消息的文档。

![image-2](/attachment/20200202/1.jpg)

从文档中，我们知道前置条件。你需要一台已经安装完成的MQ消息队列服务端，这一步已经完成。阅读文档得知：

1. RabbitMQ 是扮演的者消息`经纪人`的角色。负责对消息进行收、转发以及广播。类似邮差

2. 消息包含有三个角色：产生消息的叫生产者、消息传输的通道为队列、收到消息的客户端为消费者

3. 使用Go操作MQ，依赖使用`amqp`库。通过`go get`安装

    ```bash
    go get github.com/streadway/amqp
    ```

4. 按照第2点我们需要实现发送和接收部分。这是两段不同职能的代码，前者为生产者，后者为消费者

**第三步：编码**

看完理论之后，就要开始写代码。在官方文档中，有给出`send.go`的实现：

```go
package main

import (
    "log"

    "github.com/streadway/amqp"
)
//错误处理，当发送或者是连接失败时输出错误flag以及详细
//错误信息
func failOnError(err error, msg string) {
    if err != nil {
        log.Fatalf("%s: %s", msg, err)
    }
}

func main() {
  //建立连接
  //amqp为协议
  //用户名和密码为guest
  //服务器地址:localhost
  //端口: 5672
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    failOnError(err, "Failed to connect to RabbitMQ")
  //程序体运行完成之后，关闭与server端的连接
    defer conn.Close()

  //建立一个channel
    ch, err := conn.Channel()
    failOnError(err, "Failed to open a channel")
  //程序体运行完成之后，关闭通道。
  //defer是一个堆，先进后执行
    defer ch.Close()
    //声明队列，在MQ中需要发送消息前需要声明
  //主要是标示队列拥有的一些属性
  //是否自动删除、队列名称、持久化
    q, err := ch.QueueDeclare(
        "hello", // name
        false,   // durable
        false,   // delete when unused
        false,   // exclusive
        false,   // no-wait
        nil,     // arguments
    )
    failOnError(err, "Failed to declare a queue")
    
  //发送消息
    body := "Hello World!"
    err = ch.Publish(
        "",     // exchange
        q.Name, // routing key
        false,  // mandatory
        false,  // immediate
        amqp.Publishing{
            ContentType: "text/plain",
            Body:        []byte(body),
    })
    log.Printf(" [x] Sent %s", body)
    failOnError(err, "Failed to publish a message")
}
```

执行上面的程序可以看到消息发送成功的日志。通过上面`send.go`中的代码，里面有些内容并没有详细在文档中说明。例如`channel`是什么，为什么要在使用队列之前先打开一个通道才能使用？声明队列的每个参数的含义是什么？发送消息除了消息主体`Hello world`，其他参数的意义是什么？以及我们如何消化生产的消息。

**最后**

上面有好几个问题，带着这些问题去翻阅文档就能慢慢的从冰山一角到一览群山。有些概念上的问题，在文档中一般会在`concept`部分体现。譬如什么是[Channel](https://www.rabbitmq.com/tutorials/amqp-concepts.html#amqp-channels)。它是为了复用单个TCP连接以达到操作多个队列的目的。



#### 参考文献

- [RabbitMQ website](https://www.rabbitmq.com/)
- [RabbitMQ  concept](https://www.rabbitmq.com/tutorials/amqp-concepts.html)
- [Golang](https://golang.org/)
- [Tutorial one go](https://www.rabbitmq.com/tutorials/tutorial-one-go.html)
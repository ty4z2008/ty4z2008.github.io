---
layout: post
title:  "[翻译]消息队列OpenMessaging之使用场景概述"
date:   2019-10-04 13:16:29
categories: reading
tags: 消息队列
---

**摘要**

OpenMessaging项目由阿里巴巴发起，与雅虎、滴滴出行、Streamlio公司共同参与创立，项目意在创立厂商无关、平台无关的分布式消息及流处理领域的应用开发标准。据发起人介绍，随着标准的不断演进，会有更多的互联网、云计算厂商参与到该项目以及生态体系中来。本文的场景大致包含以下方面：

1. P2P  点对点消息
2. Publish/Subscribe 发布/订阅
3. Broadcast 广播
4. Highway/assets/images/use_cases High模式
5. Streaming 流
6. Filter 过滤
7. Routing 路由
8. Online Test 在线测试.A/B测试
9. Upgrade 升级
10. RPC 远程调用

**P2P  点对点消息**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7l8vo2d6jj30d0024mx8.jpg)

点对点消息是比较简单的使用场景。在这个场景中，队列在openMessage中只有一个分区，生产者发送消息到队列中，消费者从队列消费消息。

**发布订阅**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7l9d56tttj30d004374o.jpg)

生产者发送消息到队列，队列内部会存在多个分区。通过轮训或者是Hash的方式存储，这些分区会把消息分派给订阅了相关分区的消费者。

如果需要更高的消息定制化，可以在写入队列前引入Topic（话题）和Routing（路由）来增强消息的处理能力。结构如下图：

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7l9iy1fzyj30d0030dg3.jpg)

**广播**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7l9rehxn8j30d0043mxk.jpg)

广播的方式和发布/订阅的模式比较相似，唯一的区别在于所有的消费者都会对消息进行对应的处理。

**Highway队列**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7l9tt1pyjj30d0012dfw.jpg)

这种场景是对消息的生产特别快，当遇到生产者需要发送大量消息并且可以丢失的情况下使用。相当于批量发送消息到队列。

**消息流**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7l9wabhvij30d006874t.jpg)

StreamingConsumer是为对集成消息系统到大数据处理平台而设计，这种情况下允许StreamingConsumer从某个指定分区消费消息，有点像迭代器。

**过滤**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7lafzf4hbj30d001pmx8.jpg)

在某些场景下，数据需要“清洗”。不是所有的消息都需要达到消费者层。消费者系统消费的消息是经过加工后的消息，其中最常见的就是过滤。

上图中，在路由模型里面加入简单过滤操作（算术运算）。在这个案例中会保留包含Student标签、年龄在18与23之间的消息。

**复制**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7lahs03roj30d005gaah.jpg)

有些时候，生产者和消费者可能分布在不同的数据中心（分布式结构）。OpenMessaging提供一种简单的路由方式（复制），把消息发送给其他数据中心。

**在线测试**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7lakqtnx9j30d002fjrm.jpg)

在线测试对于系统的稳定和运行非常重要。例如A/B测试，创建一个测试队列，然后利用Routing功能对测试队列写入消息。以此来达到测试的目的。

**升级**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7lgwc913oj30d003g3yw.jpg)

上图中，假设消费者端需要发布2.0的新版本，标签为Student的消息发送到1.0队列，标签为Staff的消息发送到2.0队列。通过队列的版本和消费者版本适配，可以创建不同版本的队列来同时支持多版本的消费者。



**RPC（远程调用）**

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g7lh1qlntwj30d004omxc.jpg)

在OpenMessaging中，远程调用等价于同步消息，它不是传统的CS架构（client to server）。而是CSC架构模型（client to server to client）。

**总结**

OpenMessaging是一个实现消息队列中间价的标准，本文是对OpenMessaging应用场景的介绍。目前还处理草案阶段。

**参考资料**

- [解读OpenMessaging开源项目，阿里巴巴发起首个分布式消息领域的国际标准](http://jm.taobao.org/2017/10/18/20171018/)
- [OpenMessaging Use Case](https://github.com/openmessaging/specification/blob/master/usecase.md)


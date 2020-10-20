---
layout: post
title:  "企业级私有镜像仓库Harbor"
date:   2020-10-20 22:10:49
categories: 后端开发
header-mask: 0.25
tags: 
    - openSource
    - 后端开发
catalog: true
---

# 企业级私有镜像仓库Harbor

Harbor[1]是由VMware的国人团队研发，使用Go语言开发的开源私有仓库托管服务平台。

了解Harbor还是最近在某公众号平台看到的一篇文章，里面推荐了一本书《Harbor权威指南》[2]就饶有兴趣的看了看。

官方是这样介绍Harbor：

> An open source trusted cloud native registry project that stores, signs, and scans content.

Harbor，一个开源 可信任的云原生镜像项目。提供存储、签名和安全扫描。今天这篇文章主要是讲一下相关的设计和看法。

#### 关于Harbor的架构设计

下图是官方给出的最新架构图[3] (version 2)：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjw5es558jj31om0u0txb.jpg)

Harbor 采用分层式的架构：数据访问层、基础服务层、消费者。

这种架构是目前很多产品设计架构。以前我们在开发产品时，针对界面和开放需要开发不用的API。虽然刚刚开始的时候比较轻量，但随着时间的推移。想要支持更多的平台和开放形式，压力就逐渐增加。因为你不得不考虑兼容和更新迭代问题，这些都需要人力介入开发。在设计之初，不论是界面还是API ，统一使用一个入口管理（上图中的API Server）。这样不仅统一管理，对边缘生态的开发者足够的友好。同一语言的SDK可以公用，同一功能能集中式的一次完成迭代升级。

#### 数据访问层

redis：临时存储Job服务需要的元数据信息

data storage：也是块存储，类似硬盘。如果你是采用官方的在线部署方式会发下，它会把根目录直接挂载进去，其中数据目录也在其中。值得注意的是，如果你想使用类似云存储或者NAS的服务。Harbor是不单独处理，这一切都是由Docker处理。Docker本身就是支持其他存储驱动，只需要你挂载后像文件系统一样就可以提供给Harbor使用。[4]

Database：主要存储关系数据，各种各样的持久化配置信息。

#### 基础服务层

这部分是Harbor的核心，所有的特性支持都在这一层完成。配额管理、复制管理、Chart、配置、项目、安全扫描、垃圾回收、后台任务分发。我们所使用的功能和访问的功能都是在基础服务里面，这部分对应在源码中的 `src` 目录[5]。

#### 消费者

在Harbor里面，所有的访问者 被当作消费者。官方的管理界面、Kubelet、Helm等等这些访问API的客户端都是消费者

#### 特性

     很有意思的是，Harbor 围绕的是 Docker 的整个生命周期在服务。揽括权限管理、构建、镜像权限访问管理、安全扫描、分发。Harbor 是在整个生命周期扮演着平台的功能，安全和分发是利用插件的能力，安全可以使用开源的 Trivy[6] 或Clair [7] ，分发可以使用蚂蚁金服开源的 Dragonfly [8] 或Kraken。这种开发模式是一种趋势，同样的还有Kubernetes。利用“插件”的模式扩大自己的能力，让更多的开发者能共享到项目里面。具体的插件可以看看官方的兼容列表[10]

缓存代理 给你提供利用Harbor缓存其他源的镜像，有点类似NodeJS 淘宝做的CNPM 一样。只是Harbor是拉取到你的Harbor管理体系里面。

至于一些其他的特性，官方文档里面也说的比较清楚。感兴趣的可以看看相关的文档内容。

#### 碎碎念念

Harbor 的设计不是围绕着Docker，只要是遵守OCI的都支持。如果你的公司或者团队想要搭建一个私有化Registry管理平台可以使用Harbor，鉴权也可以使用你公司的OA用户体系。在使用虚拟机的时候，安全我们可以使用各种各样的工具进行排查。但在容器中，使用传统的方式并不行，各式的容器环境需要我们寻找新的方式。之前是工具已经有了，但是不够自动化，无法串联起来，Harbor的出现让我们搭建私有仓库，多了一个可以选择的方案。

#### 参考资料

1. 《Harbor》 [https://github.com/goharbor/harbor](https://github.com/goharbor/harbor)
2. 《Harbor 企业级Docker镜像仓库》 [https://crimsonromance.github.io/2019/01/12/harbor-企业级docker-registry/](https://crimsonromance.github.io/2019/01/12/harbor-%E4%BC%81%E4%B8%9A%E7%BA%A7docker-registry/)
3. Harbor架构概览 [https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor](https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor)
4. Docker Driver [https://docs.docker.com/registry/storage-drivers/](https://docs.docker.com/registry/storage-drivers/)
5. 基础服务源码 [https://github.com/goharbor/harbor/tree/master/src](https://github.com/goharbor/harbor/tree/master/src)
6. Trivy [https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
7. Clair [https://github.com/quay/clair](https://github.com/quay/clair)
8. Dragonfly [https://github.com/dragonflyoss/Dragonfly](https://github.com/dragonflyoss/Dragonfly)
9. Kraken [https://github.com/uber/kraken](https://github.com/uber/kraken)
10. Harbor 兼容列表 [https://goharbor.io/docs/2.0.0/install-config/harbor-compatibility-list/](https://goharbor.io/docs/2.0.0/install-config/harbor-compatibility-list/)
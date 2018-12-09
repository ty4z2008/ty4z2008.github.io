---
layout: post
title:  "迁移私有化git到gitlab"
date:   2018-12-09 15:16:29
categories: Docker
tags: 阅读笔记
---

摘要

主要从原因、调研、部署、迁移以及总结几个方面

**原因**

我们使用git用四年，团队中的所有项目都是在git下面开发。代码库的代码量在10GB+左右。随着团队的发展。工程化以及自动化的需求与呼声越来越高。我们希望通过git来实现gitops、devops、CI/CD。旧的git服务是使用[gitosis](https://github.com/res0nat0r/gitosis)搭建的.如果是自己开发改造的时间太长。

**调研**

选择了两个git工具：[gogs](gogs.io)和[gitlab](gitlab.com)

|    -      |  Gogs     |   Gitlab    |
|---        |---        |---    |
| 基本的GIT  | 1         |  1     |
| 管理界面   | 1         |  1     |
| CI/CD     |  0        |   1    |
| Codereview|   0       |   1    |

基于以上的考虑，我们最终选择Gitlab。

**部署**

官方建议是使用2核4Gb的配置。这里我主要是用的docker进行部署。使用docker部署主要是考虑到更新版本方便。当出现故障的时候能快速恢复。容器化主要处理的问题是数据落盘问题。这里我们使用磁盘挂载的形式，把数据目录存储在物理主机。然后对物理主机进行磁盘备份。

在使用部署之前还有需要处理一些日常git体验的问题。

1. git如果是使用ssh协议拉取代码，那么最常用的是22端口。解决这个问题是把主机的ssh端口修改掉。修改`/etc/ssh/sshd_config`的port为非22端口。然后重启ssh服务。（注：一般的云服务对端口要防火墙，防火墙也需要配置）。
2. 时间问题，这里通过指定时区即可

下面是docker-compose 的yaml文件配置例子，对于生产环境image最好指定版本，防止后面出现版本文本导致旧的数据不同步

```
version: '3.6'

services:
    gitlab:
        restart: always
        image: gitlab/gitlab-ce:11.5.3-ce.0
        ports:
        - "80:80"
        - "22:22"
        volumes:
        - /data0/gitlab/config:/etc/gitlab:Z
        - /data0/gitlab/logs:/var/log/gitlab:Z
        - /data0/gitlab/data:/var/opt/gitlab:Z
        hostname: 'code.domain.com'
        environment:
        - DEBUG=false

        - TZ=Asia/Shanghai
        - GITLAB_TIMEZONE=Shanghai

        - GITLAB_HTTPS=false
        - SSL_SELF_SIGNED=false

        - GITLAB_HOST=code.domain.com
        - GITLAB_PORT=80
        - GITLAB_SSH_PORT=22
        - GITLAB_RELATIVE_URL_ROOT=
        - GITLAB_SECRETS_DB_KEY_BASE=vjdLTdCdpKPs9qJT7zLvdm4JWhXJdPbRqw9CJvkTvLn4K7R3kMF7RqHrVR3rRF4z #pwgen -Bsv1 64
        - GITLAB_SECRETS_SECRET_KEY_BASE=Lc9HXj7VtXRcwjcs4hfvgWW7Vz4sRdTHw3gRxfVrvxNdrJP3qtfJXqz4vRvkNqvK #pwgen -Bsv1 64
        - GITLAB_SECRETS_OTP_KEY_BASE=XbhxW7tRnNCWjRJbnLh4LW34FrLXsM4PnrspxvHRPzmFXdHkzr4fm9H7wv9Wj9jX #pwgen -Bsv1 64

        - GITLAB_ROOT_PASSWORD=password
        - GITLAB_ROOT_EMAIL=yourname@email_domain.com

        - GITLAB_NOTIFY_ON_BROKEN_BUILDS=true
        - GITLAB_NOTIFY_PUSHER=false

        - GITLAB_EMAIL=yourname@email_domain.com
        - GITLAB_EMAIL_REPLY_TO=yourname@email_domain.com
        - GITLAB_INCOMING_EMAIL_ADDRESS=yourname@email_domain.com

        - GITLAB_BACKUP_SCHEDULE=daily
        - GITLAB_BACKUP_TIME=02:00

        - SMTP_ENABLED=true
        - SMTP_DOMAIN=exmail.qq.com
        - SMTP_HOST=smtp.exmail.qq.com
        - SMTP_PORT=465
        - SMTP_USER=yourname@email_domain.com
        - SMTP_PASS=password
        - SMTP_STARTTLS=true
        - SMTP_AUTHENTICATION=login
        networks:
          gitlib:
            aliases:
              - code.internal.domain.com
              - code.domain.com
networks:
  gitlib:
    external: true
```

**迁移**

当gitlab服务起来之后就是导入就的项目了。

之前的项目地址是`git@git.domain.com:project.git`。而gitlab中项目是属于群组或用户的。

1. 先登录你的web界面添加一个群组或用户。
2. 进入宿主机的`/data0/gitlab/data/git-data/repositories`目录，首先新建群组（与群组名对应）或用户目录（与用户名对应）。
3. 进入刚刚新建的目录
4. 通过scp命令获取是rsync命令把`project.git`导入到当前目录的`project.git`中
5. 如果你是使用root用户导入的先需要修改权限

   ```bash
   chown -R chrony project.git && chmod +770 -R project.git && echo 'project migrate done'
   ```
5. 登录到gitlab容器里面
6.  执行`gitlab-rake gitlab:import:repos`进行恢复。
7.  完成

**总结**

致此就完成了新旧git的迁移。在后续你就可以对项目设立保护分支。进行git-flow。或者是Gitops。团队工程的发展离不开开发工具的更新与迭代。引入新的工具可以让整个团队的软件工程得到一些积极方面的促进。

上面的部署比较适合小的团队。如果是大的团队在HA方面可以对DB、redis使用云服务的db服务或者是单独部署。数据文件存储可以使用NFS。部署多个gitlab服务。然后外部通过slb进行访问。图例：

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fy0l1rkz40j313u0tydi6.jpg)

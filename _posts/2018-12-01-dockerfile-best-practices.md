---
layout: post
title:  "Dockerfile最佳实践Tips"
date:   2018-12-01 15:16:29
categories: Docker
tags: 阅读笔记
---

Dockerfile最佳实践

**1.使用缓存**

Dockerfile的镜像是一层一层的叠加，每一个指令是基于之前层的镜像。如果存在相同父级映像和指令（除了ADD）中，docker不会再次执行相同的指令。而使用缓存。下面是制作镜像时都会添加的部分。注意：如果是多个参数记得使用`\`换行

```
FROM ubuntu
MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y
```
把常用以及通用的命令放在上面，这样可以更好的利用docker image cache

**2. 给镜像添加标签**

在构建镜像的时候记得添加标签.第一是让你的镜像增加可读性。第二是在镜像清理的时候docker有一个命令`docker rmi $(docker images -q)` 会清除为打标签的镜像。其实在增加可读性方便也可以使用`label`标签

```
docker build -t="crosbymichael/sentry" .
```

**3. EXPORSE端口**

在开发对外服务的应用时我们需要暴露端口给外部调用。我们不应该强制在Dockerfile里面指定外部宿主机的端口号。而是仅仅暴露一个私有的端口号。当容器运行时可以使用`docker -p`来指定具体的宿主机端口。

```
# private and public mapping 不推荐
EXPOSE 80:8080

# private only 推荐
EXPOSE 80
```

**4. CMD 和 ENTRYPOINT语法**

CMD的写法有如下两种情况。当命令没有使用`[]`语法时，默认是在`/bin/sh -c`下执行。在使用CMD还是ENTRYPOINT根据你的需要区分，如果是可变的使用`ENTRYPOINT`.不可变的使用`CMD`.推荐大家总是使用`[]`的形式。这样防止出现命令在`/bin/sh -c`产生不确定性问题

```
CMD /bin/echo
# or
CMD ["/bin/echo"]
```

**5.ENTRYPOINT 和 CMD 结合使用.**

来看看Rethinkdb的Dockerfile文件。前5行保持不变。下面设置了一个默认的命令`--help`.这样在容器启动时就能看到帮助手册。

```
# Dockerfile for Rethinkdb 
# http://www.rethinkdb.com/

FROM ubuntu

MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python-software-properties
RUN add-apt-repository ppa:rethinkdb/ppa
RUN apt-get update
RUN apt-get install -y rethinkdb

# Rethinkdb process
EXPOSE 28015
# Rethinkdb admin console
EXPOSE 8080

# Create the /rethinkdb_data dir structure
RUN /usr/bin/rethinkdb create

ENTRYPOINT ["/usr/bin/rethinkdb"]

CMD ["--help"]

```

执行`docker run crosbymichael/rethinkdb`

输出
```
Running 'rethinkdb' will create a new data directory or use an existing one,
  and serve as a RethinkDB cluster node.
File path options:
  -d [ --directory ] path           specify directory to store data and metadata
  --io-threads n                    how many simultaneous I/O operations can happen
                                    at the same time

Machine name options:
  -n [ --machine-name ] arg         the name for this machine (as will appear in
                                    the metadata).  If not specified, it will be
                                    randomly chosen from a short list of names.

Network options:
  --bind {all | addr}               add the address of a local interface to listen
                                    on when accepting connections; loopback
                                    addresses are enabled by default
  --cluster-port port               port for receiving connections from other nodes
  --driver-port port                port for rethinkdb protocol client drivers
  -o [ --port-offset ] offset       all ports used locally will have this value
                                    added
  -j [ --join ] host:port           host and port of a rethinkdb node to connect to
  .................
```

现在使用` --bind all `参数运行.

```
docker run crosbymichael/rethinkdb --bind all
```

输出

```
info: Running rethinkdb 1.7.1-0ubuntu1~precise (GCC 4.6.3)...
info: Running on Linux 3.2.0-45-virtual x86_64
info: Loading data from directory /rethinkdb_data
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/auth_metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
info: Listening for intracluster connections on port 29015
info: Listening for client driver connections on port 28015
info: Listening for administrative HTTP connections on port 8080
info: Listening on addresses: 127.0.0.1, 172.16.42.13
info: Server ready
info: Someone asked for the nonwhitelisted file /js/handlebars.runtime-1.0.0.beta.6.js, if this should be accessible add it to the whitelist.
And there it is, a full Rethinkdb instance running with access to the db and admin console by, interacting with the image the same way you interact with the binary. Very powerful and yet extremely simple. I love simple.
```

所以 ENTRYPOINT 和 CMD 搭配使用更好.

**6. 使用`.dockerignore`**

这个有点类似`.gitignore`。是用来指定要忽略的文件和目录。不要把不必要的文件打包进入镜像里面

参考

- [Dockerfile Best Practices](http://crosbymichael.com/dockerfile-best-practices.html)
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

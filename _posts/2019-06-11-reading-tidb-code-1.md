---
layout: post
title:  "TiDB 源码阅读笔记：开始"
date:   2019-06-11 00:53:49
categories: tidb 数据库系统
tags: tidb
---

**源码编译**

```shell
export GO111MODULE=on
export GOPROXY=https://goproxy.io
make

#启动服务
./bin/tidb-server

#连接
mysql -uroot -P4000 -uroot -h 127.0.0.1
```



**Connect连接**

server/server.go:Run():监听端口，接收连接。解析协议分发给不同的函数

在Run()函数中，通过插件（plugin包）的形式处理连接。

tidb-server/main.go:Main():解析参数、检查版本、注册存储（TiKV/本地TiKV）、注册监控、加载配置、合并参数、校验配置、设置全局变量、写入启动日志、注册jaeger跟踪、打印版本信息、设置binlog、启动监控、创建存储域、创建服务器、发送信号通知启动、创建服务器、清除、执行完毕。定义服务如何启动

Client —进入server/conn.go:Run()：解析连接，分发command到不同的函数:

Run()详细流程：对于连接会设置一个timeout记时，当超过（setReadTimeout()）配置时间时关闭连接。读取网络包(cc.readPacket).当读取失败时断开连接。分发Command(conn.go:dispatch())。

Command：分为Ping、Operation、Query、预处理、关闭连接、设置参数、改变用户、发送、对大数据类型的数据处理（`TEXT`,`TINYTEXT`）、参数重置

- 如果是SQL——conn.go:handleQuery:解析SQL为AST—vaildator校验——逻辑优化——物理优化——物理计划----
- TiDBContext : session数据结构

Session/session.go:定义核心的数据结构、每个会话都拥有一个session

Session/session.go:executeStatement

Q1:Session主要包含什么？

```go
//包含的内容在Session interface中定义
sessionctx.Context 
    Status
    LastInsertID
    LastMessage
    AffectedRows
    Execute
    String
    CommitTxn
    RollbackTxn
    PrepareStmt
    ExecutePreparedStmt
    DropPreparedStmt
    SetClientCapability
    SetConnectionID
    SetCommandValue
    SetProcessInfo
    SetTLSState
    SetCollation
    SetSessionManager
    Close
    Auth
    ShowProcess
    PrepareTxnCtx
    FieldList

```

执行上下文、授权、进程信息、TLS状态、命令、预备执行语句、事务状态、当前的状态值、影响的行、字符集、SQL语句。

#### SQL层架构

![image-20190605213039744](http://ww1.sinaimg.cn/large/006tNc79gy1g3qlqxi3a9j31lg0u07jx.jpg)





**解析SQL为AST**

这部分由[parser](github.com/pingcap/parser) 处理。负责将SQL解析为抽象语法树。

parser组件逻辑（拿到AST的后续处理）：

```
SELECT Hello;
```

解析过程：AST—>parser—>parser.py—>SelectStmt

SQL解析图：[https://pingcap.github.io/sqlgram](https://pingcap.github.io/sqlgram/#ALTER)

如果对语法感兴趣可以看:yacc、lex

**进入validator组件**

Visitor模式：检查语法、遍历AST树。检查是否合法在planner/preprocess.go:Preprocess()

访问者模式：主要将数据结构与数据操作分离。

**进入逻辑优化**

基于规则的优化RBO

executor/builder.go/compiler.go：逻辑计划处理核心

planner/optimize.go：执行优化

planner/core/preprocess.go:Build():构建出逻辑计划

planner/core/expression_rewriter.go:EvalAstExpr

planner/core/expression.go：expression是一个接口（interface）:编译SQL的值为常量（`select 'hello'`）、编译为SQL的函数（`select cast('1,2,2') as k`）、编译为column、编译为corralate column

expression包含表达式相关逻辑，包括各种运算符、内建函数、聚合函数（sum、count）

逻辑优化部分代码：planner/ruler*.go

列裁剪、分区裁剪等

**进入物理优化**

基于代价的优化CBO,得到物理执行计划。

**用物理计划构建出一个tidb executor**

executor/adapter.go

executor是执行器逻辑部分，包含大部分的执行逻辑。

火山模型：数据库执行器中实现的设计模式。该模式把关系代数中的每一种操作抽象为Operator 树，Operator之间使用迭代器串联。对SQL语法的AST抽象



![VS](http://ww1.sinaimg.cn/large/006tNc79gy1g3ris0lkr4j31as0negri.jpg)

解析方式：首先对表扫描，hash聚合、排序、输出值

Chunk format: [http://arrow.apache.org/](http://arrow.apache.org/)

Apache Arrow 是一种基于内存的列式数据结构

util/codec/bytes.go:关系数据存储到tikv的中。key:tableId_rowid。value是数据



执行一条DDL SQL:

```sql
create table t2(id int,name varchar(10));
```

![image-20190610231608349](http://ww1.sinaimg.cn/large/006tNc79gy1g3wgw7cvxtj31vg0u04qq.jpg)

session.go—>ddl.go—>ddl_worker.go—>domain.go—>ddl_worker.go—>ddl.go—>domain.go—>split_region.go



Debug `select 1;`

![image-20190611002902901](http://ww4.sinaimg.cn/large/006tNc79gy1g3wj00ih11j31lx0u0tnq.jpg)

参考：

- [聊聊 Apache Arrow](https://www.infoq.cn/article/apache-arrow)
- [Jaeger: open source, end-to-end distributed tracing](https://www.jaegertracing.io/)
- [Zap: Blazing fast, structured, leveled logging in Go](https://godoc.org/go.uber.org/zap)
- [从Go高性能日志库zap看如何实现高性能Go组件](https://mp.weixin.qq.com/s/i0bMh_gLLrdnhAEWlF-xDw)
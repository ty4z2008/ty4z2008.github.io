---
layout: post
title:  "PostgreSQL开发者模式错误反馈与日志设置"
date:   2018-12-09 15:53:49
categories: PostgreSQL 数据库系统
tags: postgresql
---

**when何时记录**

```
#client_min_messages = notice
log_min_messages = debug5 #debug级别是提供给开发人员使用的，这个可以看到程序调用的信息以及SQL转化为数据结构的信息，每分钟的级别
```

**where记录到哪里**

```
#log_destination = 'stderr'
logging_collector = on   #打开日志收集
log_directory = 'pg_log' #日志目录
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'   
```

**what写什么日志**

```
debug_print_parse = on #解析树
debug_print_rewritten = on #查询重写后的SQL
debug_print_plan = on    #执行计划详细
debug_pretty_print = on #对debug_print_parse,debug_print_rewritten,debug_print_plan可读性格式化
#log_checkpoints = off #如果是研究pg的磁盘IO，这个需要设置为on
log_connections = on #连接日志
log_disconnection = on #断开连接日志
#log_duration=on #语句执行时间，对于分析
```

参考：

- [Error Reporting and Logging](https://www.postgresql.org/docs/current/static/runtime-config-logging.html)
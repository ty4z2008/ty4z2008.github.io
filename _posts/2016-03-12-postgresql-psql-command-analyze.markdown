---
layout: post
title:  "PostgreSQL的psql命令分析"
date:   2016-03-21 12:23:49
categories: PostgreSQL 数据库系统
tags: postgresql
---

postgresql会提供一些系统命令方便开发人员使用，例如`\dt`（查看当前模式所有表）,`\df`（当前模式的函数）,`\l`（查看所有数据库）等等的此类命令，这些命令在系统内部其实都是sql的一个别名而已。例如`\l`:

```
SELECT d.datname as "Name", --datname数据库名
       pg_catalog.pg_get_userbyid(d.datdba) as "Owner",--pg_catalog为系统表,pg_get_userbyid通过pg_user的oid获取用户名
       pg_catalog.pg_encoding_to_char(d.encoding) as "Encoding",--转换编码对应的id转换成字符
       d.datcollate as "Collate",
       d.datctype as "Ctype",
       pg_catalog.array_to_string(d.datacl, E'\n') AS "Access privileges"--datacl为数组,转换成字符串显示
FROM pg_catalog.pg_database d
ORDER BY 1;
```

对比查询`pg_database`数据

```
SELECT * FROM pg_database;
```
查看当前拥有多少数据库，`\l`等同于上面的那一段sql，只是对数据进行了格式化处理。

如何查看这些命令对应的sql语句可以在登录时使用`-E`参数。例如:

```
psql -Upostgres -E -h192.168.2.101
```

如果你是需要做一个postgresql的Web管理平台，那么可以通过同样的思路来实现相同的功能。

参考文献：
system infomation functions:http://www.postgresql.org/docs/9.5/static/functions-info.html

---
layout: post
title:  "优化MySQL数据插入速度"
date:   2019-09-01 20:53:49
categories: 数据库系统
tags: MySQL
---

有时会遇到需要大量写入数据的场景（恢复表、批量插入），当数据插入时会觉得每秒处理的数据记录(row)并不是特别如意。

先看看插入一条记录需要经过那些阶段：

- 建立连接：TCP的3次握手，建立连接。
- 发送数据到数据库服务器
- 解析SQL
- 打开表
- 插入记录：批量插入也会被转换成一条条插入
- 写入索引：每条记录都有索引。写入记录的数量为当前表中存在的索引数目：1 x N 条索引
- 完成

从上面的阶段可以看到，可以优化的部分是最后三条。我们可以考虑通过下面的方式优化插入速度

- 如果是要插入多条记录到MySQL，可以利用`INSERT INTO tableName VALUES`语句，把多条记录合并为一条这样可以减少多次建立连接、发送数据、解析SQL的时间。

- 临时调整`bulk_insert_buffer_size`为一个比较大的值，这里我建议设置为当前系统**可用**内存的1/4。这个参数是处理插入数据的临时缓存buffer。

- `innodb_flush_log_at_trx_commit` 调整。

     为1时，表示每次事务提交时，并立即刷入到磁盘，才执行事务已经完成，系统默认配置。
    为0时指每秒刷新写入到磁盘一次，事务写入到log buffer就认为已经完成，当MySQL进程崩溃时，1秒内的数据会丢失。

    为2时，日志为每秒写入磁盘，与为1的差别是，事务完成是当写入系统缓存时标记为完成。当OS出现问题会丢1秒的数据，而mysqld服务崩溃的时候不会丢弃。

    这里在临时导入数据时建议设置为0.

- 如果数据来自文件，可以使用`LOAD DATA`语句导入数据

- 对于存在默认值的列，只有插入列与默认值不通时才执行插入，这种方式对于已有数据处理的时间是不划算的，虽然是是优化项目。

- 对于innodb的存储引擎，可以临时关闭外键检查`foreign_key_checks`/`unique_checks`参数.

    ```SQL
    SET foreign_key_checks=0;
    SET unique_checks=0;
    ... SQL import statements ...
    SET foreign_key_checks=1;
    SET unique_checks=1;
    ```



**总结**

优化项有很多项目，对于是恢复数据的场景。可以使用下面的参数设置：

```sql
SET foreign_key_checks=0;
SET unique_checks=0;
SET bulk_insert_buffer_size=1/4系统可用内存
```

以上都是会话级别有效的参数.其他的连接不受影响。

下面的参数为系统级别参数，需要重启服务：

```sql
SET innodb_flush_log_at_trx_commit=0;
```

**参考阅读**

- [insert optimization](https://dev.mysql.com/doc/refman/8.0/en/insert-optimization.html)
- [Bulk Data Loading for InnoDB Tables](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-bulk-data-loading.html)



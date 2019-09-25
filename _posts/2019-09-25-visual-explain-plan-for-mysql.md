---
layout: post
title:  "[翻译]MySQL 可视化explain查询计划"
date:   2019-09-25 13:16:29
categories: Thinking
tags: MySQL
---


> `explain`拓展特性需要在MySQL 5.6.的版本以上。本文是讲述workbench的可视化查询计划如何阅读内容.

示例SQL：

```mysql
SELECT CONCAT(customer.last_name, ', ', customer.first_name) AS customer, address.phone, film.title
FROM rental
INNER JOIN customer ON rental.customer_id = customer.customer_id
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
INNER JOIN film ON inventory.film_id = film.film_id
WHERE rental.return_date IS NULL
AND rental_date + INTERVAL film.rental_duration DAY < CURRENT_DATE()
LIMIT 5;
```

![可视化图](https://tva1.sinaimg.cn/large/006y8mN6gy1g7bjb2d9uwj30m8083glo.jpg)

图中的执行顺序：自底向上、从左到右

图形注释：

- 圆：代表操作，譬如GROUP、SORT
- 框：子查询
- 菱形：join，连接
- 标准框：数据表

文字约定

- 标准框：表名或者别名
- 加粗的字体：键/索引
- 框顶部右边的字：扫描行
- 框顶部左边的字：关系扫描代价成本（5.7版本）
- `nest loop`菱形右边文字：连接之后产生的记录
- 菱形顶部的字：JOIN的代价成本（5.7版本）

可视化图信息解释：

| System Name     | Color | Text on Visual Diagram                       | Tooltip related information                                  |
| --------------- | ----- | -------------------------------------------- | ------------------------------------------------------------ |
| SYSTEM          | 蓝色  | Single row: system constant（系统常量）      | 代价低，匹配一条。一般是主键或唯一索引做查询条件<br />       |
| CONST           | 蓝色  | Single row: constant（常量）                 | 代价低，匹配一条.主键查找                                    |
| EQ_REF          | 绿色  | Unique Key Lookup                            | 代价低 -- 使用唯一索引扫，一般出现在多表连接时<br />使用主键或唯一索引作为关联 |
| REF             | 绿色  | Non-Unique Key Lookup                        | 随着记录增加，代价也会增加。使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的记录行 |
| FULLTEXT        | 黄色  | Fulltext Index Search                        | 全文检索，代价低。特定的查询                                 |
| REF_OR_NULL     | 绿色  | Key Lookup + Fetch NULL Values               | 扫描行少时代价低;行数较多时代价高。                          |
| INDEX_MERGE     | 绿色  | Index Merge                                  | 适中 -- 索引合并查询中选择更好的索引来查询。对于多个索引分别进行条件查询，然后将结果合并。如果是index_merge for intersect会需要使用复合索引进一步优化 |
| UNIQUE_SUBQUERY | 橙色  | Unique Key Lookup into table of subquery     | Low -- Used for efficient Subquery processing                |
| INDEX_SUBQUERY  | 橙色  | Non-Unique Key Lookup into table of subquery | Low -- Used for efficient Subquery processing                |
| RANGE           | 橙色  | Index Range Scan                             | 代价中 -- 使用索引进行了范围查找`<`,`>`,`<=`,`>=`,`is null`,`in`,`between` |
| INDEX           | 红色  | Full Index Scan                              | 高 --全索引扫表。对于特别大的索引就应该较大                  |
| ALL             | 红色  | Full Table Scan                              | 代价非常高 -- 全表扫描，无使用索引。                         |
| UNKNOWN         | 黑色  | unknown                                      | Note: This is the default, in case a match cannot be determined |

参考资料

- [代价模型cost model](https://dev.mysql.com/doc/refman/8.0/en/cost-model.html)
- [Visual Explain Plan](https://dev.mysql.com/doc/workbench/en/wb-performance-explain.html)
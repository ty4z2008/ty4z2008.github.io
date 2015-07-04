---
layout: post
title:  "关于PostgreSQL的时间和货币区域问题"
date:   2015-07-04 23:59:49
categories: Jekyll Update
tags: PostgreSQL 数据库系统 
---

出现情况的背景:购买的是[Linode](https://www.linode.com/)的服务,服务的所在地在日本,而地理区域却是美国,但是部署的应用是针对国内区域,在开始安装pg的时候没有太上心,但随着业务的发展和应用的更新区域的问题就慢慢的出现了.例如用户注册的时间,收费项目计费的计算时间,货币单位(Monetary types)等等就开始影响业务的发展.那么能够做的就是对数据库系统和OS系统的位置区域进行调整.

环境:CentOS 7
数据库:PostgreSQL 9.3.6

解决方案:

1.修改CentOS的系统时区:`tzselect`命令选择Asia按照提示操作即可,另外一种方法是直接拷贝文件

 `sudo cp /usr/share/zoneinfo/$主时区/$次时区 /etc/localtime`
 
在中国可以使用(以上海为例)：

`sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`

2.修改postgresql.conf:

```
lc_messages = 'zh_CN.UTF-8'			# 系统错误信息用中文显示
lc_monetary = 'zh_CN.utf8'			# 货币格式选择人民币
lc_numeric = 'zh_CN.UTF-8'			# 数据类型
lc_time = 'zh_CN.UTF-8'				# 时区
```

保存之后重新载入配置文件即可:
```
select pg_reload_conf();
```
or

```
pg_ctl -D data_dir  reload
```

参考资料：

1.[第二十二章 区域](https://wiki.postgresql.org/wiki/9.1%E7%AC%AC%E4%BA%8C%E5%8D%81%E4%BA%8C%E7%AB%A0)

2.[Monetary Types](http://www.postgresql.org/docs/current/static/datatype-money.html)

3.[pg_ctl](http://www.postgresql.org/docs/current/static/app-pg-ctl.html)

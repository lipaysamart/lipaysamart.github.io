---
title: Mysqldump 备份完整流程
date: 2023-04-30 09:57:21
tags:
  - MySQL
categories: Middle-Operator
---


### 执行备份
```sh
mysqldump -uroot -p'EthanLiu@123' \           # 登录数据库账号密码
--all-databases \                       # 所有库进行备份
--single-transaction \                  # 保证数据可用性(不关机备份)
--master-data=2 \                       # 主服务器数据  =2 注释日志 =1 不注释日志
--flush-logs \                          # 刷新日志
> /backup/`date +%F-%H`-mysql-all.sql   # 备份位置 优先 `` 执行日期命令
```

### 观察日志
```sh
vim /backup/2023-04-30-mysql-all.sql    #查看数据备份
```

### 还原数据
```sh
mysql -p 'Ethanliu@123' < /backup/2023-04-30-mysql-all.sql
```
{% note warning %}
还原之后服务器进行重启或者数据库重启,登录密码需要用回备份时的密码，
{% endnote %}

### 二进制日志恢复
```sh
vim /backup/2023-04-30-mysql-all.sql    # 查看数据备份

mysqlbinlog localhost-bin.000002 localhost-bin.000003 --start-position=154 | mysql -uroot -p 'Ethanliu@123'
```
CHANGE MASTER TO MASTER_LOG_FILE='localhost-bin.0000010', MASTER_LOG_POS=154


#### 没有备份，误删除数据库如何还原
```sh
mysqlbinlog localhost-bin.000002 localhost-bin.000003 > binlog.txt
 
```
查看 binlog.txt  `vim binlog.txt`
找到 drop database 的标志行 (**# at 467**) 删除标志行下内容，保存
还原binlog事务日志(`cat binlog.txt | mysql -uroot -p 'Ethanliu@123'`)

---
title: MySQL 主从复制
date: 2023-04-30 11:31:32
tags:
  - MySQL
categories: Middle-Operator
---

### MySQL主从核心概念
* 在主库上把数据更改（DDL DML DCL）记录到二进制（Binary Log）中。
* 备库I/O线程将主库上的日志复制到自己的中继日志（Relay Log）中。
* 备库SQL线程读取中继日志中的事件，将其重放到备库数据库之上。

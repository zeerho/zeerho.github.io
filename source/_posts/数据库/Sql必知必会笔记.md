---
title: Sql必知必会笔记
date: 2018-04-01 22:29:00
tags: [数据库]
---

# 检索数据

## 限制结果

SQL Server 和 Access

`select top 5 {col} from {table};`

DB2

`select {col} from {table} fetch first 5 rows only;`

Oracle

`select {col} from {table} where rownum <= 5;`

MySQL、MariaDB、PostgreSQL 和 SQLite

`select {col} from {table} limit 5;`

`select {col} from {table} limit 5 offset 3;` 第 3 行起的 5 行数据

`select {col} from {table} limit 5,3;` MySQL 和 MariaDB 的快捷键
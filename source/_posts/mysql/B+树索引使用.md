---
title: B+树索引的使用
date: 2022/05/22 21:08:06
math: true
categories:
  - [mysql]
tags:
  - [sql]
---
# InnoDB 使用的索引方案

## InnoDB 头记录信息

| 名称 | 作用  |
| ---- | ---- |
| record_type | 表示记录的类型 |    
| next_record | 表示记录下一跳记录 |    
|  field_info | 字段信息   |    
| 其他信息| 表示记录的其他的信息|

record_type 的取值代表下面的意思
- 0: 普通的用户记录 
- 1: 目录记录
- 2: Infimum 记录
- 3: Superemum 记录

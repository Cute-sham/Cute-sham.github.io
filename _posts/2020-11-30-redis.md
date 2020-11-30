---
layout: post
title: 'Hello'
date: 2020-11-30
author: redis
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: redis
---

> Transform your plain text into static websites and blogs.

### Redis是什么
Redis是现在最受欢迎的NoSQL数据库之一，Redis是一个使用ANSI C编写的开源、包含多种数据结构、支持网络、基于内存、可选持久性的键值对存储数据库，其具备如下特性：

基于内存运行，性能高效
支持分布式，理论上可以无限扩展
key-value存储系统
开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API
相比于其他数据库类型，Redis具备的特点是：

C/S通讯模型
单进程单线程模型
丰富的数据类型
操作具有原子性
持久化
高并发读写
支持lua脚本

### Redis的应用场景有哪些？
  Redis 的应用场景包括：缓存系统（“热点”数据：高频读、低频写）、计数器、消息队列系统、排行榜、社交网络和实时系统。

### Redis的数据类型及主要特性

  Redis提供的数据类型主要分为5种自有类型和一种自定义类型，这5种自有类型包括：String类型、哈希类型、列表类型、集合类型和顺序集合类型。
### String类型：
  它是一个二进制安全的字符串，意味着它不仅能够存储字符串、还能存储图片、视频等多种类型, 最大长度支持512M。

  对每种数据类型，Redis都提供了丰富的操作命令，如：

  GET/MGET
  SET/SETEX/MSET/MSETNX
  INCR/DECR
  GETSET
  DEL
### 哈希类型：
  该类型是由field和关联的value组成的map。其中，field和value都是字符串类型的。

  Hash的操作命令如下：

  HGET/HMGET/HGETALL
  HSET/HMSET/HSETNX
  HEXISTS/HLEN
  HKEYS/HDEL
  HVALS
### 列表类型：
  该类型是一个插入顺序排序的字符串元素集合, 基于双链表实现。

  List的操作命令如下：

  LPUSH/LPUSHX/LPOP/RPUSH/RPUSHX/RPOP/LINSERT/LSET
  LINDEX/LRANGE
  LLEN/LTRIM
### 集合类型：
  Set类型是一种无顺序集合, 它和List类型最大的区别是：集合中的元素没有顺序, 且元素是唯一的。

  Set类型的底层是通过哈希表实现的，其操作命令为：

  SADD/SPOP/SMOVE/SCARD
  SINTER/SDIFF/SDIFFSTORE/SUNION
  Set类型主要应用于：在某些场景，如社交场景中，通过交集、并集和差集运算，通过Set类型可以非常方便地查找共同好友、共同关注和共同偏好等社交关系。
### 顺序集合类型：
  ZSet是一种有序集合类型，每个元素都会关联一个double类型的分数权值，通过这个权值来为集合中的成员进行从小到大的排序。与Set类型一样，其底层也是通过哈希表实现的。

  ZSet命令：

  ZADD/ZPOP/ZMOVE/ZCARD/ZCOUNT
  ZINTER/ZDIFF/ZDIFFSTORE/ZUNION
<iframe type="text/html" width="100%" height="385" src="http://www.youtube.com/embed/gfmjMWjn-Xg" frameborder="0"></iframe>

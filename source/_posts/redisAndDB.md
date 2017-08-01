---
title: 分布式服务 —— redis和DB数据一致性
date: 2017-07-09 17:08:56
tags:
- 分布式数据一致性
- redis
- DB
categories:
- 分布式
---


# 缓存和DB的数据一致性
redis与Mysql的数据一致性问题,有时候我们为了提高系统的反应速度,减少IO频率,会选择减少db的读压力，使用cache做缓存,加快读速度.

## 缓存的选择
- 缓存:redis (redis-cluster) / memcache
- JVM 堆内存: (server端的本地缓存) ,适合存放简短的数据,如果数据量较大会影响server的堆内存,影响GC,对于并发量比较高的可以选择使用ConcurrentHashMap;
- 堆外缓存: 堆外内存就是把内存对象分配在Java虚拟机的堆以外的内存，这些内存直接受操作系统管理（而不是虚拟机），这样做的结果就是能够在一定程度上减少垃圾回收对应用程序造成的影响。

**堆外缓存相关开源实现**  

名称 | 介绍
--- | ---
Ehcache | Ehcache 3.0：3.0基于其商业公司一个非开源的堆外组件的实现。
Chronical Map | OpenHFT包括很多类库，使用这些类库很少产生垃圾，并且应用程序使用这些类库后也很少发生Minor GC。类库主要包括：Chronicle Map，Chronicle Queue等等。
OHC | 来源于Cassandra 3.0， Apache v2。



会引起cache一致性问题。因为db会有事务性导致回滚，而cache无法回滚，会导致脏数据。

## 缓存更新问题

> 【推荐阅读】，缓存架构设计细节二三事：

https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=404087915&idx=1&sn=075664193f334874a3fc87fd4f712ebc&scene=21#wechat_redirect

重点说下写：如果写db成功后，更新cache，会有事务性和并发性两方面问题。

1. 事务性问题：
一个事务包含多个db操作，操作一些db成功，写cache成功，操作二写db失败，事务回滚，db数据回滚，cache无法回滚，导致脏数据。

2. 并发性问题：
两个更新操作并发，如更新名字，并且cache中key以名字为关键字，更新一写db成功，写缓存XXXX_name1成功。更新二写db成功，写缓存XXXX_name2成功。导致cache脏数据。

这里再说一下一般更新操作顺序是失效cache，写db，写cache。会有并发问题。

两个并发操作，更新和读，左边写线程，右边为读线程

①更新操作删除cache

②读操作读cache，miss

③读db，此时是旧数据 

④写db，写cache

⑤写cache 导致cache中脏数据。

虽然写db成功后，失效cache也会有并发问题：更新和读并发   
①查询cache，miss，读db

②写db，失效cache

③写chache

导致cache中脏数据，但是概率极低，并且一般db中写时间长于读时间，并且写会锁表，读需要在写前进入，并且要晚于写操作更新缓存，所以发生概率极低。
## 缓存更新推荐方式

解决方法是 2PC （两阶段提交协议）或是Paxos协议，代价较大。

所以我们采用的方式是：

1. 写数据只写db
2. 先失效cache，再更新数据更新db，（最好不逆序，具体可以参考推荐文章里的介绍）
3. 读数据，先读cache，未命中读db，写入cache

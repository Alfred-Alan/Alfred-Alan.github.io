---
layout: post
title: 'Redis 持久化机制'
description: '谈一谈redis的持久化'
categories: [Redis] 
image: /assets/img/blog/redis-black.png

accent_image: /assets/img/blog/redis-black.png
---

- Table of Contents
{:toc .large-only}

## redis 持久化
Redis由于是使用内存存储机制 而内存在突然停电或死机之后内存就会被清空 如何把数据持久化到磁盘是一个难题

所以redis持久化方式与穿透数据库的方式有比较多的差别，

RDB定时快照方式(snapshot)<br/>
AOF基于语句追加文件(Append only file)<br/>
虚拟内存(vm) （被废弃）<br/>
Diskstore方式 （被废弃）<br/>

<br/>

## 定时快照方式(snapshot)：

该持久化方式实际是在Redis内部一个定时器事件，每隔一段时间就会检查当前时间数据发生改变次数

当如果满足配置的条件则通过操作系统来创建出一个子进程，子进程与父进程共享相同的空间。

这时就可以通过子进程来遍历整个内存来进行存储操作，而主进程仍然可以提供服务，

当有写入时 由操作系统按照内存页(page)为单位来进行 copy-on-write 保证父子进程之间不会受影响

例：save 900 1

​		save 300 10

​		save 60 10000

代表为 900秒内一次读写 300秒内10次读写 60秒内10000次读写

比如 05分的时候更改5次  而电脑在6分的时候死机了 那么这更改的数据也就丢失了

虽然是定期持久化 但是性能要比aof好多了

<br/>

## 基于语句追加方式(aof)：

aof 方式类似mysql的基于语句的binglog方式，即每条会使redis内存数据发生改变的命令都会嘴角到以log文件中。

这个log文件就是redis的持久化数据

aof的方式主要缺点是最佳log文件可能过大，当系统重启恢复数据如果是aof的方式加载数据会非常慢，

过大的数据可能需要很长时间才能加载完，当然这个耗时并不是因为磁盘文件读取慢，而是由于读取的所有命令都要在内存中执行一遍。每条命令都要写log,所以使用aof方式 redis的读写性能也会下降

相比于RDB的好处就是 每次更改都会进行记录 即使电脑崩掉 也可以回复原来的数据






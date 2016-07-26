---
title: "理解StorageLevel在Spark中的使用场景"
date: 2016-04-08 09:30:08 +0800
categories: [bigdata]
tags: [spark]
---

# 背景介绍

RDD提供的对象持久化方法

## cache()
将对象存储在*内存*中。
## persist(StorageLevel)
通过*StorageLevel*参数指定的行为存储对象。
## unpersist()
释放掉存储的对象。并非强制释放，仅仅进行了标记可以被GC掉。

# StorageLevel场景

## MEMORY_ONLY
对象存储在*内存*中，如果内存不足将出现*OutOfMemory*的错误。
## MEMORY_AND_DISK
对象存储在*内存*中，如果内存不足，将数据存放到*磁盘*上。这是我最常用的方式，因为能够以较低的内存需求来架设Hadoop集群。
## MEMORY_ONLY_SER
转换为*字节数组*的对象存储, 相对于*MEMORY_ONLY*省点空间，多消耗点CPU。
## MEMORY_AND_DISK_SER
转换为*字节数组*的对象存储，相对于*MEMORY_AND_DISK*省点空间，多消耗点CPU。
## DISK_ONLY
对象存储在*磁盘*中,只有*被处理*的对象才会被*读入*到*内存*中。这个好省内存，但速度是最慢的了。

# 一些经验点

## OutOfMemory错误
* 适当加大--executor-memory参数值
* 如果可能，尽量仅适用MEMORY_AND_DISK方式存储

## 定时清理垃圾文件
* HDFS中*用户*的/tmp目录，Spark不会主动删除
* Yarn中任务的日志目录，通常日志级别设置为WARN


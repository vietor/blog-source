---
title: "简易安装ZooKeeper服务器"
date: 2016-04-27 13:40:00 +0800
categories: [bigdata]
tags: [zookeeper]
---

**ZooKeeper** 是一个分布式协调系统，曾经是Apache Hadoop的一个子项目。

当前众多项目通过ZooKeeper来进行集群的实现机制，如Kafka、Hadoop等；使得使用者对ZooKeeper的认识可以类比为一个“数据库”系统。

## 环境安装

* 一台安装了JDK的linux服务器.  
* 下载 3.4.x from [zookeeper.apache.org](http://zookeeper.apache.org/releases.html#download)
* 加压zookeeper包到 /opt 目录并创建一个软链接 /opt/zookeeper
* 使用使用 /opt/zookeeper 作为操作目录

## 配置

* 创建数据目录

``` bash
$ mkdir /zookeeper
$ mkdir /zookeeper/data
$ mkdir /zookeeper/logs
```

* 创建文件 conf/zoo.cfg

``` ini
tickTime=2000
dataDir=/zookeeper/data
clientPort=2181
```

* 修改文件 conf/log4j.properties

``` ini
zookeeper.log.dir=/zookeeper/logs/
zookeeper.tracelog.dir=/zookeeper/logs/
```

## 启动与停止

* 启动

``` bash
./bin/zkServer.sh start
```

* 停止

``` bash
./bin/zkServer.sh stop
```

## 使用

``` bash
./bin/zkCli.sh
```


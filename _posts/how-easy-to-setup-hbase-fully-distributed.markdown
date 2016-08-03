---
title: "How easy to setup HBase Fully-distributed"
date: 2016-07-26 21:38:07 +0800
categories: [bigdata]
tags: [hadoop, phoenix, hbase, english]
---

**HBase** is a NoSQL under **Hadoop**.

## Environment

* Three installed jdk's linux server.  
  Demo HBase server ip was 192.168.200.1, 192.168.200.2, 192.168.200.3.  
  Used existing HDFS server ip was 192.168.100.1.  
  Used existing Zookeeper server ip was 192.168.100.2.  
* Download HBase 1.2.x from [hbase.apache.org](http://hbase.apache.org/)
* Extract Hbase's package file to /opt and create a symlink name with /opt/hbase
* Always in /opt/hbase for working

## Configuration

### Master (192.168.200.1)

* Modify file conf/hbase-site.xml

``` xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://192.168.100.1:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>192.168.2.12</value>
  </property>
  <property>
    <name>hbase.zookeeper.clientPort</name>
    <value>2181</value>
  </property>
  <property>
    <name>zookeeper.znode.parent</name>
    <value>/hbase</value>
  </property>
  <property>
    <name>hbase.master.hostname</name>
    <value>192.168.200.1</value>
  </property>
</configuration>
```

hbase.master.hostname set real ip address with itself, don't try the hostname.

### RegionServer (192.168.200.2/3)

* Modify file conf/hbase-site.xml

``` xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://192.168.100.1:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>192.168.2.12</value>
  </property>
  <property>
    <name>hbase.zookeeper.clientPort</name>
    <value>2181</value>
  </property>
  <property>
    <name>zookeeper.znode.parent</name>
    <value>/hbase</value>
  </property>
  <property>
    <name>hbase.regionserver.hostname</name>
    <value>192.168.200.2</value>
  </property>
</configuration>
```

hbase.regionserver.hostname set real ip address with itself, don't try the hostname.

### More Configuration

* Modify file conf/hbase-site.xml

- HBase files replication in HDFS

``` xml
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>
```

## Startup & Shutdown

### Master

* Start

``` bash
./bin/hbase-daemon.sh start master
```

* Stop

``` bash
./bin/hbase-daemon.sh stop master
```

### RegionServer

* Start

``` bash
./bin/hbase-daemon.sh start regionserver
```

* Stop

``` bash
./bin/hbase-daemon.sh stop regionserver
```

## Web interface

Open http://192.168.200.1:16010 in web browser, enjoy it.

## Phoenix Installed

**Phoenix** is a SQL wapper for **HBase**.

* Download phoenix-[version]-bin.tar.gz from [phoenix.apache.org](https://phoenix.apache.org/download.html)
* Extract phoenix-[version]-server.jar and **Copy** it to '/opt/hbase/lib' directory in **All HBase instance**
* Restart Add HBase instance.
* How to usage it? [Click here](http://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html)

### Configuration

* Modify file conf/hbase-site.xml

``` xml
<property>
    <name>hbase.table.sanity.checks</name>
    <value>false</value>
</property>
```

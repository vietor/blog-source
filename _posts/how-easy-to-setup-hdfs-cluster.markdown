---
title: "How easy to setup HDFS cluster"
date: 2015-04-17 09:38:47 +0800
categories: [bigdata]
tags: [hadoop, hdfs, english]
---

**HDFS** is a file store system for **Hadoop**

## Environment

* Three installed jdk's linux server.  
  Demo HDFS server ip was 192.168.100.1, 192.168.100.2, 192.168.100.3.
* Download hadoop 2.6.x from [hadoop.apache.org](http://hadoop.apache.org/releases.html#Download)
* Extract hadoop's package file to /opt and create a symlink name with /opt/hadoop
* Always in /opt/hadoop for working

## Configuration

### Global (all server)

* Create target directory

``` bash
$ mkdir /hdfs
$ mkdir /hdfs/logs
```

* Modify file etc/hadoop/hadoop-env.sh

``` bash
export JAVA_HOME=<real java home>
export HADOOP_LOG_DIR=/hdfs/logs
```

* Modify file etc/hadoop/core-site.xml

``` xml
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://192.168.100.1:8020</value>
    <final>true</final>
  </property>
</configuration>
```

### Namenode (192.168.100.1)

* Create target directory

``` bash
$ mkdir -p /hdfs/name
```

* Modify file etc/hadoop/hdfs-site.xml

``` xml
<configuration>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/hdfs/name</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
    <value>false</value>
  </property>
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>
</configuration>
```

* Format namenode directory

``` bash
$ ./bin/hdfs namenode -format
```


### DataNode (192.168.100.2/3)

* Create target directory

``` bash
$ mkdir -p /hdfs/data
```

* Modify file etc/hadoop/hdfs-site.xml (192.168.100.2)

``` xml
<configuration>
  <property>
    <name>dfs.datanode.hostname</name>
    <value>192.168.100.2</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/hdfs/data</value>
    <final>true</final>
  </property>
</configuration>
```
dfs.datanode.hostname set real ip address with itself, don't try the hostname.

## Startup & Shutdown

### NameNode

* Start

``` bash
./sbin/hadoop-daemon.sh --config ./etc/hadoop --script hdfs start namenode
```

* Stop

``` bash
./sbin/hadoop-daemon.sh --config ./etc/hadoop --script hdfs stop namenode
```

### DataNode

* Start

``` bash
./sbin/hadoop-daemon.sh --config ./etc/hadoop --script hdfs start datanode
```

* Stop

``` bash
./sbin/hadoop-daemon.sh --config ./etc/hadoop --script hdfs stop datanode
```

## Web interface

Open http://192.168.100.1:50070 in web browser, enjoy it.


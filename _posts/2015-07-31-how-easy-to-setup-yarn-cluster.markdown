---
title: "How easy to setup YARN cluster"
date: 2015-07-31 14:38:47 +0800
categories: [bigdata]
tags: [hadoop, yarn]
---

**YARN** is a task execute system for **Hadoop**

## Environment

* Three installed jdk's linux server.  
  Demo YARN server ip was 192.168.200.1, 192.168.200.2, 192.168.200.3.  
  Used existing HDFS server ip was 192.168.100.1.
* Download hadoop 2.6.x from [hadoop.apache.org](http://hadoop.apache.org/releases.html#Download)
* Extract hadoop's package file to /opt and create a symlink name with /opt/hadoop
* Always in /opt/hadoop for working

## Configuration

### Global (all server)

* Create target directory

``` bash
$ mkdir /yarn
$ mkdir /yarn/logs
```

* Modify file etc/hadoop/yarn-env.sh

``` bash
export YARN_LOG_DIR=/yarn/logs
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

* Modify file etc/hadoop/yarn-site.xml

``` xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>192.168.200.1</value>
  </property>
</configuration>
```

### ResourceManager (192.168.200.1)

* Create target directory

``` bash
$ mkdir -p /yarn/recovery
```

* Modify file etc/hadoop/yarn-site.xml

``` xml
<configuration>
  <property>
   <name>yarn.resourcemanager.recovery.enabled</name>
   <value>true</value>
  </property>
  <property>
   <name>yarn.resourcemanager.fs.state-store.uri</name>
   <value>file:/yarn/recovery</value>
  </property>
</configuration>
```

### NodeManager (192.168.200.2/3)

* Create target directory

``` bash
$ mkdir -p /yarn/recovery
```

* Modify file etc/hadoop/yarn-site.xml (192.168.200.2)

``` xml
<configuration>
  <property>
   <name>yarn.nodemanager.hostname</name>
   <value>192.168.200.2</value>
  </property>
  <property>
   <name>yarn.nodemanager.address</name>
   <value>${yarn.nodemanager.hostname}:8048</value>
  </property>
  <property>
   <name>yarn.nodemanager.recovery.enabled</name>
   <value>true</value>
  </property>
  <property>
   <name>yarn.nodemanager.recovery.dir</name>
   <value>/yarn/recovery</value>
  </property>
</configuration>
```
dfs.nodemanager.hostname set real ip address with itself, don't try the hostname.

## Startup & Shutdown

### ResourceManager

* Start

``` bash
./sbin/yarn-daemon.sh --config ./etc/hadoop start resourcemanager
```

* Stop

``` bash
./sbin/yarn-daemon.sh --config ./etc/hadoop stop resourcemanager
```

### NodeManager

* Start

``` bash
./sbin/yarn-daemon.sh --config ./etc/hadoop start nodemanager
```

* Stop

``` bash
./sbin/yarn-daemon.sh --config ./etc/hadoop stop nodemanager
```

## Web interface

Open http://192.168.200.1:8080 in web browser, enjoy it.

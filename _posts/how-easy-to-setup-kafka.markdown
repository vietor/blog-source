---
title: "How easy to setup Kafka"
date: 2016-08-04 20:38:07 +0800
categories: [bigdata]
tags: [kafka, english]
---

**Kafka** is a Runtime Message Queue.

## Environment

* Three installed jdk's linux server.  
  Demo Kafka server ip was 192.168.200.1, 192.168.200.2, 192.168.200.3.  
  Used existing Zookeeper server ip was 192.168.100.2.  
* Download Kafka 0.9.x from [kafka.apache.org](http://kafka.apache.org/downloads.html)
* Extract Hbase's package file to /opt and create a symlink name with /opt/kafka
* Always in /opt/kafka for working

## Configuration

### Global (all server)

* Create target directory

``` bash
$ mkdir /kafka
```

* Modify file config/server.properties

``` ini
port=9092
log.dirs=/kafka
zookeeper.connect=192.168.2.12:2181/kafka
```

### Node (192.168.200.1)

* Modify file config/server.properties

``` ini
broker.id=1
advertised.host.name=192.168.200.1
```

### Node (192.168.200.2)

* Modify file config/server.properties

``` ini
broker.id=2
advertised.host.name=192.168.200.2
```

### Node (192.168.200.3)

* Modify file config/server.properties

``` ini
broker.id=3
advertised.host.name=192.168.200.3
```

## Startup & Shutdown

* Start

``` bash
./bin/kafka-server-start.sh config/server.properties
```

* Stop

``` bash
./bin/kafka-server-stop.sh config/server.properties
```

## Usage

### Create Topic

``` bash
./bin/kafka-topics.sh --zookeeper=192.168.100.2:2181/kafka --create --topic test
```

### Send Message To Topic (Test)

``` bash
./bin/kafka-console-producer.sh --broker-list=192.168.200.1:9092,192.168.200.2:9092,192.168.200.3:9092 --topic=test
```

### Receive Message From Topic (Test)

``` bash
./bin/kafka-console-consumer.sh --zookeeper=192.168.100.2:2181/kafka --topic=test
```

## Mirror Server

### Environment

* Mirror Server ip was 192.168.200.10
* The Kafka2 server group
  Zookeeper: 192.168.100.2:2181/kafka2  
  Demo server ip was 192.168.200.11,192.168.200.12,192.168.20.13  
* Install to /opt/kafka, and configure it.
* Data stream copy from kafka to kafka2

### Configuration

* Modify file config/consumer.properties

``` ini
zookeeper.connect=192.168.100.2:2181/kafka
```

* Modify file config/producer.properties

``` ini
bootstrap.servers=192.168.200.11:9092,192.168.200.12:9092,192.168.200.13:9092
metadata.broker.list=192.168.200.11:9092,192.168.200.12:9092,192.168.200.13:9092
```

### Startup

``` bash
./bin/kafka-mirror-maker.sh --consumer.config=config/consumer.properties --producer.config=config/producer.properties --blacklist='ignore*'
```

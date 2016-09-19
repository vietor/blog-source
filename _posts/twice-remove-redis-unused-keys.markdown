---
title: "再一次清理Redis中多余的Key的操作过程"
date: 2016-09-19 17:29:43 +0800
categories: [bigdata]
tags: [redis]
---

在我们的一个系统中，一个Redis存储KEY的TTL设置为5年，但后期内存压力较大需要清理较久的数据。根据业务逻辑的特性，确定删除TTL有效期小于4年的KEY（即，创建时间超过1年）。

# 将Redis的rdb文件导出
通常有两种方案：
- 直接使用copy命令复制dump.rdb文件；
- 通过redis-cli的--rdb命令导出。

# 将要删除的Key从rdb中导出来

需要首先安装[redis-rdb-tools](https://github.com/LesTR/redis-rdb-tools)

将带所有的KEY及TTL等信息导出到csv文件
``` bash
rdb --command memory dump.rdb > usertoken_keys.csv
```
此时生成的CSV文件格式如下:
``` bash
database,type,key,size_in_bytes,encoding,num_elements,len_largest_element,ttl
0,string,"kt.usertoken-C102685U46553459",237,string,86,86,130272503.0
0,string,"kt.usertoken-C102685U402255982",238,string,86,86,135267585.0
0,string,"kt.usertoken-C103439U432281392",238,string,86,86,148310228.0
0,string,"kt.usertoken-C103367U440243305",238,string,86,86,150836002.0
```

将CSV中TTL小于4年的KEY字段导出：
``` bash
sed '1d' usertoken_keys.csv | awk -F',' '{if($8 < 126144000){gsub(/[:\"]/,"",$3); print $3}}' > rkeys.csv
```
此时生成的rkeys.txt文件中就包含所有的KEY，可通过写一个python程序读此文件完成实际的删除。生成文件内容样例：
``` bash
kt.usertoken-C102685U46553459
kt.usertoken-C102685U402255982
kt.usertoken-C101287U35606393
kt.usertoken-C101675U404094112
kt.usertoken-C101140U36998042
```

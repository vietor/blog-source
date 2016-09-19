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

将所有**要删除的KEY**导出到csv文件  
*我们的场景是排除所有包含kt.usertoken的KEY*
``` bash
rdb --command memory dump.rdb | grep -v "kt.usertoken" > rkeys.csv 
```
此时生成的CSV文件格式如下:
``` bash
database,type,key,size_in_bytes,encoding,num_elements,len_largest_element
0,string,"PREFIX_USER_MSG_912c05fac706a6fa679326e95d188f67_0",142,string,2,2
0,string,"PREFIX_USER_MSG_99641fecd9d30a3c07b02836c7a8147f2618cb56_6847",438,string,287,287
0,string,"PREFIX_USER_MSG_c0c694ddbec7a82d1755fcd79026260e834a0e29_0",150,string,2,2
0,string,"PREFIX_USER_MSG_e246022a79f1fbd52b3dbecef18c51d27a6f3e9f_0",150,string,2,2
0,string,"PREFIX_USER_MSG_719bd47ebf34743150b5535d841e9628bc27decd_6851",153,string,2,2
```

将CSV中的KEY字段导出：
``` bash
sed '1d' rkeys.csv | awk 'match($0, /\,\"(.*)\"\,/, a) {print a[1]}' > rkeys.txt
```
此时生成的rkeys.txt文件中就包含所有的KEY，可通过写一个python程序读此文件完成实际的删除。生成文件内容样例：
``` bash
PREFIX_USER_MSG_912c05fac706a6fa679326e95d188f67_0
PREFIX_USER_MSG_99641fecd9d30a3c07b02836c7a8147f2618cb56_6847
PREFIX_USER_MSG_c0c694ddbec7a82d1755fcd79026260e834a0e29_0
PREFIX_USER_MSG_e246022a79f1fbd52b3dbecef18c51d27a6f3e9f_0
```

# 操作过程中遇到的一些坑
- 使用[csvtool](https://github.com/Chris00/ocaml-csv)无法操作**大文件**，所以采用awk代替。
- 使用bash脚本来读取rkeys.txt进而调用**redis-cli del**来进行KEY的删除很容易出现“特殊字符”问题，所以尽量使用程序来删除。


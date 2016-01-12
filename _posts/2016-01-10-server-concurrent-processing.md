---
layout: post
title: "服务器并发处理能力"
description: ""
category: 
author: "kingnet"
tags: []
---

<h3>吞吐率 </h3>
   单位时间内服务器处理的请求数据来描述其并发处理能力，单位是  reqs/s
   我们更关心的是服务器并发处理能力的上限，也就是单位时间内服务器能够处理的
   最大请求数，即最大吞吐率。

   “压力测试” 通过模拟足够数目的并发用户数，分别持续发送一定的HTTP请求数，并统计
   测试持续的总时间，计算出基于这种“压力”下的吞吐率，即为一个平均值。

    压力描述一般包括两部分。即并发用户数和总请求数
    也就是模拟多少个用户同时向服务器发送多少个请求。

    并发用户数 就是指定某一时刻同时向服务器发送请求的用户总数


请求等待时间
1. 用户平均请求等待时间 主要衡量服务器在一定并发用户数的情况下，对于单个用户用户的服务质量
2. 服务器平均请求处理时间 衡量服务器的整体服务质量，它其实就是吞吐率的倒数


```
ab -n1000 -c100 http://localhost/phpinfo.php

-n1000 表示总请求数为1000
-c 100 表示并发用户数位100
This is ApacheBench, Version 2.3 <$Revision: 1663405 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking IP地址 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        nginx
Server Hostname:        localhost
Server Port:            80

Document Path:          /phpinfo.php
Document Length:        89452 bytes

Concurrency Level:      100      表示并发用户数，我们设置的参数
Time taken for tests:   130.281 seconds 
表示所有这些请求被处理完成所花费的总时间

Complete requests:      1000   表示总请求数，我们这种的参数
Failed requests:        275
   (Connect: 0, Receive: 0, Length: 275, Exceptions: 0)
Total transferred:      89631112 bytes
HTML transferred:       89452112 bytes

Requests per second:    7.68 [#/sec] (mean)
吞吐率，计算公式 = Complete requests / Time taken for tests

Time per request:       13028.053 [ms] (mean)
用户平均请求等待时间
计算公式 =  Time taken for tests/(Complete requests/Concurrency Level)

Time per request:       130.281 [ms] (mean, across all concurrent requests)
服务器平均请求处理时间,吞吐率的倒数
计算公式 = Time taken for tests / Complete requests 

Transfer rate:          671.86 [Kbytes/sec] received
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       12  677 963.5    503    8151
Processing:  5967 12075 3429.7  11435   41223
Waiting:       14 1016 1123.3    859   10352
Total:       6372 12752 3858.9  11946   44680


```

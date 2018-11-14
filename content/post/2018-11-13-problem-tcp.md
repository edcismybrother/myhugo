---
date: 2018-11-13
title: "tcp长连接压测遇到的问题"
author: "皮智文"
tags: ["tcp","bug"]
url: "/tcp长连接压测"
---

# tcp长连接压测遇到的问题

## 问题1：在大量长连接下，会出现很多拨号失败。

、、、
dial tcp 192.168.31.232:6001: 
connectex: A connection attempt failed because the connected party did not properly respond after a period of time,
or established connection failed because connected host has failed to respond
、、、
拨号失败的问题在于服务端响应不及时
问题分析：在大量长连接同时请求时，出现了cpu、资源调用阻塞，很多的连接在等待分配
解决办法：轮询创建客户端加上延时

## 问题2：部分连接成功的客户端，会在请求的时候得不到相应的响应

目前的模拟客户端设置的时1分钟的超时，在5000长连接的压测下，30s的超时设置，会出现n个超时，1分钟的超时设置，超时的客户端会减少
问题分析：估计也是资源占用造成的问题
解决办法：加大服务器内核

## 问题3：部分连接成功的客户端，请求后被告知连接被远程端强制关闭，但服务端却没有主动关闭连接的操作

客户端已经dial成功，但通过返回的conn进行write和read操作被告知连接强制关闭，但服务端没有任何close迹象
原因分析：dial是在第二次握手的时候返回conn，而服务端accept是在第三次握手的时候返回conn，也就是说握手只进行到第二次握手，
这是因为握手的时候有两个队列，一个syn队列一个backlog队列（accept队列），一次握手，cli告诉ser要建立连接，第二次握手，ser通知
cli等待确认连接，这时连接状态进入syn队列，第三次握手，cli通知ser确认连接，连接状态进入backlog队列。
以上就是握手的过程，syn和backlog队列有限制，如果在大量并发连接出现，连接数超过backlog的长度的连接，将会阻塞在队列外，一定时间没有进入到backlog队列，连接将会被丢弃，这就出现了客户端dail成功，而服务端却没有accept。
解决办法：修改服务器内核配置，syn队列上限： /proc/sys/net/core/somaxconn；accept队列上限：/proc/sys/net/ipv4/tcp_max_syn_backlog

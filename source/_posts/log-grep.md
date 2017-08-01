---
title: 线上日志排查
date: 2017-03-09 17:08:56
tags: 
- 线上
- 日志
- log
- 问题排查
categories: 
- tools
---




### 线上问题排查

ssh登录相关server，然后你可以想干什么干什么了。。。

查看谋渠道出现的问题,使用grep命令，相关参数可[参考](http://man.linuxde.net/grep)
```
cat  sec-adv.stdout.log |grep  "08-0"
```
-C 显示前后20行

```
cat  sec-adv.stdout.log |grep  "08-0" -C30
```
统计广告拉取错误次数   -c 计算符合范本样式的列数。

```
cat  sec-adv.stdout.log |grep  "Fetch ad error" -c
```

重定向到日志文件
```
cat  sec-adv.stdout.log |grep  "Fetch ad error" > exception.log
```

远程拷贝到本地
```
scp c3_fetchAdError.log oliver@ip:/Users/oliver/tem
```

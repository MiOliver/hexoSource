---
title: redis数据导入导出
date: 2017-04-25 11:21:20
tags:
- redis
- sed
- 导入
- 导出
categories: 
- tools
---


简单的数据导出导出方法:

## redis数据导出
导出数据到某个文件overseaimei中
```
echo "smembers universal.01-2.imei" | redis-cluster >> overseaimei.raw
```
此处的redis-cluster重使用alias替换了
```
alias redis-cluster='redis-cli -h 10.*.*.* -p 6382 -c'
```
等同于
```
echo "smembers universal.01-2.imei" | redis-cli -h 10.*.*.* -p 6382 -c >> overseaimei.raw
```

## redis数据导入
上述方式导出的是原始数据,在导入数据之前做一下简单数据处理,原始数据格式是这样的:
```
5acac4d190c93efa38964a66d4e7d996
3e6cba815feb7bc14019fd17806b42e4
50fedc7fff6024a307d9459e93c66647
148eb49a710b839ca78569767ddb11bb
41e1ea70094bb91f583b05424521b047
```
开头每行加一下redis的add命令,使用sed命令,具体用法参考下面说明:
```
sed 's/^/sadd universal.01-2.imei /g' overseaimei.raw  >> backimei.txt
```
更改完后的数据:
```
sadd universal.01-2.imei a2e8abe1a087be26fc011db57a9e7b30
sadd universal.01-2.imei fe99ee3d3d48f878666c5482d2310632
sadd universal.01-2.imei 5acac4d190c93efa38964a66d4e7d996
sadd universal.01-2.imei 3e6cba815feb7bc14019fd17806b42e4
sadd universal.01-2.imei 50fedc7fff6024a307d9459e93c66647
sadd universal.01-2.imei 148eb49a710b839ca78569767ddb11b
```

执行导入操作:
```
cat backimei.txt | redis-cli -h 10.38.*.* -p 6380 -c
```

至此完成导入操作,= O(∩_∩)O~

## sed 简单用法
详细可以参考陈浩老师的[博客](http://coolshell.cn/articles/9104.html)
相关正则表达式
```
^ 表示一行的开头。如：/^#/ 以#开头的匹配。
$ 表示一行的结尾。如：/}$/ 以}结尾的匹配。
\< 表示词首。 如 \<abc 表示以 abc 为首的詞。
\> 表示词尾。 如 abc\> 表示以 abc 結尾的詞。
. 表示任何单个字符。
* 表示某个字符出现了0次或多次。
[ ] 字符集合。 如：[abc]表示匹配a或b或c，还有[a-zA-Z]表示匹配所有的26个字符。如果其中有^表示反，如[^a]表示非a的字符
```
例子:
在每一行最前面加点东西：
```
sed 's/^/#/g' pets.txt
```
s表示替换命令，/^表示开头，/#表示把匹配替换成#，/g 表示一行上的替换所有的匹配,简单解释就是在行首添加#

同样的在每一行尾加点东西：
```
sed 's/$/ --- /g' pets.txt
```

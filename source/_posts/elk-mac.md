---
title: ELK 本地环境搭建
date: 2017-03-27 16:42:47
tags:
- elasticsearch
- kabana
- logstash
categories:
- tools	
---
### 什么是ELK
ELK 其实并不是一款软件，而是一整套解决方案，是三个软件产品的首字母缩写，Elasticsearch，Logstash 和 Kibana。这三款软件都是开源软件，通常是配合使用，而且又先后归于 Elastic.co 公司名下，故被简称为 ELK 协议栈;

日志主要包括系统日志、应用程序日志和安全日志。系统运维和开发人员可以通过日志了解服务器软硬件信息、检查配置过程中的错误及错误发生的原因。经常分析日志可以了解服务器的负荷，性能安全性，从而及时采取措施纠正错误  
开源实时日志分析ELK平台能够完美的解决我们上述的问题，ELK由ElasticSearch、Logstash和Kiabana三个开源工具组成。官方网站： https://www.elastic.co/products

Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。
Logstash是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。
Kibana 也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

关于此三者的介绍可以从参考IBM社区的文章：https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/
介绍的算是比较具体了。

笔者使用的系统是Mac OSX，版本选择是**elasticsearch-5.0.0**   +    **kibana-5.0.0-darwin-x86_64**   +   **logstash-5.2.2**

### Elasticsearch Install
download from  https://www.elastic.co/start

下载->解压->修改config->运行测试

```
 mv ~/Downloads/elasticsearch-5.0.0.tar.gz yourfolder

tar -xzvf elasticsearch-5.0.0.tar.gz -C elk

cd elk/elasticsearch-5.0.0/config

vi  elasticsearch.yml

```
找到并修改此行：
network.host: localhost

- 启动：

```
cd ../bin
./elasticsearch
```
- 测试：

```
curl 'localhost:9200/'
{
  "name" : "rioHP-U",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "R-H7YYg_QLGO4zGmuKWnlA",
  "version" : {
    "number" : "5.0.0",
    "build_hash" : "253032b",
    "build_date" : "2016-10-26T04:37:51.531Z",
    "build_snapshot" : false,
    "lucene_version" : "6.2.0"
  },
  "tagline" : "You Know, for Search"
}
```
此时，表示你的elasticsearch已经配置成功了。

### Kibana install
download from  https://www.elastic.co/start

下载->解压->修改config->运行

```
mv ~/Downloads/kibana-5.0.0-darwin-x86_64.tar.gz yourfolder

tar -xzvf kibana-5.0.0-darwin-x86_64.tar.gz -C elk

cd elk/kibana-5.0.0-darwin-x86_64/config

vi kibana.yml

```
更改 elasticsearch.url，指向你的Elasticsearch cluster的url,如果是本地，一般直接去掉注释符#即可

```
# elasticsearch.url: "http://localhost:9200"
```
- 启动

```
cd ../bin
./kibana

  log   [03:09:44.945] [info][listening] Server running at http://localhost:5601
  log   [03:09:44.946] [info][status][ui settings] Status changed from uninitialized to yellow - Elasticsearch plugin is yellow
  log   [03:09:49.957] [info][status][plugin:elasticsearch@5.0.0] Status changed from yellow to yellow - No existing Kibana index found
  log   [03:09:50.648] [info][status][plugin:elasticsearch@5.0.0] Status changed from yellow to green - Kibana index ready
  log   [03:09:50.648] [info][status][ui settings] Status changed from yellow to green - Ready
  
```

此时，表示你的kibana已经启动成功了。

可以打开http://localhost:5601； 查看一下

### Logstash install

download from  https://www.elastic.co/products

下载->解压

```
mv ~/Downloads/logstash-5.2.2.tar.gz yourfolder

tar -xzvf logstash-5.2.2.tar.gz -C elk

```
在对应的bin/目录创建一个配置文件logstash-simple.conf

```
touch logstash-simple.conf
```
更改文件内容：
```
input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```
运行logstash,-f 指定你的配置文件

```
./logstash -f logstash-simple.conf

[2017-03-27T11:20:18,711][INFO ][logstash.pipeline        ] Pipeline main started
The stdin plugin is now waiting for input:
```
管道启动成功，等待用户输入，从 hello world 开始：
```
hello world
{
    "@timestamp" => 2017-03-27T03:23:35.374Z,
      "@version" => "1",
          "host" => "Olivers-iMac.local",
       "message" => "hello world"
}
```
此时在kibana中刷新一下，即可看到相关日志，见下图
{% asset_img kibana.png kibana %}


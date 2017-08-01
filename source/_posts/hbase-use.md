---
title: Hbase常用命令
date: 2017-03-08 11:21:20
tags: 
- Hbase
- Shell
- 常用
categories: 
- tools
---

本文列出了hbase使用中用到的常用命令  
### hbase 进入
1.使用jsp查看是否hbase正常  
2.进入hbase shell,找到hbase的bin目录
```
bin/hbase shell

```
### hbase 基本命令

名称 |	命令表达式
--- |---
创建表|	create '表名称', '列名称1','列名称2','列名称N'
添加记录 |	put '表名称', '行名称', '列名称:xxx', '值'
查看记录|	get '表名称', '行名称'
查看表中的记录总数|	count  '表名称'
删除记录|	delete  '表名' ,'行名称' , '列名称'
删除一张表|	先要屏蔽该表，才能对该表进行删除，第一步 disable '表名称' 第二步 drop '表名称'
查看所有记录|	scan "表名称"  
查看某个表某个列中数据|	scan "表名称" , {COLUMNS => ['A'], LIMIT => 10}
更新记录|	就是重写一遍进行覆盖

### 普通操作举例
1.查询服务器状态：status
```
hbase(main):002:0> status
1 active master, 0 backup masters, 1 servers, 0 dead, 3.0000 average load
```

2.查询Hbase版本：version
```
hbase(main):003:0> vesion
NameError: undefined local variable or method `vesion' for #<Object:0x52ac9a2c>
```
3.查看表描述
```
hbase(main):006:0> describe 'table1'
Table table1 is ENABLED                                                                                                  
table1                                                                                                                   
COLUMN FAMILIES DESCRIPTION                                                                                              
{NAME => 't1', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', VERSIONS => '1', COMPRESSIO
N => 'NONE', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_MEMORY => 'fa
lse', BLOCKCACHE => 'true'}                                                                                              
1 row(s) in 0.0290 seconds
```

### 增删改操作

1.创建一张表userinfo ：create 'userinfo','name','age'
```
hbase(main):011:0> create 'userinfo','name','age'
0 row(s) in 1.3220 seconds

=> Hbase::Table - userinfo
```
2.列出所有表：list
```
hbase(main):013:0> list
TABLE                                                                                                                    
table1                                                                                                           
userinfo                                                         `                                                   
2 row(s) in 0.0230 seconds

=> ["table1", "userinfo"]
```
3.获得表的描述：describe
```
hbase(main):014:0> describe 'userinfo'
Table userinfo is ENABLED                                                                                                
userinfo                                                                                                                 
COLUMN FAMILIES DESCRIPTION                                                                                              
{NAME => 'age', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', VERSIONS => '1', COMPRESSI
ON => 'NONE', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_MEMORY => 'f
alse', BLOCKCACHE => 'true'}                                                                                             
{NAME => 'name', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', VERSIONS => '1', COMPRESS
ION => 'NONE', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_MEMORY => '
false', BLOCKCACHE => 'true'}                                                                                            
2 row(s) in 0.0430 seconds
```
4.查看表是否存在：exists
```
hbase(main):017:0> exists 'table2'
Table table2 does not exist                                                                                            
0 row(s) in 0.0210 seconds
```
5.判断表是否为：‘enable’
```
hbase(main):018:0> is_enabled 'userinfo'
true 
```
6.判断表是否为：‘disable’
```
hbase(main):019:0> is_disabled 'userinfo'
false  
```
7.删除一个列族  disable alter enable
```
hbase(main):003:0> disable 'table1'
0 row(s) in 0.0230 seconds

hbase(main):004:0> alter 'table1',{ NAME => 't2' , METHOD => 'delete'}
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 2.2240 seconds

hbase(main):005:0> enable 'table1'
0 row(s) in 1.2990 seconds
```
8.删除一个表：drop ， 删除表前，需要先屏蔽该表。
```
hbase(main):007:0> disable 'table1'
0 row(s) in 2.2910 seconds

hbase(main):008:0> drop 'table1'
0 row(s) in 1.3030 seconds
```

9.往userinfo中插入几条记录：put ’表名‘，’行名‘，’列族名:xxx‘,'value'
```
hbase(main):016:0> put 'userinfo','row1','name:col1','Berg'
0 row(s) in 0.0870 seconds

```

10.全表扫描：scan
```
hbase(main):001:0> scan 'userinfo'
ROW                             COLUMN+CELL                                             
row1                           column=name:col1, timestamp=1463066505217, value=Berg                                    
 ```
 扫描部分数据:scan '表名称' , {COLUMNS => ['A'], LIMIT => 10}，扫描某列，10行数据
 
 ```
 scan 'tablename' , {COLUMNS => ['A'], LIMIT => 10}
 ```
 扫描出错的时候会告诉你相关例子：
 ```
 hbase> scan 'hbase:meta'
  hbase> scan 'hbase:meta', {COLUMNS => 'info:regioninfo'}
  hbase> scan 'ns1:t1', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'xyz'}
  hbase> scan 't1', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'xyz'}
  hbase> scan 't1', {COLUMNS => 'c1', TIMERANGE => [1303668804, 1303668904]}
  hbase> scan 't1', {REVERSED => true}
  hbase> scan 't1', {DEBUG => true}
  hbase> scan 't1', {FILTER => "(PrefixFilter ('row2') AND
    (QualifierFilter (>=, 'binary:xyz'))) AND (TimestampsFilter ( 123, 456))"}
 ```
 
11.获得某一行的所有数据，也是获得某一行名的所有数据
```
hbase(main):003:0> get 'userinfo', 'row1'
COLUMN                          CELL                                                                                     
 age:col2                       timestamp=1463066528653, value=22                                                        
 name:col1                      timestamp=1463066505217, value=Berg
 ```
 
12.获得某行，某列族，某列的所有数据

```
hbase(main):004:0> get 'userinfo','row1','name:col1'
COLUMN                          CELL                                                                                     
 name:col1                      timestamp=1463066505217, value=Berg                                                      
1 row(s) in 0.0190 seconds
```
13.更新一条记录 ： put（  name:col1的值更改为： BergBergBerg   ）
```
hbase(main):009:0> put 'userinfo','row2','name:col1','BergBergBerg'
0 row(s) in 0.0720 seconds
```
获取更新后的值： 

```
hbase(main):011:0> get 'userinfo','row2','name:col1'
COLUMN                          CELL                                                                                     
 name:col1                      timestamp=1463067259346, value=BergBergBerg                                              
1 row(s) in 0.0280 seconds
```
14.查询表中有多少行：count
```
hbase(main):012:0> count 'userinfo'
2 row(s) in 0.0450 seconds

=> 2
```
15.删除 某行 某列族的值：delete

```
hbase(main):023:0> delete 'userinfo','row2','age:col2'
0 row(s) in 0.0140 seconds

hbase(main):024:0> scan 'userinfo'
ROW                             COLUMN+CELL                                                                              
 row1                           column=age:col2, timestamp=1463066528653, value=22                                       
 row1                           column=name:col1, timestamp=1463066505217, value=Berg                                    
 row2                           column=name:col1, timestamp=1463067259346, value=BergBergBerg                            
2 row(s) in 0.0300 seconds
```
16.删除整行的值：deleteall
```
hbase(main):026:0> deleteall 'userinfo','row2'
0 row(s) in 0.0190 seconds

hbase(main):027:0> scan 'userinfo'
ROW                             COLUMN+CELL                                                                              
 row1                           column=age:col2, timestamp=1463066528653, value=22                                       
 row1                           column=name:col1, timestamp=1463066505217, value=Berg                                    
1 row(s) in 0.0320 seconds
```
17.给 row1 这行 age列，并使用counter实现递增 ： incr 
```
hbase(main):024:0> incr 'userinfo','row1','age:id'
COUNTER VALUE = 1
0 row(s) in 0.0170 seconds

hbase(main):025:0> incr 'userinfo','row1','age:id'
COUNTER VALUE = 2
0 row(s) in 0.0210 seconds

hbase(main):026:0> incr 'userinfo','row1','age:id'
COUNTER VALUE = 3
0 row(s) in 0.1270 seconds
```
 获取当前counter的值：

```
hbase(main):027:0> get_
get_auths     get_counter   get_splits    get_table
hbase(main):027:0> get_counter 'userinfo','row1','age:id'
COUNTER VALUE = 3
```
18.将整个表清空：truncate
```
hbase(main):028:0> truncate 'userinfo'
Truncating 'userinfo' table (it may take a while):
 - Disabling table...
 - Truncating table...
0 row(s) in 4.3360 seconds

hbase(main):029:0> scan 'userinfo'
ROW                             COLUMN+CELL                                                                              
0 row(s) in 0.3490 seconds
```

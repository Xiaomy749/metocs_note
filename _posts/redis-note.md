---
title: redis_note
date: 2021-11-21 00:06:51
tags: [Redis]
categories:
- [Redis]
---

# 													Redis 笔记

Redis 常用命令整理，归档，记录



## Redis 连接

### Echo 命令 - 打印字符串

```shell
127.0.0.1:6379> echo "hello world"
"hello world"
127.0.0.1:6379> 
```

### Select 命令 - 切换到指定的数据库

```shell
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> select 2
OK
127.0.0.1:6379[2]> select 3
OK
127.0.0.1:6379[3]> select 0
OK
127.0.0.1:6379> 

```

### Ping 命令 - 查看服务是否运行

如果连接正常就返回一个 PONG ，否则返回一个连接错误。

```shell
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> 

```

### Quit 命令 - 关闭当前连接

```shell
redis 127.0.0.1:6379> quit
```

### Auth 命令 - 验证密码是否正确



## Redis key

### 匹配并返回所有匹配的key

 KEYS    pattern          

查找所有符合给定模式pattern（正则表达式）的 key 

支持的正则表达模式：

- `h?llo` 匹配 `hello`, `hallo` 和 `hxllo`
- `h*llo` 匹配 `hllo` 和 `heeeello`
- `h[ae]llo` 匹配 `hello` 和 `hallo,` 但是不匹配 `hillo`
- `h[^e]llo` 匹配 `hallo`, `hbllo`, … 但是不匹配 `hello`
- `h[a-b]llo` 匹配 `hallo` 和 `hbllo`

如果你想取消字符的特殊匹配（正则表达式，可以在它的前面加`\`。

```shell
127.0.0.1:6379> keys * 				# 匹配所有
1) "test"
2) "mykey"
127.0.0.1:6379> 

```

### 删除一个或多个key

DEL     key [key ...]          

**时间复杂度：**O(N) 将要被删除的key的数量，当删除的key是字符串以外的复杂数据类型时比如List,Set,Hash删除这个key的时间复杂度是O(1)。

删除指定的一批keys，如果删除中的某些key不存在，则直接忽略。

```shell
127.0.0.1:6379> del mykey test						
(integer) 2											#  返回删除数量
127.0.0.1:6379> 
```

### 将指定key的值序列化

DUMP  [KEY]			

序列化给定 key ，并返回被序列化的值，使用 [RESTORE](http://www.redis.cn/commands/restore) 命令可以将这个值反序列化为 Redis 键。

返回值

如果 key 不存在，那么返回 nil。否则，返回序列化之后的值。

```shell
127.0.0.1:6379> set mykey hello
OK
127.0.0.1:6379> dump mykey
"\x00\x05hello\t\x00\xb3\x80\x8e\xba1\xb2C\xbb"
127.0.0.1:6379> 
```

### 反序列化值并关联给定key 

RESTORE  key  ttl   serialized-value [REPLACE]    

反序列化给定的序列化值，并将它和给定的 key 关联。

参数 ttl 以毫秒为单位为 key 设置生存时间；如果 ttl 为 0 ，那么不设置生存时间。

返回值

如果反序列化成功那么返回 OK ，否则返回一个错误

```shell
127.0.0.1:6379> er mykeyreserver 0 "\x00\x05hello\t\x00\xb3\x80\x8e\xba1\xb2C\xbb"
OK
127.0.0.1:6379> get mykeyreserver
"hello"
127.0.0.1:6379> 

```

###  判断key是否存在

EXISTS   key [key ...]          

- 1 如果key存在
- 0 如果key不存在

```shell
127.0.0.1:6379> exists mykey			
(integer) 1
127.0.0.1:6379> 
```

### 设置`key`的过期时间

EXPIRE   key  seconds          

设置`key`的过期时间，超过时间后，将会自动删除该`key`。在Redis的术语中一个`key`的相关超时是不确定的。

注意 redis 设置过期时间后的key 并不会真正的被删除，而是 超时后只有对`key`执行DEL命令或者SET命令或者GETSET命令时才会清除。 

这意味着，从概念上讲所有改变`key`的值的操作都会使他清除。

如果`key`被RENAME命令修改，相关的超时时间会转移到新`key`上面

如果`key`被RENAME命令修改，比如原来就存在`Key_A`,然后调用`RENAME Key_B Key_A`命令，这时不管原来`Key_A`是永久的还是设置为超时的，都会由`Key_B`的有效期状态覆盖。

**刷新过期时间**

对已经有过期时间的`key`执行`EXPIRE`操作，将会更新它的过期时间。

**返回值**

- `1` 如果成功设置过期时间。
- `0` 如果`key`不存在或者不能设置过期时间。



|        命令         |                             简介                             |
| :-----------------: | :----------------------------------------------------------: |
| expire  key（time） |        设置一个key的超时时间  精度可以使用毫秒或秒。         |
|       ttl key       | 查看当前key 的剩余时间    返回 剩余时间  秒数    -1  该key 没有设置超时时间  -2  没有这个key 或已过期 |



```shell
127.0.0.1:6379> set mykey 1						#	设置初始值
OK

127.0.0.1:6379> expire mykey 10					#  给这个key 设置 10秒的超时时间
(integer) 1


127.0.0.1:6379> ttl mykey
(integer) 5

127.0.0.1:6379> ttl mykey
(integer) -2


127.0.0.1:6379> set mykey 1						#	从重新设置 
OK
127.0.0.1:6379> ttl mykey						#	返回 -1  该key 没有设置超时时间
(integer) -1
```



EXPIREAT   key  timestamp          

[EXPIREAT](http://www.redis.cn/commands/expireat.html) 的作用和 [EXPIRE](http://www.redis.cn/commands/expire.html)类似，都用于为 key 设置生存时间。不同在于 [EXPIREAT](http://www.redis.cn/commands/expireat.html) 命令接受的时间参数是 UNIX 时间戳 Unix timestamp 。

```shell
127.0.0.1:6379> EXPIREAT mykey 1293840000				
(integer) 1
127.0.0.1:6379> exists mykey
(integer) 0
127.0.0.1:6379> 

```

### 将 key 原子性地从当前实例传送到目标实例的指定数据库上

MIGRATE host  port  key  destination-db  timeout   [COPY] | [REPLACE]

这个命令是一个原子操作，它在执行的时候会阻塞进行迁移的两个实例，直到以下任意结果发生：迁移成功，迁移失败，等到超时。

命令的内部实现是这样的：它在当前实例对给定 key 执行 [DUMP](http://www.redis.cn/commands/dump.html) 命令 ，将它序列化，然后传送到目标实例，目标实例再使用 [RESTORE](http://www.redis.cn/commands/restore.html) 对数据进行反序列化，并将反序列化所得的数据添加到数据库中；当前实例就像目标实例的客户端那样，只要看到 [RESTORE](http://www.redis.cn/commands/restore.html) 命令返回 OK ，它就会调用 [DEL](http://www.redis.cn/commands/del.html) 删除自己数据库上的 `key` 。

timeout 参数以毫秒为格式，指定当前实例和目标实例进行沟通的最大间隔时间。这说明操作并不一定要在 timeout 毫秒内完成，只是说数据传送的时间不能超过这个 timeout 数



### 将当前数据库的 key 移动到给定db 中

将当前数据库的 key 移动到给定的数据库 db 当中。

如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。

因此，也可以利用这一特性，将 `MOVE` 当作锁(locking)原语(primitive)。

### OBJECT

- OBJECT REFCOUNT 该命令主要用于调试(debugging)，它能够返回指定key所对应value被引用的次数.
- OBJECT ENCODING  该命令返回指定key对应value所使用的内部表示(representation)(译者注：也可以理解为数据的压缩方式).
- OBJECT IDLETIME  该命令返回指定key对应的value自被存储之后空闲的时间，以秒为单位(没有读写操作的请求) ，这个值返回以10秒为单位的秒级别时间，这一点可能在以后的实现中改善

```shell
redis> lpush mylist "Hello World"
(integer) 4
redis> object refcount mylist
(integer) 1
redis> object encoding mylist
"ziplist"
redis> object idletime mylist
(integer) 10
```

### 持久化key

PERSIST KEY

移除给定key的生存时间，将这个 key 从(带生存时间 key )转换成『持久的』(一个不带生存时间、永不过期的 key )。

```shell
127.0.0.1:6379> expire mykey 30				设置 30秒过期
(integer) 1
127.0.0.1:6379> ttl mykey 					查看剩余时间
(integer) 25
127.0.0.1:6379> persist mykey				持久化key
(integer) 1
127.0.0.1:6379> ttl mykey 					查看剩余时间
(integer) -1
127.0.0.1:6379> 
```



### 以毫秒设置key过期时间

PEXPIRE	KEY	milliseconds

### 以毫秒级时间戳设置key 的过期时间

PEXPIREAT	KEY	TIMESTAMP

### 以毫秒级查询指定key的剩余时间

PTTL 	KEY

### 随机获取一个key

RANDOMKEY

没有key 返回 nil  否则随机返回一个

### 重命名一个key

RENAME	KEY	newName

如果newName 与之前的 key 相同则返回错误

如果已将有这个key 了 就会覆盖已有的key的内容

### 仅当新名称不存在时重命名key

RENAMENX	key	newName

当且仅当 newkey 不存在时，将 key 改名为 newkey 。

当 key 不存在时，返回一个错误。







## Redis Strings

这是最简单Redis类型。如果你只用这种类型，Redis就像一个可以持久化的memcached服务器（注：memcache的数据仅保存在内存中，服务器重启后，数据将丢失）。



|     命令      |                          简介                          |
| :-----------: | :----------------------------------------------------: |
| set key value | 设定指定key的值，如果不存在该key则创建，如果存在会覆盖 |
|    get key    |                 通过指定的key获取value                 |



```shell
127.0.0.1:6379> set mykey somevalue
OK
127.0.0.1:6379> 
```

```shell
127.0.0.1:6379> get mykey
"somevalue"
127.0.0.1:6379> 
```

#### stirng 类型的基础操作

通常用SET 命令和 GET 命令来设置和获取字符串值。

值可以是任何种类的字符串（包括二进制数据），例如你可以在一个键下保存一副 jpeg 图片。值的长度不能超过512 MB。

值得注意的是，set 命令有一些可选参数，可以达到一些特殊效果。

```
// nx：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value
> set mykey newval nx
(nil)
// xx：只在键已经存在时，才对键进行设置操作。
> set mykey newval xx
OK
```

```shell
127.0.0.1:6379> set mykey newval nx     	# 之前已经设置过key  ：  mykey  因此 返回   nil  
(nil)

127.0.0.1:6379> get mykey					# 已经有了
"newval"
127.0.0.1:6379>

----------------------------------------------------------------------------------------------------------------------------------------

127.0.0.1:6379> get mykey1
(nil)
127.0.0.1:6379> 

127.0.0.1:6379> set mykey newval xx			# 之前存在过这个key  就会设置  不存在 返回  nil
OK

127.0.0.1:6379> set  mykey1 newal xx		# 之前不存在这个key
(nil)
127.0.0.1:6379> 

```

#### 原子自增

虽然字符串是Redis的基本值类型，但你仍然能通过它完成一些有趣的操作。例如：原子递增：

|        命令        |            简介            |
| :----------------: | :------------------------: |
|     incr  key      | 将值指定key    的值  自增1 |
| incrby  key  value | 将指定key的 值 增加  value |
|     decr  key      | 将值指定key    的值  自减1 |
| decrby  key  value | 将指定key的 值 减少 value  |



```shell
127.0.0.1:6379> set counter 100				#	设置原始数据
OK
127.0.0.1:6379> 				

127.0.0.1:6379> incr counter
(integer) 101
127.0.0.1:6379> incr counter
(integer) 102
127.0.0.1:6379> 

127.0.0.1:6379> incrby counter 50
(integer) 152
127.0.0.1:6379> incrby counter 100
(integer) 252
127.0.0.1:6379> 

——————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————————

127.0.0.1:6379> decr counter
(integer) 251
127.0.0.1:6379> 

127.0.0.1:6379> decrby counter 50
(integer) 201
127.0.0.1:6379> 
```

[INCR](http://www.redis.cn/commands/incr.html) 命令将字符串值解析成整型，将其加一，最后将结果保存为新的字符串值，类似的命令有[INCRBY](http://www.redis.cn/commands/incrby.html), [DECR](http://www.redis.cn/commands/decr.html) 和 [DECRBY](http://www.redis.cn/commands/decrby.html)。实际上他们在内部就是同一个命令，只是看上去有点儿不同。

[INCR](http://www.redis.cn/commands/incr.html)是原子操作意味着什么呢？就是说即使多个客户端对同一个key发出[INCR](http://www.redis.cn/commands/incr.html)命令，也决不会导致竞争的情况。例如如下情况永远不可能发生：『客户端1和客户端2同时读出“10”，他们俩都对其加到11，然后将新值设置为11』。最终的值一定是12，read-increment-set操作完成时，其他客户端不会在同一时间执行任何命令。

对字符串，另一个的令人感兴趣的操作是[GETSET](http://www.redis.cn/commands/getset.html)命令，行如其名：他为key设置新值并且返回原值。这有什么用处呢？例如：你的系统每当有新用户访问时就用[INCR](http://www.redis.cn/commands/incr.html)命令操作一个Redis key。你希望每小时对这个信息收集一次。你就可以[GETSET](http://www.redis.cn/commands/getset.html)这个key并给其赋值0并读取原值。

```shell
GETSET 命令

127.0.0.1:6379> set test testGetSet				#  设定初始值
OK
127.0.0.1:6379> 

127.0.0.1:6379> getset test newval				#	测试getset 命令  设置新值  返回 旧值
"testGetSet"
127.0.0.1:6379> get test						#   查看新值	
"newval"
127.0.0.1:6379> 
```

使用示例 ：你的系统每当有新用户访问时就用[INCR](http://www.redis.cn/commands/incr.html)命令操作一个Redis key。你希望每小时对这个信息收集一次。你就可以[GETSET](http://www.redis.cn/commands/getset.html)这个key并给其赋值0清空该值并读取原值。

#### 一次保存多条数据



|               命令               |                          简介                          |
| :------------------------------: | :----------------------------------------------------: |
| mset  key value key value ...... |                一次存储多个key对应的值                 |
| mget  key value key value ...... | 一次获取多个key对应的值  MGET 命令返回由值组成的数组。 |

```shell
127.0.0.1:6379> mset a 10 b 20 c 30						# 一次存储多个key对应的值
OK	
127.0.0.1:6379> keys * 									# 查看当前全部的 key
1) "a"
2) "b"
3) "c"

127.0.0.1:6379> mget a b c								# 一次获取多个key对应的值
1) "10"
2) "20"
3) "30"
127.0.0.1:6379> 
```

#### 修改或查询键空间

|    命令     |                             简介                             |
| :---------: | :----------------------------------------------------------: |
| exists  key |        使用EXISTS命令返回1或0标识给定key的值是否存在         |
|  del  key   | 可以删除key对应的值，返回1或0标识值是被删除(值存在)或者没被删除(key对应的值不存在)。 |



```shell
127.0.0.1:6379> set mykey hello							# 设置初始值
OK
127.0.0.1:6379> 

127.0.0.1:6379> del mykey								# 删除这个key
(integer) 1
127.0.0.1:6379> 
		
127.0.0.1:6379> exists mykey 							# 判断是否存在  
(integer) 0												# 返回零  已经不存在了
127.0.0.1:6379> 


127.0.0.1:6379> del mykey								# 再次测试删除
(integer) 0												# 返回零 删除失败没有这个key
127.0.0.1:6379> 

```



#### 返回指定key对应的值的类型

|   命令   |                             简介                             |
| :------: | :----------------------------------------------------------: |
| type key | [TYPE](http://www.redis.cn/commands/type.html)命令可以返回key对应的值的存储类型 |

```shell
127.0.0.1:6379> set mykey 1								#	设定一个初始值  使用set 命令 将value以string 类型保存
OK
127.0.0.1:6379> type mykey								#	获取指定key 对用的value 类型
string
127.0.0.1:6379> del mykey 								#	删除这个key 与其对用的value
(integer) 1
127.0.0.1:6379> type mykey								#	获取 之前被删除的key 现在的类型 （不准确）  已经不存在了 ..
none
127.0.0.1:6379> 

```





## Redis lists 

Redis lists基于Linked Lists实现。

### Lpush 	Rpush	插入元素

|              命令               |                 简介                 |
| :-----------------------------: | :----------------------------------: |
|        lpush  key  value        | 可向list的左边（头部）添加一个新元素 |
|        rpush  key  value        | 可向list的右边（尾部）添加一个新元素 |
| lrange  key  value  start   end |    可从list中取出一定范围的元素:     |

```shell
127.0.0.1:6379> rpush mylist A
(integer) 1
127.0.0.1:6379> rpush mylist B
(integer) 2
127.0.0.1:6379>  rpush mylist C
(integer) 3
127.0.0.1:6379> lpush mylist first
(integer) 4
127.0.0.1:6379> lrange mylist 0 -1					
1) "first"
2) "A"
3) "B"
4) "C"
127.0.0.1:6379> lrange mylist 0  1					
1) "first"
2) "A"
127.0.0.1:6379> 
```

注意:[LRANGE](http://www.redis.cn/commands/lrange.html) 带有两个索引，一定范围的第一个和最后一个元素。这两个索引都可以为负来告知Redis从尾部开始计数，因此-1表示最后一个元素，-2表示list中的倒数第二个元素，以此类推。

上面的所有命令的参数都可变，方便你一次向list存入多个值。

```shell
127.0.0.1:6379> rpush mylist 1 2 3 4 5 foo bar				#	加入多条数据		
(integer) 11								
127.0.0.1:6379> lrange mylist 0 -1		
 1) "first"
 2) "A"
 3) "B"
 4) "C"
 5) "1"
 6) "2"
 7) "3"
 8) "4"
 9) "5"
10) "foo"
11) "bar"
127.0.0.1:6379> 
```

### Lpop Rpop 删除元素

| 命令 |         简介         |
| :--: | :------------------: |
| rpop | 从右边第一个删除元素 |
| lpop | 从左边第一个删除元素 |



```shell
127.0.0.1:6379> rpop mylist
"bar"
127.0.0.1:6379> lpop mylist
"first"
127.0.0.1:6379> 

127.0.0.1:6379> lpop mylist							#	没有元素返回  nil
(nil)
```

### Lpushx 命令 - 将一个或多个值插入到已存在的列表头部

```shell
127.0.0.1:6379> lpushx mylist one
(integer) 3
127.0.0.1:6379> lrange mylist 0 -1
1) "one"
2) "redis"
3) "4"
127.0.0.1:6379> 

127.0.0.1:6379> lpushx mylist1 two
(integer) 0
127.0.0.1:6379> 

```

### Rpushx 命令 - 为已存在的列表添加值

```shell
127.0.0.1:6379> rpushx mylist two
(integer) 4
127.0.0.1:6379> lrange mylist 0 -1
1) "one"
2) "redis"
3) "4"
4) "two"
127.0.0.1:6379> 

127.0.0.1:6379> rpushx mylist1 two
(integer) 0
127.0.0.1:6379> 
```



### Lrange 命令  -  获取列表指定范围内的元素

Redis Lrange 返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。 其中 0 表示列表的第一个元素， 1  表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

```shell
127.0.0.1:6379> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
127.0.0.1:6379> 
```

### Lindex 命令 - 通过索引获取列表中的元素

Redis Lindex 命令用于通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

```shell
127.0.0.1:6379> lindex mylist 3
"4"
127.0.0.1:6379> 
```

### Lset 命令 - 通过索引设置列表元素的值

Redis Lset 通过索引来设置元素的值。

当索引参数超出范围，或对一个空列表进行 LSET 时，返回一个错误。

```shell
127.0.0.1:6379> lset mylist 2 new
OK
127.0.0.1:6379> lrange mylist 0 -1
1) "1"
2) "2"
3) "new"
4) "4"
5) "5"
6) "6"
7) "7"
127.0.0.1:6379> 
```

### Linsert 命令 - 在列表的元素前或者后插入元素

Redis Linsert 命令用于在列表的元素前或者后插入元素。 当指定元素不存在于列表中时，不执行任何操作。 当列表不存在时，被视为空列表，不执行任何操作。 如果 key 不是列表类型，返回一个错误。

```shell
127.0.0.1:6379> linsert mylist before new hello
(integer) 8
127.0.0.1:6379> lrange mylist 0 -1
1) "1"
2) "2"
3) "hello"
4) "new"
5) "4"
6) "5"
7) "6"
8) "7"
127.0.0.1:6379> 


127.0.0.1:6379> linsert mylist after new redis
(integer) 9
127.0.0.1:6379> 
127.0.0.1:6379> lrange mylist 0 -1
1) "1"
2) "2"
3) "hello"
4) "new"
5) "redis"
6) "4"
7) "5"
8) "6"
9) "7"
127.0.0.1:6379> 

```

### Llen 命令 -获取列表长度

Redis Llen 命令用于返回列表的长度。 如果列表 key 不存在，则 key 被解释为一个空列表，返回 0 。 如果 key 不是列表类型，返回一个错误。

```shell
127.0.0.1:6379> llen mylist
(integer) 9
127.0.0.1:6379> 
```

### Lrem 命令 - 移除列表元素

Redis Lrem 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。

COUNT 的值可以是以下几种：

- count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
- count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
- count = 0 : 移除表中所有与 VALUE 相等的值。

```shell
127.0.0.1:6379> lrange mylist 0 -1
 1) "test"
 2) "hello"
 3) "hello"
 4) "hello"
 5) "1"
 6) "2"
 7) "hello"
 8) "new"
 9) "redis"
10) "4"
11) "5"
12) "6"
13) "7"
14) "hello"
15) "hello"
16) "hello"
127.0.0.1:6379>

127.0.0.1:6379> lrem mylist -2 hello 
(integer) 2

127.0.0.1:6379> lrange mylist 0 -1
 1) "test"
 2) "hello"
 3) "hello"
 4) "hello"
 5) "1"
 6) "2"
 7) "hello"
 8) "new"
 9) "redis"
10) "4"
11) "5"
12) "6"
13) "7"
14) "hello"
127.0.0.1:6379> 

127.0.0.1:6379> lrem mylist 1 hello
(integer) 1
127.0.0.1:6379> lrange mylist 0 -1
 1) "test"
 2) "hello"
 3) "hello"
 4) "1"
 5) "2"
 6) "hello"
 7) "new"
 8) "redis"
 9) "4"
10) "5"
11) "6" 
12) "7"
13) "hello"
127.0.0.1:6379> 

127.0.0.1:6379> lrem mylist 0 hello
(integer) 4
127.0.0.1:6379> lrange mylist 0 -1
1) "test"
2) "1"
3) "2"
4) "new"
5) "redis"
6) "4"
7) "5"
8) "6"
9) "7"
127.0.0.1:6379> 
```

### Ltrim 命令 - 对一个列表进行修剪

Redis Ltrim 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

下标 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

```shell
127.0.0.1:6379> lrange mylist 0 -1
1) "test"
2) "1"
3) "2"
4) "new"
5) "redis"
6) "4"
7) "5"
127.0.0.1:6379> 

127.0.0.1:6379> lrange mylist 0 -1
1) "new"
2) "redis"
3) "4"
4) "5"
127.0.0.1:6379> 
```

### Blpop 命令 - 移出并获取列表的第一个元素

Redis Blpop 命令移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

```shell
127.0.0.1:6379> BLPOP mylist 100
1) "mylist"												# key
2) "new"												# 弹出的元素
```

### Brpop 命令 - 移出并获取列表的第一个元素

```shell
127.0.0.1:6379> brpop mylist 100
1) "mylist"
2) "5"
127.0.0.1:6379> 
```

### Rpoplpush 命令 - 移除列表的最后一个元素，并将该元素添加到另一个列表并返回

```shell
127.0.0.1:6379> lrange mylist 0 -1
1) "one"
2) "redis"
3) "4"
4) "two"
127.0.0.1:6379> rpoplpush mylist myotherist
"two"

127.0.0.1:6379> lrange myotherist 0 -1
1) "two"
127.0.0.1:6379> lrange mylist 0 -1
1) "one"
2) "redis"
3) "4"
127.0.0.1:6379> 
```

### Brpoplpush 命令 - 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回 

如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

```shell
127.0.0.1:6379> brpoplpush myotherst mylist 10			
# myotherst 无元素  客户端阻塞

127.0.0.1:6379> brpoplpush myotherst mylist 10
(nil)
(10.09s)
127.0.0.1:6379> 

```

## Redis hash

Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。



###  Hset 命令 - 将哈希表 key 中的字段 field 的值设为 value 

```shell
127.0.0.1:6379> hset myhash name metocs
(integer) 1
127.0.0.1:6379> 
```

### Hget 命令 - 获取存储在哈希表中指定字段的值

```shell
127.0.0.1:6379> hget myhash name
"metocs"
127.0.0.1:6379> 
```

### Hsetnx 命令 - 只有在字段 field 不存在时，设置哈希表字段的值

```shell
127.0.0.1:6379> hsetnx myhash sex 0
(integer) 1
127.0.0.1:6379> hsetnx myhash name 0
(integer) 0
127.0.0.1:6379> 
```

### Hmset 命令 - 同时将多个 field-value (域-值)对设置到哈希表 key 中

```shell
127.0.0.1:6379> hmset myhash age 25 phone 1761000000
OK
127.0.0.1:6379> 
```

### Hmget 命令 - 获取所有给定字段的值

```shell
127.0.0.1:6379> hmget myhash name age phone
1) "metocs"
2) "25"
3) "1761000000"
127.0.0.1:6379> 
```

### Hkeys 命令 - 获取所有哈希表中的字段

```shell
127.0.0.1:6379> hkeys myhash
1) "name"
2) "age"
3) "phone"
127.0.0.1:6379> 
```

### Hvals 命令 - 获取哈希表中所有值

```shell
127.0.0.1:6379> hvals myhash
1) "metocs"
2) "25"
3) "1761000000"
127.0.0.1:6379> 
```

### Hgetall 命令 - 获取在哈希表中指定 key 的所有字段和值

```shell
127.0.0.1:6379> hgetall myhash
1) "name"
2) "metocs"
3) "age"
4) "25"
5) "phone"
6) "1761000000"
127.0.0.1:6379> 
```

### Hexists 命令 - 查看哈希表 key 中，指定的字段是否存在

```shell
127.0.0.1:6379> hexists myhash name
(integer) 1
127.0.0.1:6379> hexists myhash no
(integer) 0
127.0.0.1:6379> 
```

### Hlen 命令 - 获取哈希表中字段的数量

```shell
127.0.0.1:6379> hlen myhash
(integer) 4
127.0.0.1:6379> 

127.0.0.1:6379> hkeys myhash
1) "name"
2) "age"
3) "phone"
4) "sex"
127.0.0.1:6379> 
```

### Hdel 命令 - 删除一个或多个哈希表字段

```shell
127.0.0.1:6379> hkeys myhash
1) "name"
2) "age"
3) "phone"
4) "sex"
127.0.0.1:6379> hdel myhash name phone
(integer) 2
127.0.0.1:6379> hkeys myhash
1) "age"
2) "sex"
127.0.0.1:6379> 
```

### Hincrby 命令 - 为哈希表 key 中的指定字段的整数值加上增量 increment 

Redis Hincrby 命令用于为哈希表中的字段值加上指定增量值。

增量也可以为负数，相当于对指定字段进行减法操作。

如果哈希表的 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令。

如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。

对一个储存字符串值的字段执行 HINCRBY 命令将造成一个错误。

本操作的值被限制在 64 位(bit)有符号数字表示之内。

```shell
127.0.0.1:6379> hincrby myhash age -10
(integer) 15
127.0.0.1:6379> hgetall myhash
1) "age"
2) "15"
3) "sex"
4) "0"
127.0.0.1:6379> 
```

### Redis Hincrbyfloat 命令 - 为哈希表 key 中的指定字段的浮点数值加上增量 increment 

```shell
127.0.0.1:6379> hincrbyfloat myhash age 0.1
"15.1"
127.0.0.1:6379> hgetall myhash
1) "age"
2) "15.1"
3) "sex"
4) "0"
127.0.0.1:6379> 
```

## Redis set

Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。



### Sadd 命令 - 向集合添加一个或多个成员



Redis Sadd 命令将一个或多个成员元素加入到集合中，已经存在于集合的成员元素将被忽略。

假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合。

当集合 key 不是集合类型时，返回一个错误。

```shell
127.0.0.1:6379> sadd myset 1 2 3 4 5 6 7 8 9 
(integer) 9
127.0.0.1:6379> 
```

### Scard 命令 - 获取集合的成员数

```shell
127.0.0.1:6379> scard myset
(integer) 9
127.0.0.1:6379> 
```

### Smembers 命令 - 返回集合中的所有成员

```shell
127.0.0.1:6379> smembers myset
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
9) "9"
127.0.0.1:6379> 
```

### Srandmember 命令 - 返回集合中一个或多个随机数

```
127.0.0.1:6379> srandmember myset
"7"
127.0.0.1:6379> 
```

### Sismember 命令 - 判断 member 元素是否是集合 key 的成员

```shell
127.0.0.1:6379> sismember myset 2
(integer) 1
127.0.0.1:6379> 
```

### Srem 命令 - 移除集合中一个或多个成员

```shell
127.0.0.1:6379> srem myset 1
(integer) 1
127.0.0.1:6379> smembers myset
1) "2"
2) "3"
3) "4"
4) "5"
5) "6"
6) "7"
7) "8"
8) "9"
127.0.0.1:6379> 
```

###  Spop 命令 - 移除并返回集合中的一个随机元素

```shell
127.0.0.1:6379> spop myset
"6"
127.0.0.1:6379> 
```

### Smove 命令 - 将 member 元素从 source 集合移动到 destination 集合

```shell
127.0.0.1:6379> smove myset2 myset 1
(integer) 1
127.0.0.1:6379> 
```

### Sinter 命令 - 返回给定所有集合的交集

```shell
127.0.0.1:6379> sadd myset2 1 2 3 4 5 
(integer) 5
127.0.0.1:6379> sinter myset myset2
1) "2"
2) "3"
3) "4"
4) "5"
127.0.0.1:6379> 
```

### Sinterstore 命令 - 返回给定所有集合的交集并存储在 destination 中

Redis Sinterstore 命令将给定集合之间的交集存储在指定的集合中。如果指定的集合已经存在，则将其覆盖。

```shell
127.0.0.1:6379> sinterstore newset myset myset2
(integer) 4
127.0.0.1:6379> smembers newset
1) "2"
2) "3"
3) "4"
4) "5"
127.0.0.1:6379> 
```

### Sdiff 命令 - 返回给定所有集合的差集

```shell
127.0.0.1:6379> sdiff myset newset
1) "1"
2) "7"
3) "8"
4) "9"
127.0.0.1:6379> 
```

### Sdiffstore 命令 - 返回给定所有集合的差集并存储在 destination 中

```shell
127.0.0.1:6379> sdiffstore diifset myset newset
(integer) 4
127.0.0.1:6379> smembers diifset
1) "1"
2) "7"
3) "8"
4) "9"
127.0.0.1:6379> 
```

### Sunion 命令 - 返回所有给定集合的并集

```shell
127.0.0.1:6379> smembers newset
1) "2"
2) "3"
3) "4"
4) "5"
127.0.0.1:6379> smembers myset
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "7"
7) "8"
8) "9"
127.0.0.1:6379> sadd myset 10
(integer) 1
127.0.0.1:6379> sunion myset newset
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "7"
7) "8"
8) "9"
9) "10"
127.0.0.1:6379> 

```

### Sunionstore 命令 - 所有给定集合的并集存储在 destination 集合中

```shell
127.0.0.1:6379> sunionstore addset  myset newset
(integer) 9
127.0.0.1:6379> smembers addset
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "7"
7) "8"
8) "9"
9) "10"
127.0.0.1:6379> 
```

### Sscan 命令 - 迭代集合中的元素

```shell
 SSCAN KEY [MATCH pattern] [COUNT count]
```

## Redis sorted set

### Zadd命令 - 向有序集合添加一个或多个成员，或者更新已存在成员的分数

```shell
redis 127.0.0.1:6379> ZADD KEY_NAME SCORE1 VALUE1.. SCOREN VALUEN

127.0.0.1:6379> zadd myzset  1 hello 1 world 
(integer) 2
127.0.0.1:6379> 
```

### Zcount 命令 - 计算在有序集合中指定区间分数的成员数

```shell
redis 127.0.0.1:6379> ZCOUNT key min max

127.0.0.1:6379> zcount myzset 1 2
(integer) 4
127.0.0.1:6379> 
```

### Zrange 命令 - 通过索引区间返回有序集合成指定区间内的成员

```shell
redis 127.0.0.1:6379> ZRANGE key start stop [WITHSCORES]

127.0.0.1:6379> zrange myzset  0  -1  
1) "hello"
2) "world"
3) "hi"
4) "metocs"
5) "test"
```

### Zcard 命令 - 获取有序集合的成员数

```shell
redis 127.0.0.1:6379> ZCARD KEY_NAME


127.0.0.1:6379> zcard myzset 
(integer) 5
127.0.0.1:6379> 
```



###  Zrevrange 命令 - 返回有序集中指定区间内的成员，通过索引，分数从高到底

```shell
redis 127.0.0.1:6379> ZREVRANGE key start stop [WITHSCORES]

127.0.0.1:6379> zrevrange myzset 0 -1 WITHSCORES
 1) "test"
 2) "3"
 3) "metocs"
 4) "2"
 5) "hi"
 6) "2"
 7) "world"
 8) "1"
 9) "hello"
10) "1"
127.0.0.1:6379> 
```

### Zrevrank 命令 - 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序



### Zrank 命令 - 返回有序集合中指定成员的索引

```shell
redis 127.0.0.1:6379> ZRANK key member

127.0.0.1:6379> zrevrange myzset 0 -1 WITHSCORES
 1) "test"
 2) "3"
 3) "metocs"
 4) "2"
 5) "hi"
 6) "2"
 7) "world"
 8) "1"
 9) "hello"
10) "1"
127.0.0.1:6379> zrank myzset hello
(integer) 0
127.0.0.1:6379> zrank myzset hi
(integer) 2
127.0.0.1:6379> zrank myzset test
(integer) 4
127.0.0.1:6379> zrank myzset metocs
(integer) 3
127.0.0.1:6379> zrank myzset world
(integer) 1
127.0.0.1:6379> 
```

### Zscore 命令 - 返回有序集中，成员的分数值

```shell
redis 127.0.0.1:6379> ZSCORE key member

127.0.0.1:6379> zscore myzset world
"1"
127.0.0.1:6379> 
```

### Redis Zrem 命令 - 移除有序集合中的一个或多个成员



**未完待续**

## Redis 持久化

Redis 提供了一系列不同的持久性选项： 

- **RDB** （Redis 数据库）：RDB 持久性以指定的时间间隔执行数据集的时间点快照。 
- **AOF** （Append Only File）：AOF 持久化记录服务器收到的每个写操作，在服务器启动时会再次播放，重建原始数据集。 命令使用与 Redis 协议本身相同的格式以仅附加的方式记录。 当日志变得太大时，Redis 能够在后台重写日志。 
- **无持久性** ：如果您希望数据在服务器运行时一直存在，您可以完全禁用持久性。 
- **RDB + AOF** ：可以在同一个实例中组合 AOF 和 RDB。 请注意，在这种情况下，当 Redis 重新启动时，AOF 文件将用于重建原始数据集，因为它保证是最完整的。 

要理解的最重要的事情是不同的权衡             RDB 和 AOF 持久化。  让我们从 RDB 开始： 






























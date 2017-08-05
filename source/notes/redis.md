---
title: 《The Little Redis Book》笔记
type: notes
order: 3
category: notes
---

参考：[极客学院](http://wiki.jikexueyuan.com/project/the-little-redis-book/)

简而言之，一种可持久化的key-value内存存储。redis的各个数据库相对独立，使用数字编号进行区分，默认编号是`0`，通过`select 1`切换。切换后CLI提示符会显示成`redis 127.0.0.1:6379[1]>`

*关键字key*

- 标识数据块
- 常用`:`分隔符来组织关键字，如`users:hzlu`

*值values*

- redis把值看作一个**字节序列**，并不关心它实际上是什么
- 对redis而言，关键字就是一切，值没有任何意义，**不允许通过值来查询**。

*持久化*

- 默认下，redis根据已变更的key数量进行判断，然后在磁盘里创建数据库的快照(snapshot)，存储名为`dump.rdb`文件中
- 默认下，如果多于1000个key变更，每隔60秒存一次；或少于9个变更，每隔15分钟存一次。

## 安装使用

先去安装运行redis，在osx下直接`brew install redis`

启动服务`redis-server`，默认配置文件在`/usr/local/etc/redis.conf`里

启动redis控制台`redis-cli`，默认分配的端口为`6379`，在控制台下输入`info`可以得到服务器的大量信息。

```
在控制台下获取相关配置信息
CONFIG GET <setting-name>
CONFIG GET *									#获取全部配置
CONFIG SET <setting-name> <setting-value>		#编辑配置
```

## 数据类型(五种)

每个命令都对应一种特定的数据结构，当我们使用set时，redis就知道我们在使用字符串；当使用hset时，它就知道是要用哈希结构。

### 字符串string

二进制安全的，可以包含任何数据，比如JPG图片或序列化对象，一个键最大能存储512MB

```
set <key> <value>
get <key>
strlen <key>	#获取key对应值的长度
getrange <key> <from> <to> #对应值切片
append <key> <value> #把value附加，如果key不存在会创建
incr <key>		#当key对应的值不是整数时会出错
incrby <key> 5	#增加5
decr <key>
decrby <key> <step>
```

### 散列hash
Redis hash是一个string类型的field和value的映射表。

散列数据结构很像字符串数据结构，区别在于散列数据提供了一个额外的间接层 ： 域field

```
hmset user:1 username foo password bar points baz
#hmset的m表示multiple，一次设置多个域
hgetall user:1			#user:1为键值，每个hash可以存储2^32-1个域
hkeys <key>		#返回所有的域
hdel <key> <field>	#删除散列特定域
```

### 列表list
简单的字符串列表

```
lpush <key> <element>
lrange <key> <start> <stop>
ltrim <key> <start> <stop> #删除指定范围外的值，只留下范围内的值
#线性时间复杂度O(N)，N表示要删除的元素个数
```

### 集合set
string无序集合，集合是通过哈希表实现的

```
sadd <key> <member1> <member2> ...
smembers <key>	#返回集合元素
sismember <key> <member>	#判断是否在集合中，是1，否0
#时间复杂度为O(1)，最快速的
sinter <key1> <key2>	#并集
sinterstore <newkey> <key1> <key2> #储存并集结果
```

### 分类集合sorted set
string类型的有序集合，有序集合中的每个元素都关联了一个double类型的**score**，redis通过**score**来进行排序。成员唯一，score可重复

```
zadd <key> <score1> <member1> <score2> <member2> ...
#时间复杂度为O(log(N))，第二快速
```
秩rank，根据score对member排序后的次序，就是秩

```
zrangebyscore <key> <start-score> <end-score>
zremrangebyscore <key> <start> <end>
zcount <key> <from> <to>	#计算出标记为from到to的人数
zrevrank <key> <member>		#获取成员从高到低的秩
zrank <key> <member>		#从低到高的秩
```

## 仿多关键字查询
redis没有提供关键字链接功能，或者称为关键字别名，例如通过key1,key2都可以查询到相同的数据块，redis是拒绝的。

那怎么实现呢？

- 把相同的数据用不同的key都存一遍。糟糕的办法，产生双倍内存！
- 散列

```
#实际用户信息
set users:6379 "{id: 9001, email: 6379@gmail.com}"
#使用域作为一个二级索引
hset users:lookup:email 6379@gmail.com 6379
```
用ruby

```
id = redis.hget("users:lookup:email", "6379@gmail.com")
user = redis.get("user:#{id}")
```


## 键命令

命令(可小写) | 作用
---- | ----
DEL key | key存在时删除key
DUMP key | 序列化给定key，并返回序列化的值
EXISTS key | 检查key是否存在
EXPIRE key seconds | 设置过期时间，单位秒
EXPIREAT key timestamp | 设置过期时间，时间戳参数
PEXPIRE key milliseconds | 设置过期时间，毫秒
KEYS pattern |查找给定模式的key
MOVE key db | 把当前数据库的key移到数据库db中
PERSIST key | 持久化key
PTTL key | 返回剩余生存时间，毫秒
TTL key | 返回剩余生存时间，秒
RANDOMKEY | 随机返回一个key
RENAME key newkey | 改名
RENAMENX key newkey | 仅当newkey不存在时才把key改名
TYPE key | 返回类型

## 基数统计
Redis HyperLogLog是用来做基数统计的算法，但只会根据输入元素来计算基数，而不会存储元素本身。

```
PFADD key foo	#添加元素到hyperloglog中
pfadd key bar
pfadd key baz
pfadd key foo
pfcount key		#返回基数3，不重复元素的个数
```

## Redis发布订阅
pub/sub是一种消息通信模式，发送者pub发送消息，订阅者sub接收消息。

redis客户端可以订阅任意数量的频道，客户端就是订阅者，频道就是发布者

```
subscribe <channel>		#订阅频道channel
unsubscribe <channel>		#退订频道channel
publish <channel> <message> #向频道发布信息message
```

## 事务
###redis实际上是*单线程*运行的，这就是为什么每个redis命令能够保证具有原子性。

虽然redis是单线程运行，但我们可以同时运行多个redis客户端进程，常见的并发问题还是会出现
不过可以对key增加`watch`调用，如果另一个客户端改变了key的值，我们的事务将会失败。

```
MULTI		#开始事务
command1	#命令入队
command2
command3
EXEC		#执行事务
DISCARD		#取消事务，放弃事务块内的所有命令
```
```
redis.watch('powerlevel')
current = redis.get("powerlevel")
redis.multi()
redis.set("powerlevel", current + 1)
redis.exec()
```

在调用`watch`后，如果另一个客户端改变了`powerlevel`的值，事务将会运行失败。

### 监控和延迟日志
### 监控
`monitor`

设置redis去记录所有命令

```
config set slowlog-log-slower-than 0
```

执行命令后检索日志

```
slowlog get			#检索所有日志
slowlog get 10		#检索最近10个日志
slowlog len			#获取日志数量
```

### 对值进行排序sort命令
可以在一个列表、集合或者分类集合里对值进行排序。注意：分类集合是通过标记来进行排序，而不是集合里的成员。

```
rpush users:leto:guesses 5 9 10 2 3 4 19
sort users:leto:guesses #返回升序排序后的值
sadd friends:ghanima leto paul chani jessica alia duncan
#limit分页，desc降序，alpha按字母排
sort friends:ghanima limit 0 3 desc alpha
------------
sadd watch:leto 12339 1382 338 9338
set severity:12339 3
set severity:1382 2
set severity:338 5
set severity:9338 4
sort watch:leto by severity:* desc
根据散列的域排序
hset bug:12339 severity 3
hset bug:12339 priority 1
hset bug:12339 details "{id: 12339, ....}"
另一个散列
hset bug:1382 severity 2
hset bug:1382 priority 2
hset bug:1382 details "{id: 1382, ....}"
->用来查看散列中指定的域，get会进行值替代和域查看,store存储输出，类型为list
sort watch:leto by bug:*->priority get bug:*->details store watch_by_priority:leto
```

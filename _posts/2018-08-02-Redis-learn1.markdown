---
layout: post
title: "Redis数据库简介"
img: bowen49.jpg
date: 2018-08-02 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Redis]
---


## Redis 简介

+ Redi是一个高性能的key-value数据库

+ Redis支持数据额持久化，可以将内存中的数据保存在磁盘，重启的时候再次加载进行使用

+ Redis不仅支持简单的key-value类型的数据，同时还提供了list,set，zset,hash等数据结构的存储

+ Redis支持数据的备份，即master-slave模式的数据备份

** Redis的优势**

+ 读写性能极高
+ 支持丰富的数据类型
+ 所有操作时原子性的（要么成功执行，要么失败完全不执行）
+ Redis运行在内存中但是可以持久化到磁盘，所以在对不听数据集进行高速读写时需要权衡内存。


## Redis的安装

Windows下安装：


1. 下载对应版本的压缩安装包，下载地址<https://github.com/MSOpenTech/redis/releases>
2. 解压到C盘,将文件重命名为redis
3. 打开一个 cmd 窗口 使用cd命令切换目录到 C:\redis 运行 redis-server.exe redis.windows.conf 。如果想方便的话，可以把 redis 的路径加到系统的环境变量里，这样就省得再输路径了，后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的
4. 这时候另启一个cmd窗口，原来的不要关闭，不然就无法访问服务端了。
切换到redis目录下运行 redis-cli.exe -h 127.0.0.1 -p 6379 。设置键值对 set myKey abc，取出键值对 get myKey

Linux下安装

+ 1.下载地址<http://redis.io/download>

```
$ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
$ tar xzf redis-2.8.17.tar.gz
$ cd redis-2.8.17
$ make
```

+ 2.make完后 redis-2.8.17目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下
+ 3.下面启动redis服务

```
$ cd src
$ ./redis-server
```

## Redis配置

Redis 的配置文件位于 Redis 安装目录下，文件名为 redis.conf。
你可以通过 CONFIG 命令查看或设置配置项。

参数说明：

redis.conf 配置项说明如下：
1.Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
    daemonize no

2.当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
    pidfile /var/run/redis.pid

3.指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
    port 6379

4.绑定的主机地址
    bind 127.0.0.1

5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
    timeout 300

6.指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
    loglevel verbose

7.日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
    logfile stdout

8.设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
    databases 16

9.指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

```
    save <seconds> <changes>
    Redis默认配置文件中提供了三个条件：
    save 900 1
    save 300 10
    save 60 10000
    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
```
 
10.指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
    rdbcompression yes

11.指定本地数据库文件名，默认值为dump.rdb
    dbfilename dump.rdb

12.指定本地数据库存放目录
    dir ./

13.设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
    slaveof <masterip> <masterport>

14.当master服务设置了密码保护时，slav服务连接master的密码
    masterauth <master-password>

15.设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
    requirepass foobared

16.设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
    maxclients 128

17.指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
    maxmemory <bytes>

18.指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
    appendonly no

19.指定更新日志文件名，默认为appendonly.aof
     appendfilename appendonly.aof

20.指定更新日志条件，共有3个可选值： 
```
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折衷，默认值）
    appendfsync everysec
```

21.指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
     vm-enabled no

22.虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
     vm-swap-file /tmp/redis.swap

23.将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
     vm-max-memory 0

24.Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
     vm-page-size 32

25.设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
     vm-pages 134217728

26.设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
     vm-max-threads 4

27.设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    glueoutputbuf yes

28.指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
```
    hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
```

29.指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
    activerehashing yes

30.指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    include /path/to/local.conf


## Redis数据类型

Redis支持五种数据类型：string,hash,list,set以及zset(sorted set:有序集合)

**String**

```
redis 127.0.0.1:6379> SET name "runoob"
OK
redis 127.0.0.1:6379> GET name
"runoob"

```
在以上实例中我们使用了 Redis 的 SET 和 GET 命令。键为 name，对应的值为 runoob。

注意：一个键最大能存储512MB

**Hash**

Redis hash 是一个键值(key=>value)对集合。
Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

```
redis> HMSET myhash field1 "Hello" field2 "World"
"OK"
redis> HGET myhash field1
"Hello"
redis> HGET myhash field2
"World"
```

实例中我们使用了 Redis HMSET, HGET 命令，HMSET 设置了两个 field=>value 对, HGET 获取对应 field 对应的 value。
每个 hash 可以存储 232 -1 键值对（40多亿）。

**List**

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

```

redis 127.0.0.1:6379> lpush runoob redis
(integer) 1
redis 127.0.0.1:6379> lpush runoob mongodb
(integer) 2
redis 127.0.0.1:6379> lpush runoob rabitmq
(integer) 3
redis 127.0.0.1:6379> lrange runoob 0 10
1) "rabitmq"
2) "mongodb"
3) "redis"
redis 127.0.0.1:6379>

```

列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。


**Set**

Redis的Set是string类型的无序集合。
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

sadd 命令:
添加一个 string 元素到 key 对应的 set 集合中，成功返回1，如果元素已经在集合中返回 0，如果 key 对应的 set 不存在则返回错误。

```
sadd key member

redis 127.0.0.1:6379> sadd runoob redis
(integer) 1
redis 127.0.0.1:6379> sadd runoob mongodb
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 0
redis 127.0.0.1:6379> smembers runoob

1) "redis"
2) "rabitmq"
3) "mongodb"
```

注意：以上实例中 rabitmq 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。
集合中最大的成员数为 232 - 1(4294967295, 每个集合可存储40多亿个成员)。

**zset**

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
zset的成员是唯一的,但分数(score)却可以重复。

zadd 命令:
添加元素到集合，元素在集合中存在则更新对应score

```
zadd key score member 

redis 127.0.0.1:6379> zadd runoob 0 redis
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 mongodb
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 0
redis 127.0.0.1:6379> > ZRANGEBYSCORE runoob 0 1000
1) "mongodb"
2) "rabitmq"
3) "redis"
```


## Redis命令

Redis 命令用于在 redis 服务上执行操作。
要在 redis 服务上执行命令需要一个 redis 客户端。Redis 客户端在我们之前下载的的 redis 的安装包中。

语法

Redis 客户端的基本语法为：

```
$ redis-cli
```

实例

以下实例讲解了如何启动 redis 客户端：
启动 redis 客户端，打开终端并输入命令 redis-cli。该命令会连接本地的 redis 服务。

```
$redis-cli
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING

PONG
```

在以上实例中我们连接到本地的 redis 服务并执行 PING 命令，该命令用于检测 redis 服务是否启动。

**在远程服务器上执行命令**


如果需要在远程 redis 服务上执行命令，同样我们使用的也是 redis-cli 命令。

语法

```
$ redis-cli -h host -p port -a password
```

实例

以下实例演示了如何连接到主机为 127.0.0.1，端口为 6379 ，密码为 mypass 的 redis 服务上。

```
$redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING

PONG
```

## Redis键（key)

Redis 键命令用于管理 redis 的键。

语法

Redis 键命令的基本语法如下：

```
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

实例

```
redis 127.0.0.1:6379> SET runoobkey redis
OK
redis 127.0.0.1:6379> DEL runoobkey
(integer) 1
```

在以上实例中 DEL 是一个命令， runoobkey 是一个键。 如果键被删除成功，命令执行后输出 (integer) 1，否则将输出 (integer) 0

**Redis keys命令**

1.DEL key
该命令用于在 key 存在时删除 key。

2.DUMP key 
序列化给定 key ，并返回被序列化的值。

3.EXISTS key 
检查给定 key 是否存在。

4.EXPIRE key seconds
为给定 key 设置过期时间。

5.EXPIREAT key timestamp 
EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。

6.PEXPIRE key milliseconds 
设置 key 的过期时间以毫秒计。

7.PEXPIREAT key milliseconds-timestamp 
设置 key 过期时间的时间戳(unix timestamp) 以毫秒计

8.KEYS pattern 
查找所有符合给定模式( pattern)的 key 。

9.MOVE key db 
将当前数据库的 key 移动到给定的数据库 db 当中。

10.PERSIST key 
移除 key 的过期时间，key 将持久保持。

11.PTTL key 
以毫秒为单位返回 key 的剩余的过期时间。

12.TTL key 
以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。

13.RANDOMKEY 
从当前数据库中随机返回一个 key 。

14.RENAME key newkey 
修改 key 的名称

15.RENAMENX key newkey 
仅当 newkey 不存在时，将 key 改名为 newkey 。

16.TYPE key 
返回 key 所储存的值的类型。

## Redis 字符串(String)

Redis 字符串数据类型的相关命令用于管理 redis 字符串值，基本语法如下：

语法

```
redis 127.0.0.1:6379> COMMAND KEY_NAME
```

实例

```
redis 127.0.0.1:6379> SET runoobkey redis
OK
redis 127.0.0.1:6379> GET runoobkey
"redis"
```

在以上实例中我们使用了 SET 和 GET 命令，键为 runoobkey。
Redis 字符串命令

下表列出了常用的 redis 字符串命令：


1.SET key value 
设置指定 key 的值

2.GET key 
获取指定 key 的值。

3.GETRANGE key start end 
返回 key 中字符串值的子字符

4.GETSET key value
将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

5.GETBIT key offset
对 key 所储存的字符串值，获取指定偏移量上的位(bit)。

6.MGET key1 [key2..]
获取所有(一个或多个)给定 key 的值。

7.SETBIT key offset value
对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

8.SETEX key seconds value
将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。

9.SETNX key value
只有在 key 不存在时设置 key 的值。

10.SETRANGE key offset value
用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。

11.STRLEN key
返回 key 所储存的字符串值的长度。

12.MSET key value [key value ...]
同时设置一个或多个 key-value 对。

13	MSETNX key value [key value ...] 
同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

14.PSETEX key milliseconds value
这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。

15.INCR key
将 key 中储存的数字值增一。

16.INCRBY key increment
将 key 所储存的值加上给定的增量值（increment） 。

17.INCRBYFLOAT key increment
将 key 所储存的值加上给定的浮点增量值（increment） 。

18.DECR key
将 key 中储存的数字值减一。

19.DECRBY key decrement
key 所储存的值减去给定的减量值（decrement） 。

20.APPEND key value
如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。

## Redis 哈希(Hash)

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

实例

```
127.0.0.1:6379>  HMSET runoobkey name "redis tutorial" description 
"redis basic commands for caching" likes 20 visitors 23000
OK
127.0.0.1:6379>  HGETALL runoobkey
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```

在以上实例中，我们设置了 redis 的一些描述信息(name, description, likes, visitors) 到哈希表的 runoobkey 中。

Redis hash 命令

下表列出了 redis hash 基本的相关命令：

1.HDEL key field1 [field2] 
删除一个或多个哈希表字段

2.HEXISTS key field 
查看哈希表 key 中，指定的字段是否存在。

3.HGET key field 
获取存储在哈希表中指定字段的值。

4.HGETALL key 
获取在哈希表中指定 key 的所有字段和值

5.HINCRBY key field increment 
为哈希表 key 中的指定字段的整数值加上增量 increment 。

6.HINCRBYFLOAT key field increment 
为哈希表 key 中的指定字段的浮点数值加上增量 increment 。

7.HKEYS key 
获取所有哈希表中的字段

8.HLEN key 
获取哈希表中字段的数量

9.HMGET key field1 [field2] 
获取所有给定字段的值

10.HMSET key field1 value1 [field2 value2 ] 
同时将多个 field-value (域-值)对设置到哈希表 key 中。

11.HSET key field value 
将哈希表 key 中的字段 field 的值设为 value 。

12.HSETNX key field value 
只有在字段 field 不存在时，设置哈希表字段的值。

13.HVALS key 
获取哈希表中所有值

14.HSCAN key cursor [MATCH pattern] [COUNT count] 
迭代哈希表中的键值对。

## Redis 列表(List)

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）
一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

实例

```
redis 127.0.0.1:6379> LPUSH runoobkey redis
(integer) 1
redis 127.0.0.1:6379> LPUSH runoobkey mongodb
(integer) 2
redis 127.0.0.1:6379> LPUSH runoobkey mysql
(integer) 3
redis 127.0.0.1:6379> LRANGE runoobkey 0 10

1) "mysql"
2) "mongodb"
3) "redis"
```

在以上实例中我们使用了 LPUSH 将三个值插入了名为 runoobkey 的列表当中。

Redis 列表命令

下表列出了列表相关的基本命令：


1.BLPOP key1 [key2 ] timeout 
移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

2.BRPOP key1 [key2 ] timeout 
移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

3.BRPOPLPUSH source destination timeout 
从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

4.LINDEX key index 
通过索引获取列表中的元素

5.LINSERT key BEFORE|AFTER pivot value 
在列表的元素前或者后插入元素

6.LLEN key 
获取列表长度

7.LPOP key 
移出并获取列表的第一个元素

8.LPUSH key value1 [value2] 
将一个或多个值插入到列表头部

9.LPUSHX key value 
将一个值插入到已存在的列表头部

10.LRANGE key start stop 
获取列表指定范围内的元素

11.LREM key count value 
移除列表元素

12.LSET key index value 
通过索引设置列表元素的值

13.LTRIM key start stop 
对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

14.RPOP key 
移除并获取列表最后一个元素

15.RPOPLPUSH source destination 
移除列表的最后一个元素，并将该元素添加到另一个列表并返回

16.RPUSH key value1 [value2] 
在列表中添加一个或多个值

17.RPUSHX key value 
为已存在的列表添加值


## Redis 集合(Set)

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。
Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。
集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)

实例

```
redis 127.0.0.1:6379> SADD runoobkey redis
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mongodb
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 0
redis 127.0.0.1:6379> SMEMBERS runoobkey

1) "mysql"
2) "mongodb"
3) "redis"

```

在以上实例中我们通过 SADD 命令向名为 runoobkey 的集合插入的三个元素。

Redis 集合命令

下表列出了 Redis 集合基本命令：


1.SADD key member1 [member2] 
向集合添加一个或多个成员

2.SCARD key 
获取集合的成员数

3.SDIFF key1 [key2] 
返回给定所有集合的差集

4.SDIFFSTORE destination key1 [key2] 
返回给定所有集合的差集并存储在 destination 中

5.SINTER key1 [key2] 
返回给定所有集合的交集

6.SINTERSTORE destination key1 [key2] 
返回给定所有集合的交集并存储在 destination 中

7.SISMEMBER key member 
判断 member 元素是否是集合 key 的成员

8.SMEMBERS key 
返回集合中的所有成员

9.SMOVE source destination member 
将 member 元素从 source 集合移动到 destination 集合

10.SPOP key 
移除并返回集合中的一个随机元素

11.SRANDMEMBER key [count] 
返回集合中一个或多个随机数

12.SREM key member1 [member2] 
移除集合中一个或多个成员

13.SUNION key1 [key2] 
返回所有给定集合的并集

14.SUNIONSTORE destination key1 [key2] 
所有给定集合的并集存储在 destination 集合中

15.SSCAN key cursor [MATCH pattern] [COUNT count] 
迭代集合中的元素


##Redis 有序集合(sorted set)


Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
有序集合的成员是唯一的,但分数(score)却可以重复。
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。


实例

```
redis 127.0.0.1:6379> ZADD runoobkey 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD runoobkey 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE runoobkey 0 10 WITHSCORES

1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"

```

在以上实例中我们通过命令 ZADD 向 redis 的有序集合中添加了三个值并关联上分数.

Redis 有序集合命令

下表列出了 redis 有序集合的基本命令:


1.ZADD key score1 member1 [score2 member2] 
向有序集合添加一个或多个成员，或者更新已存在成员的分数

2.ZCARD key 
获取有序集合的成员数

3.ZCOUNT key min max 
计算在有序集合中指定区间分数的成员数

4.ZINCRBY key increment member 
有序集合中对指定成员的分数加上增量 increment

5.ZINTERSTORE destination numkeys key [key ...] 
计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中

6.ZLEXCOUNT key min max 
在有序集合中计算指定字典区间内成员数量

7.ZRANGE key start stop [WITHSCORES] 
通过索引区间返回有序集合成指定区间内的成员

8.ZRANGEBYLEX key min max [LIMIT offset count] 
通过字典区间返回有序集合的成员

9.ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 
通过分数返回有序集合指定区间内的成员

10.ZRANK key member 
返回有序集合中指定成员的索引

11.ZREM key member [member ...] 
移除有序集合中的一个或多个成员

12.ZREMRANGEBYLEX key min max 
移除有序集合中给定的字典区间的所有成员

13.ZREMRANGEBYRANK key start stop 
移除有序集合中给定的排名区间的所有成员

14.ZREMRANGEBYSCORE key min max 
移除有序集合中给定的分数区间的所有成员

15.ZREVRANGE key start stop [WITHSCORES] 
返回有序集中指定区间内的成员，通过索引，分数从高到底

16.ZREVRANGEBYSCORE key max min [WITHSCORES] 
返回有序集中指定分数区间内的成员，分数从高到低排序

17.ZREVRANK key member 
返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序

18.ZSCORE key member 
返回有序集中，成员的分数值

19.ZUNIONSTORE destination numkeys key [key ...] 
计算给定的一个或多个有序集的并集，并存储在新的 key 中

20.ZSCAN key cursor [MATCH pattern] [COUNT count] 
迭代有序集合中的元素（包括元素成员和元素分值）

## Redis HyperLogLog

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。
Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。
但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。
什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

实例

以下实例演示了 HyperLogLog 的工作过程：

```
redis 127.0.0.1:6379> PFADD runoobkey "redis"

1) (integer) 1

redis 127.0.0.1:6379> PFADD runoobkey "mongodb"

1) (integer) 1

redis 127.0.0.1:6379> PFADD runoobkey "mysql"

1) (integer) 1

redis 127.0.0.1:6379> PFCOUNT runoobkey

(integer) 3
```


Redis HyperLogLog 命令

下表列出了 redis HyperLogLog 的基本命令：


1.PFADD key element [element ...] 
添加指定元素到 HyperLogLog 中。

2.PFCOUNT key [key ...] 
返回给定 HyperLogLog 的基数估算值。

3.PFMERGE destkey sourcekey [sourcekey ...] 
将多个 HyperLogLog 合并为一个 HyperLogLog


## Redis 发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![]({{site.url}}/assets/bowen49/49-1.png){:height="50%" width="90%"}

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![]({{site.url}}/assets/bowen49/49-1.png){:height="50%" width="90%"}

实例

以下演示了发布订阅是如何工作。在我们实例中我们创建了订阅频道名为redisChat:

```
redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```

现在，我们先重新开启个 redis 客户端，然后在同一个频道 redisChat 发布两次消息，订阅者就能接收到消息。

```
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"

(integer) 1

redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"

(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```

Redis 发布订阅命令

下表列出了 redis 发布订阅常用命令：

1.PSUBSCRIBE pattern [pattern ...] 
订阅一个或多个符合给定模式的频道。

2.PUBSUB subcommand [argument [argument ...]] 
查看订阅与发布系统状态。

3.PUBLISH channel message 
将信息发送到指定的频道。

4.PUNSUBSCRIBE [pattern [pattern ...]] 
退订所有给定模式的频道。

5.SUBSCRIBE channel [channel ...] 
订阅给定的一个或多个频道的信息。

6.UNSUBSCRIBE [channel [channel ...]] 
指退订给定的频道。

详细的参考文献请参考：
<http://www.runoob.com/redis/redis-intro.html>

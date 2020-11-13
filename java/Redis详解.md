# 分布式架构
1. 不同业务功能模块分散部署在不同的服务器上
2. 每个子系统负责一个或者多个不同的业务模块
3. 服务之间可以相互交互与通信
4. 分布式系统设计对用户透明，用户不考虑系统请求到那个服务器或者系统架构如何
5. 可以发展为集群分布式系统架构
## 单体架构
**弊端**：例如造汽车，单体架构将所有零部件安装到车上，每个零部件（模块）之间耦合度比较高，如果某个零件需要升级，那么其他模块都需要跟着一起升级和部署。相当于车子中某个零部件坏了，需要把整个车子的零部件拆掉，再进行更换。
## 分布式架构
优点：
1、 业务解耦
2、系统模块化，可重用
3、提升系统并发量
4、优化运维部署效率，迭代发布会非常轻巧，只需要发布更新过的模块
缺点：
1、架构复杂
2、部署多个子系统复杂
3、系统之间的通信耗时
4、新人融入团队慢
5、调试复杂
设计原则：
1、异步解耦
把耦合的模块进行解耦，模块之间通信尽量使用异步，实在不行的情况使用同步，使用消息队列发送异步请求
2、幂等一致性，用户请求可能经过多个子系统，不管是查询还是增删改不能因为用户提高了两次还是三次，最终结果是一致的，不会因为多次点击产生副作用，尤其是增加或者删除，不会因为用户多次点击产生附加的影响
3、拆分原则，可以用多个拆分原则，以业务功能参考
4、融合分布式中间件
5、容错高可用
## NoSql
非关系型数据库，为了解决大规模诗句集合多重数据种类带来的挑战，尤其是大数据应用难题。
### 传统数据库缺点
1、数据的读写操作频率较高，但由于传统数据库按照行存储，如果只想针对某一列进行运算，关系型数据库仍然会将整行数据从存储设备中读入内存，导致读写I/O占用较高。
2、存储的是行记录，`无法存储数据结构`
3、`表结构schema扩展不方便`，如要需要修改表结构，需要执行执行DDL(data definition language)，语句修改，修改期间会导致锁表，部分服务不可用
4、`全文搜索功能较弱`，关系型数据库下只能够进行子字符串的匹配查询，当表的数据逐渐变大的时候，like查询的匹配会非常慢，即使在有索引的情况下。况且关系型数据库也不应该对文本字段进行索引
5、`存储和处理复杂关系型数据功能较弱`
### NoSQL解决方案
NoSQL，泛指非关系型的数据库，可以理解为SQL的一个有力补充。
1. 键值对数据库 Redis、Memcache
2. 列式数据库  Hbase、Cassandra
3. 文档型数据库 MongoDB、CouchDB
4. 图形数据库  
## 分布式缓存
1. 任意服务器节点可以快速的拿到缓存数据
2. 基于内存式的缓存
# Redis
分布式缓存中间件、key-value形式存储、数据存储在内存中读取更快，提供海量数据存储访问。遵守**BSD协议**（开源协议，可以自由使用，修改源代码，也可以修改后的代码作为开源或再发布）
## 使用Redis的好处
官方测试50个并发执行100000个请求，读的速度`110000次/s` 写的速速`81000次/s`，高并发优秀的解决方案。
1. 缓存数据存在与内存中，类似与HashMap查找和操作的时间复杂度都是O（1）
2. 支持丰富数据类型，支持`string，hash，list，set，zset`
3. 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
4. 丰富的特性，可用于缓存，消息，计数，按key设置过期时间，过期后将会自动删除
5. 单线程，预防多线程的竞态问题

### Redis.conf配置文件
## Redis的内存维护策略
redis时常会在内存中存储大量数据以满足高速读写的要求，即使集群部署来动态扩容，也应该及时整理内存，维护系统的性能。**redis有两种解决方案**
### 1、为数据设置超时时间
没有设置时间，则缓存永远不过期
### 2、采用LRU算法动态将不用的数据删除

> 内存管理的一种页面置换算法，对于在内存中但又不用的数据块叫做LRU，操作系统会根据那些数据属于LRU而将其移出内存而腾出空间来加载另外的数据。
1. 查询所有key中最近最不常使用的数据进行删除，这是应用最广泛的策略
2. 设定超时时间的数据中，删除最不常使用的数据
## Redis应用场景
### 设置过期时间
1. 限时的优惠活动信息
2. 网站数据缓存（对于一些需要定时更新的信息，例如积分排行榜）
3. 手机验证码（一般120s）
4. 限制网站访客频率（一分钟最多访问10次）

## Redis的数据结构
### 1、strings
```java
set mystr "hello world!" //设置字符串类型
get mystr //读取字符串类型
```
还可以对字符串类型进行数值操作

```java
127.0.0.1:6379> set mynum "2"
OK
127.0.0.1:6379> get mynum
"2"
127.0.0.1:6379> incr mynum
(integer) 3
127.0.0.1:6379> get mynum
"3"
```
**在遇到数值操作时，redis会将字符串类型转换成数值。**
**由于INCR等指令本身就具有原子操作的特性**，所以我们完全可以利用redis的`INCR、INCRBY、DECR、DECRBY`等指令来实现原子计数的效果，假如，在某种场景下有3个客户端同时读取了mynum的值（值为2），然后对其同时进行了加1的操作，那么，最后mynum的值一定是5。不少网站都利用redis的这个特性来实现业务上的统计计数需求。
**如果有多客户同时执行setnx，只有一个能设置成功，可做分布式锁**
set命令也支持批量处理，mset/mget批量设置和批量获取。批量操作提高了执行效率，否则一批次的n次查询需要发起n次请求。
### 2、哈希hash
`Hash存的是字符串和字符串值之间的映射`，特别适合`存储对象`，比如一个`用户要存储其全名、姓氏、年龄等等，就很适合使用哈希`（此处如果用String类型就会占用过多的key空间）。

 - `hset key field value` 单个赋值
 - `hmset key field value [filed value]...`批量赋值

```java
//建立哈希，并赋值
127.0.0.1:6379> HMSET user:001 username antirez password P1pp0 age 34 
OK

//列出哈希的内容
127.0.0.1:6379> HGETALL user:001 
1) "username"
2) "antirez"
3) "password"
4) "P1pp0"
5) "age"
6) "34"

//更改哈希中的某一个值
127.0.0.1:6379> HSET user:001 password 12345 
(integer) 0

//再次列出哈希的内容
127.0.0.1:6379> HGETALL user:001 
1) "username"
2) "antirez"
3) "password"
4) "12345"
5) "age"
6) "34"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602154745749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
在一般应用场景中，用户信息缓存化，可以有三种解决方案，1、set存储，2、对象信息序列化存储，3、hset存储，用户信息不经常修改的情况可用2，value值不大的情况可以用3.
### 3、lists集合
首先要明确一点，redis中的lists在底层实现上并不是数组，而是`链表`，因此是`增删改快，定位较慢`。
lists的常用操作包括`LPUSH、RPUSH、LRANGE`等。我们可以用`LPUSH`在lists的左侧插入一个新元素，用`RPUSH`在lists的右侧插入一个新元素，用`LRANGE`命令从lists中指定一个范围来提取元素。

```java
//新建一个list叫做mylist，并在列表头部插入元素"1"
127.0.0.1:6379> lpush mylist "1" 
//返回当前mylist中的元素个数
(integer) 1 
//在mylist右侧插入元素"2"
127.0.0.1:6379> rpush mylist "2" 
(integer) 2
//在mylist左侧插入元素"0"
127.0.0.1:6379> lpush mylist "0" 
(integer) 3
//列出mylist中从编号0到编号1的元素
127.0.0.1:6379> lrange mylist 0 1 
1) "0"
2) "1"
//列出mylist中从编号0到倒数第一个元素
127.0.0.1:6379> lrange mylist 0 -1 
1) "0"
2) "1"
3) "2"
```
### 4、set无序集合
Redis的集合，是一种`无序的集合`，集合中的元素没有先后顺序。集合相关的操作也很丰富，如添加新元素、删除已有元素、取交集、取并集、取差集等。

```java
//向集合myset中加入一个新元素"one"
127.0.0.1:6379> sadd myset "one" 
(integer) 1
127.0.0.1:6379> sadd myset "two"
(integer) 1
//列出集合myset中的所有元素
127.0.0.1:6379> smembers myset 
1) "one"
2) "two"
//判断元素1是否在集合myset中，返回1表示存在
127.0.0.1:6379> sismember myset "one" 
(integer) 1
//判断元素3是否在集合myset中，返回0表示不存在
127.0.0.1:6379> sismember myset "three" 
(integer) 0
//新建一个新的集合yourset
127.0.0.1:6379> sadd yourset "1" 
(integer) 1
127.0.0.1:6379> sadd yourset "2"
(integer) 1
127.0.0.1:6379> smembers yourset
1) "1"
2) "2"
//对两个集合求并集
127.0.0.1:6379> sunion myset yourset 
1) "1"
2) "one"
3) "2"
4) "two"
```
使用场景：标签，社交，查询有共同兴趣爱好的人，智能推荐
### 5、zset有序集合
同无序集合一样，存储string类型且不能重复。`每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。`

```java
redis 127.0.0.1:6379> ZADD key 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD key 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD key 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD key 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD key 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE key 0 10 WITHSCORES
1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```
常用于排行榜，如视频网站需要对用户上传视频做排行榜，或点赞数
### redis的全局命令
```java
查看所有键：keys *，也可以用*进行模糊匹配。
键总数 dbsize 如果存在大量键，线上禁止使用此指令
检查键是否存在：exists keyName 存在返回1，不存在返回0
删除键：del keyName 返回删除键个数，删除不存在键返回0
键过期：expire keyName seconds
键的数据结构类型：type key 返回string,键不存在返回nil
```
## Redis持久化机制
Redis是一个`支持持久化`的内存数据库，为了`防止数据丢失Redis需要经常将内存中的数据同步到磁盘来保证持久化`，它提供了两种持久化的方式，分别是`RDB（Redis DataBase）`和`AOF（Append Only File）`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201002151641900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)

### 1.RDB（Redis DataBase）
简而言之，就是在不同的时间点，`将redis存储的数据生成快照并存储到磁盘等介质上；`是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照（数据库中所有键值对数据）。恢复时是将快照文件直接读到内存里。
Redis默认为该方式，`内存中数据以快照的方式写入到二进制文件中，默认的文件名为dump.rdb`（如果需要恢复数据，只需将备份文件dump.rdb 移动到 redis 安装目录并启动服务即可）。可以通过配置设置自动做快照持久化的方式。配置redis在n秒内如果超过m个key被修改就自动做快照，下面是默认的快照保存配置。
**配置自动触发**
```java
save 900 1         #900秒内如果超过1个key被修改，则发起快照保存
save 300 10        #300秒内容如超过10个key被修改，则发起快照保存
save 60 10000      #60秒内容如超过10000个key被修改，则发起快照保存
```
**手动触发**
- save
- bgsave
client 也可以使用save或者bgsave命令手动通知redis做一次快照持久化。save操作是在主线程中保存快照的，由于redis是用一个主线程来处理所有 client的请求，这种方式会阻塞所有client请求，不推荐使用。

- 优点1	压缩后的二进制文，适用于备份、全量复制，用于灾难恢复
- 优点2	大数据量时，加载RDB恢复数据远快于AOF方式
- 缺点1	无法做到实时持久化，每次都要创建子进程，频繁操作成本过高
- 缺点2	保存后的二进制文件，存在老版本不兼容新版本rdb文件的问题
### 2.AOF（Append Only File）
AOF则是换了一个角度来实现持久化，那就是将`redis执行过的所有写指令记录下来`，在下次redis重新启动时，只要把这些`写指令从前到后再重复执行一遍`，就可以实现数据恢复了，`相关配置如下`：

```java
appendonly  no         # redis默认关闭AOF机制，可以将no改成yes实现AOF持久化
appendfsync always     # 每次有数据修改发生时都会写入AOF文件。
appendfsync everysec   # 每秒钟同步一次，该策略为AOF的缺省策略。
appendfsync no         # 从不同步。高效但是数据不会被持久化。
```
**如何恢复数据**
具体操作方式：如何从AOF恢复数据？
1. 设置appendonly yes；
2. 将appendonly.aof放到dir参数指定的目录；
3. 启动Redis，Redis会自动加载appendonly.aof文件。

其实RDB和AOF两种方式也可以同时使用，在这种情况下，如果redis重启的话，则会`优先采用AOF方式来进行数据恢复`，这是因为AOF方式的数据恢复完整度更高。如果你没有数据持久化的需求，也完全可以关闭RDB和AOF方式，这样的话，redis将变成一个纯内存数据库，就像memcache一样。
**重启时恢复加载AOF与RDB顺序及流程：**

- 当AOF和RDB文件同时存在时，优先加载AOF
- 若关闭了AOF，加载RDB文件
- 加载AOF/RDB成功，redis重启成功
- AOF/RDB存在错误，redis启动失败并打印错误信息
## Springboot使用Redis
### 首页轮播图
```
轮播图查询
因为轮播图改的机会比较少，为了减轻服务器请求的压力，可以把轮播图数据放到redis中
流程如下：
缓存轮播图结构<String,String>
1. 每次请求轮播图列表前，先判断redis中是否存在轮播图
		如果存在，在取出轮播图列表的JSON字符串，然后使用JSON工具类将字符串转换成list集合
		如果不存在，则在数据库中查询
		将查询出来的集合转换成JSON字符串后存储到redis缓存中
2. 最后返回list集合给前端（redis中取出/数据库中查出）

轮播图修改
1、 一旦广告轮播图发生更改，后台业务就立刻执行删除缓存，然后重新查询数据库轮播图数据重新缓存
2、 定时重置，比如每天凌晨三点重置
3、 每个轮播图都有可能是一个广告，每个广告都会有一个过期时间，直到时间过期了我们再进行重置
```
### 首页分类数据
首页分类数据的修改可能会比轮播图更加低，所以把分类数据也缓存起来减轻对数据库的压力

### 购物车数据
```
前端用户在登录的情况下，添加商品到购物车，会同时在后端同步购物车到redis缓存中
需要判断当前购物车中是否存在相同的商品，如果存在，则数量累加购买数量

添加商品

这里添加购物车数据还要加上用户id加以区别
		添加的格式 set("subCat:"+userId) 这里加上冒号，redis中会用文件夹把这购物车数据呈现出来
缓存中是否存在购物车的数据
	如果说不为空，
	1. 取出JSON字符串，转换成对象
	2. 并且判断待添加商品是否在购物车中存在
			采用遍历集合并且判断集合中商品id是否与待添加商品相同
			如果都没有相同则添加商品到购物车中，否则就累加商品的购买数量
如果缓存中没有
	则直接new一个商品集合，把添加的商品添加到集合中
	同时把集合数据放到redis缓存中
```
## Redis主从复制
我们了解了Redis两种不同的持久化方式，Redis服务器通过持久化，把Redis内存中持久化到硬盘当中，当Redis宕机时，我们重启Redis服务器时，可以由RDB文件或AOF文件恢复内存中的数据。
`不过持久化后的数据仍然只在一台机器上`，**因此当硬件发生故障时，比如主板或CPU坏了，这时候无法重启服务器，有什么办法可以保证服务器发生故障时数据的安全性**？或者可以快速恢复数据呢？想做到这一点，**我们需要再了解Redis另外一种机制：主从复制。**
### 什么是主从复制
Redis的主从复制机制是指可以让`从服务器(slave)`能精确复制`主服务器(master)`的数据，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200611143043271.png)
上面的图表示的是一台master服务器与slave服务器的情况，其实一台`master服务器`也可以对应`多台slave服务器`，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200611143137586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
另外，`slave服务器也可以有自己的slave服务器`，`这样的服务器称为sub-slave`,而这些sub-slave通过主从复制最终数据也能与master保持一致，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061114321941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### 主从复制的方式和工作原理
Redis的`主从复制是异步复制`，异步分为两个方面，
1、master服务器在将数据同步到slave时是异步的，因此master服务器在这里仍然可以接收其他请求，
2、slave在接收同步数据也是异步的。
> 同步，就是调用某个东西是，调用方得等待这个调用返回结果才能继续往后执行。
> 异步，调用方不会立即得到结果，而是在调用发出后调用者可用继续执行后续操作，被调用者通过状态来通知调用者，或者通过回调函数来处理这个调用

#### Redis主从复制方式
1、当master服务器与slave服务器正常连接时，master服务器会发送数据命令流给slave服务器,将自身数据的改变复制到slave服务器即增量复制。
2、当因为各种原因master服务器与slave服务器断开后，slave服务器在重新连上master服务器时会尝试重新获取断开后未同步的数据即部分同步，或者称为部分复制。
3、如果无法部分同步(比如初次同步)，则会请求进行全量同步，这时master服务器会将自己的rdb文件发送给slave服务器进行数据同步，并记录同步期间的其他写入，再发送给slave服务器，以达到完全同步的目的，这种方式称为全量复制。

```
即第一次复制为全量赋值，后面的复制为增量复制
当服务器断开后重连，会重新获取断开后未同步部分数据
```
#### Redis主从复制工作原理
master服务器会记录一个`replicationId`的伪随机字符串，`用于标识当前的数据集版本`，还会`记录一个当数据集的偏移量offset`，**不管master是否有配置slave服务器，replication Id和offset会一直记录并成对存在**，我们可以通过以下命令查看replication Id和offset：

```
info 查看replication部分
```
通过redis-cli在master或slave服务器执行该命令会打印类似以下信息(不同服务器数据不同，打印信息不同)：

```
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=9472,lag=1
master_replid:2cbd65f847c0acd608c69f93010dcaa6dd551cee
master_repl_offset:9472
```
当master与slave正常连接时`，slave使用PSYNC命令向master发送自己记录的旧master的replication id和offset`，而`master会计算与slave之间的数据偏移量，并将缓冲区中的偏移数量同步到slave`，此时master和slave的数据一致。
而**如果slave引用的replication太旧了，master与slave之间的数据差异太大，则master与slave之间会使用全量复制的进行数据同步。**

### Redis配置主从复制
Redis的主从配置非常简单，我们可以使用`两种方式`来配置主从服务器，在这时我们先假设`Redis的master服务器地址为192.168.0.101`。
**客户端发送同步命令**

```
# 向客户端
saveof 192.168.1.101 6379
```
**slave服务器配置主服务器**
在这里slave服务器的`redis.conf`通过saveof选项，可以指定master服务器，如下：
```
slaveof 192.168.1.101 6379
```
**master要求验证即master服务器由密码**
上面配置的是master服务器没有设置密码的情况，如果master设置了密码，则可以在连接到slave服务器的redis-cli执行下面的命令：

```
# <password>指代实际的密码
config set masterauth <password>
```
或者在slave服务器的redis.conf中配置下面的选项：
```
masterauth 123456
```
**slave默认为只读的**
在Redis2.6以后，slave只读模式是默认开启的，我们可以通过配置文件中的slave-read-only选项配置是否开启只读模式：

```
# 默认是yes
slave-read-only yes/no 
```
或者在客户端中通过config set命令设置是否开启只读模式：

```
config set slave-read-only no
```
### 避免slave被清空
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200611154153664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
lave会被清空？slave不用同步了master的数据吗？备份的数据怎么会清空了呢？
当master服务器**关闭了持久化时**，如果发生故障后自动重启时，由本地没有保存持久化的数据，重启的Redis内存数据为空，而**slave会自动同步master的数据，这时候，slave服务器的数据也会被清空。**
`如何避免slave被清空呢`？
如果条件允许(一般都可以的)，master服务器还是要开启持久化，这样master故障重启时，可以快速恢复数据，而同步这台master的slave数据也不会被清空。
**即master主机无法持久化，或者无法重启的时候就将从服务器升级成master**
如果`master不能开启持久化，则不应该设置让master发生故障后重启`(有些机器会配置自动重启)，而是`将某个slave服务器升级为master服务器，对外继续提供服务。`
### 主从复制中的key过期问题
我们都知道Redis可以通过设置key的过期时间来限制key的生存时间，`Redis处理key过期有惰性删除和定期删除两种机制`，而在`配置主从复制后，slave服务器就没有权限处理过期的key`，这样的话，对于在master上过期的key，在slave服务器就可能被读取，所以**master会累积过期的key，积累一定的量之后，发送del命令到slave，删除slave上的key。**
如果**slave服务器升级为master服务器 ，则它将开始独立地计算key过期时间，而不需要通过master服务器的帮助。**
### Redis主从复制的作用
#### 保存Redis数据副本
当我们只是通过RDB或AOF把Redis的内存数据持久化毕竟只是在本地，并不能保证绝对的安全，而通过将数据同步slave服务器上，可以保留多一个数据备份，更好地保证数据的安全。
### 读写分离
在配置了主从复制之后，如果master服务器的读写压力太大，可以进行读写分离，客户端向master服务器写入数据，在读数据时，则访问slave服务器，从而减轻master服务器的访问压力。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200611154726837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### 高可用性与故障转移
服务器的**高可用性是指服务器能提供7*24小时不间断的服务**，Redis可以通过Sentinel系统管理多个Redis服务器，当master服务器发生故障时，Sentineal系统会根据一定的规则将某台slave服务器升级为master服务器,继续提供服务，实现故障转移，保证Redis服务不间断。

## Redis缓存过期机制
1. 定期删除（主动）占用内存较高
2. 惰性删除（被动）

## Redis主从复制存在的问题
Redis 主从复制 可将 主节点 数据同步给 从节点，从节点此时有两个作用：
1、一旦 主节点宕机，从节点 作为 主节点 的 备份 可以随时顶上来。
2、扩展 主节点 的 读能力，分担主节点读压力。
`主从复制` 同时存在以下几个问题：
1、、一旦主节点宕机，`从节点晋升成主节点`，同时`需要修改应用方的主节点地址`，还需`命令所有从节点去复制新的主节点`，整个过程`需要人工干预`。
2、主节点 的 写能力 受到 单机的限制。
3、主节点 的 存储能力 受到 单机的限制
4、原生复制的弊端在早期的版本中也会比较突出，比如：Redis 复制中断 后，从节点会发起`psync`。此时`如果同步不成功`，则会进行`全量同步`，`主库`执行`全量备份`的同时，可能会造成`毫秒或秒级的卡顿`。
## Redis哨兵模式
### Redis哨兵模式解决的问题
**主机宕机**
 - 谁来确定主机宕机
 - 怎么将宕机的master下线
 - 如何找一个slave作为新的master
 - 如何通知其他slave连接新的master
 - 启动新的master和slave
 - 选择新的master后数据同步（全量复制和部分复制）
 - 修改配置后，原始的master恢复了怎么办？
### 哨兵简介
哨兵（sentinel）是一个分布式系统，用于对主从结构中的每台服务器进行`监控` ，当出现故障的时候通过`投票机制选择`新的master，并将所有的slave连接到新的master。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612094605493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### Redis Sentinel的主要功能
**监控**
`Sentinel` 会不断的检查 `主服务器` 和 `从服务器` 是否正常运行
同步各个节点的状态信息


**通知**
当被监控的某个 Redis 服务器出现问题，Sentinel 通过 `API 脚本` 向其他哨兵或者客户端发送通知。

**自动故障转移**
当 `主节点 不能正常工作`时，Sentinel 会开始一次 `自动的 故障转移操作`，它会断开master和slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址。

**配置提供者**
在 Redis Sentinel 模式下，客户端应用 在`初始化时连接`的是 `Sentinel 节点集合`，从中`获取主节点的信息`
**注意**
哨兵sentinel也是一台redis服务器，只是不提供数据服务器，通常哨兵的配置数量为单数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612101142345.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)

### 哨兵工作原理
#### 主从切换
哨兵在进行主从切换过程中经历三个阶段
 - 监控
 - 通知
 - 故障转移

**阶段一：监控阶段**
 每个 Sentinel 以 每秒钟 一次的频率，向它所知的 主服务器、从服务器 以及其他 Sentinel 实例 发送一个 PING 命令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612110452156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_7
用于同步各个节点的状态信息：

 - 获取各个sentinel的状态是否在线
 - 获取master的状态
runid
role：master
各个slave的详细信息
 - 根据master中的slave信息去获取所有slave的状态
runid
role：slave
master_host、master_port
offset：与主节点的数据偏移量
... 

 
**阶段二：通知阶段**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061211060364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
Sentinel 向其监控的 Redis 节点 **__sentinel__:hello** 这个 channel 发布自己的信息及主节点相关的配置。sentinel将收到的回复发给其他的sentinel节点保持信息的同步。

**阶段三：故障转移**
如果一个 实例（instance）距离 最后一次 有效回复 PING 命令的时间超过 **down-after-milliseconds** 所指定的值，那么这个实例会被 Sentinel 标记为 主观下线。这时sentinel节点会向其他sentinel节点同步信息，那么正在监视这个主服务器的所有 Sentinel 节点，要以 `每秒一次` 的频率确认 主服务器 的确进入了 `主观下线` 状态。
1. sentinel1判断到master为`主观下线状态`
2. sentinel1向其他sentinel发送消息：master下线了
3. 其他监控master的sentinel服务器 `每秒一次` 的频率确认master主服务器的确进入了 `下线` 状态。
其中只要`过半数的sentinel`确认master下线则master就转为客观下线状态
4. 将master服务器转为`客观下线`状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612112553150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
master客观下线后就要进行主从切换，从slave中选出一个新的master，sentinel单数的原因是sentinel服务器中通过投票机制选出一个`leader`去进行主从切换
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061211592727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
选出leader后由leader从现在的slave中`优选`出一个master
选master条件，它会将与 失效主节点 是 主从关系 的其中一个 从节点 升级为新的 主节点，并且将其他的 从节点 指向 新的主节点。

### Sentinel的通信命令
Sentinel 节点连接一个 Redis 实例的时候，会创建 cmd 和 pub/sub 两个 连接。Sentinel 通过 `cmd 连接`给 `Redis 发送命令`，通过 `pub/sub` 连接到 Redis 实例上的`其他 Sentinel 实例`。
`Sentinel 与 Redis` 主节点 和 从节点 `交互的命令`，主要包括：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612121414253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
`Sentinel 与 Sentinel 交互的命令`，主要包括：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200612121435665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### Redis sentinel哨兵模式部署

[redis哨兵模式部署详解文章](https://juejin.im/post/5b7d226a6fb9a01a1e01ff64#heading-8)

## 缓存穿透
我们在项目中使用缓存通常都是先检查缓存中是否存在，如果存在直接返回缓存内容，如果不存在就直接查询数据库然后再缓存查询结果返回。这个时候如果我们查询大量数据在缓存中一直不存在，就会造成每一次请求都查询DB，这样缓存就失去了意义，在流量大时，可能数据库就宕机了这就是缓存穿透现象。
要是有人利用不存在的key频繁攻击我们的应用，这就是缓存穿透，给数据库带来了很大的并发压力。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613100407620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### 一、缓存空对象

> 缓存中没有数据库中也没有查到的key，对该key值缓存一个空对象，防止该key频繁访问数据库。

简单粗暴的方法，如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。

### 二、布隆过滤器

> 它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

#### 布隆过滤器原理
布隆过滤器的原理是，当一个元素被加入集合时，通过`K个散列函数将这个元素映射成一个位数组中的K个点`，把它们`置为1`。检索时，我们只要看看这些点是不是都是1就（大约）知道集合中有没有它了：`如果这些点有任何一个0，则被检元素一定不在`；如果都是1，则被检元素`很可能在`（有一定的误判率）。这就是布隆过滤器的基本思想。
Bloom Filter跟单哈希函数Bit-Map不同之处在于：Bloom Filter使用了k个哈希函数，每个字符串跟k个bit对应。从而降低了冲突的概率。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613094745921.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
#### 使用方法

> 布隆过滤器能确定一定不存在的key，但是判断可能存在的key有误差，这样大部分不在缓存中的key就能被布隆过滤器过滤掉

1、我们先把我们数据库的数据全都加载到我们的过滤器中，比如数据库的id现在有：1、2、3
2、那就用id：1 为例子他在上图中经过三次hash之后，把三次原本值0的地方改为1
3、下次数据进来查询的时候如果id的值是1，那么我就把1拿去三次hash 发现三次hash的值，跟上面的三个位置完全一样，那就能证明过滤器中有1的
4、反之如果不一样就说明不存在了

#### 布隆过滤器缺点

> 布隆过滤器只能添加key和判断key是否存在。

bloom filter之所以能做到在时间和空间上的效率比较高，是因为牺牲了判断的准确率、删除的便利性。
- `存在误判`，可能要查到的元素并没有在容器中，但是hash之后得到的k个位置上值都是1。如果bloom filter中存储的是黑名单，那么可以通过建立一个白名单来存储可能会误判的元素。
- `删除困难`。一个放入容器的元素映射到bit数组的k个位置上是1，删除的时候不能简单的直接置为0，可能会影响其他元素的判断。可以采用Counting Bloom Filter。如果数据库中进行删除了，布隆过滤器没法做到删除缓存中的key值，只能重新初始化布隆过滤器。

## 缓存击穿
在平常高并发的系统中，`大量的请求同时查询一个 key 时，此时这个key正好失效了`，`就会导致大量的请求都打到数据库上面去。导致数据库压力剧增，这种现象我们称为缓存击穿`。
### 解决方案
1. 设置热点数据用不过期（最简单直接的办法）
2. 后台刷新（定时任务在过期时间内进行刷新缓存）
后台定义一个job(`定时任务)专门用来主动更新缓存数据.`比如,一个缓存中的数据过期时间是30分钟,那么job每隔29分钟定时刷新数据(将从数据库中查到的数据更新到缓存中).
3. 检查更新

> 将缓存key的过期时间(绝对时间)一起保存到缓存中(可以拼接,可以添加新字段,可以采用单独的key保存..不管用什么方式,只要两者建立好关联关系就行).在每次执行get操作后,都将get出来的缓存过期时间与当前系统时间做一个对比,如果缓存过期时间-当前系统时间<=1分钟(自定义的一个值),则主动更新缓存.这样就能保证缓存中的数据始终是最新的(和方案一一样,让数据不过期.)

这种方案在特殊情况下也会有问题。假设缓存过期时间是12:00，而 11:59 
到 12:00这 1 分钟时间里恰好没有 get 请求过来，又恰好请求都在 11:30 分的时 
候高并发过来，那就悲剧了。这种情况比较极端，但并不是没有可能。因为“高 
并发”也可能是阶段性在某个时间点爆发。

4.  加互斥锁
互斥锁过程，让第一个进程拿锁，查缓存没有、进入查询数据库，将数据放入缓存中，释放锁，一小部分获取锁的进程都可以查询缓存，大部分直接判断缓存是否有。
  1）缓存中有数据，直接走上述代码13行后就返回结果了
  2）缓存中没有数据`，第1个进入的线程，获取锁并从数据库去取数据，没释放锁之前，其他并行进入的线程会等待100ms，再重新去缓存取数据`。这样就防止都去数据库重复取数据，重复往缓存中更新数据情况出现。
  3）当然这是简化处理，`理论上如果能根据key值加锁就更好了，就是线程A从数据库取key1的数据并不妨碍线程B取key2的数据`，上面`代码明显做不到这点。`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200613100838635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 缓存雪崩
缓存雪崩是指我们设置缓存key采用了相同的过期时间，导致缓存在某一时刻同时失效，例如有一万个数据同时过期，这瞬间就要去数据库查询一万个数据，导致数据库DB压力过大雪崩。
### 解决
在原有失效时间基础上增加一个随机值，比如1-5分钟内随机，这样每一个缓存的过期时间的重复率就会降低，就很难造成集体缓存失效的事件，而导致数据库压力过大。




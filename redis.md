## redis

 Redis 数据库采用 I/O 多路复用技术实现文件事件处理器
 I/O 多路复用：linux有五类io模型 1.阻塞 2.非阻塞 3.io多路复用 4.事件驱动 5.异步
    阻塞：一次网络io时，C端发出请求，S端收到。当C端发出一个请求，进行io时，就不能进行其他操作了，需要同步的等待结果的返回。
    io多路复用：有多个C端同时发送请求，这些IO操作会被selector(epoll，kqueue)给暂时挂起，入内存队列。此时S端可以自己选择什么时候读取、处理这些io，也就是说S端可以同时hold住多个io。

#### redis 为什么那么快？

  1. 纯内存操作
  2. 单线程操作、避免了频繁的上下文切换(多线程需要占用更多的CPU资源)
  3. 采用了非阻塞I/O多路复用机制

#### redis 后台运行？

redis.conf // 将 daemonize no 改为 yes

#### redis 发布订阅

![1629986839260](redis.assets/1629986839260.png)

第一个：消息发送者，2：消息发送者；3 消息订阅者

```bash
subscribe 主题名 # 订阅主题
publish 主题 message # 往主题中发送信息
```

#### redis 主从复制

概念：将一台服务器内容，复制到其他redis服务器

![1629988887301](./redis.assets/1629988887301.png)

##### 主从复制的作用

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

 	2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复
 	3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，从节点提供读服务，分担服务器负载，尤其是在写少读多的情况下，通过多个从节点分担读负载，可以大大提高redis服务器并发量
 	4. 高可用基石：主从复制还是哨兵和集群能够实施的基础，因此说主从复制是redis高可用的基础。

##### 主从配置

```bash
127.0.0.1:6379>slaveof 127.0.0.1 6379 # 暂时配置
```

redis config 中配置 可永久生效

##### 复制原理

slave 启动成功连接到master后会发送一个sync同步命令

master接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，==master将传送整个文件到salve,并完成一次同步。==

全量复制：salve服务在接收到数据库文件数据后，将其存盘并加载到内存中。

增量复制：master继续将所有收集到的修改命令依次传给salve，完成同步

但是只要是重新连接master，一次完全同步将被自动执行。

### 哨兵模式

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行，其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例**。

![1630071513083](redis.assets/1630071513083.png)

哨兵有两个作用

- 通过发送命令，让redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换到master，然后通过发布订阅模式，通知其他的从服务器，修改配置文件，让它们切换主机

然而一个哨兵进程对redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控，各个哨兵之间还会进行监控，这就形成了多哨兵模式

![1630071868177](redis.assets/1630071868177.png)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象称为**主观下线**，当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover[故障转移]操作，切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。

#### 测试

##### 1. 配置哨兵配置文件 sentinel.conf

```bash
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

这里面的数字 1 表示，当主机挂了，slave票数最多的成为主机

如果此时主机恢复，会变为从机

优点：

1. 哨兵集群，基于主从复制模式，所有的主从配置优点，它会有
2. 主从可以切换，故障可以转移，系统的可用性会更好
3. 哨兵模式就是主从模式的升级，手动到自动，系统更加健壮

缺点：

1. redis不好在线扩容，集群容量一但到达上限，在线扩容就十分麻烦
2. 实现哨兵模式的配置很麻烦，里面有很多选择



### redis数据类型

#### 1. string

```bash
# 查看所有key
keys * 

# 设置key
set key value 

# 获取key
get key 

# 移除当前key
move key 

# 清空所有key
flushall

# 判断key是否存在
exists name

#  给key设置过期时间
expire key val

# 查询当前key的剩余时间
ttl key

# 查看当前key的类型
type name

# 往字符串后面追加字符，如果key不存在，就相当于set key
append key value 

#  获取key的值长度
strlen key

# 字符串累计、累减
incr key # key值加1
decr key # key值减1
incrby key value # key值加value， value： 步长
decrby key value # key值加value， value： 步长

# 字符串范围
getrange key 0 3 # 截取字符串[0,3]
setrange key 1 xx # 替换指定位置开始的字符串

setex (set with expire) # 设置过期时间
setex key 30 "val" # 设置key 30s 过期， 值为val

# 不存在设置（分布式锁中会常常使用），存在设置失败
setnx (set if not exist) 
setnx key val # 语法

mset k1 v1 k2 v2 # 同时设置多个键值
msetnx k1 v1 k2 v2 # 是一个原子性的操作，要么一起成功，要么一起失败
mget k1 k2 k3 # 同时获取多个对象

# 对象
set user:1{name:zhangsan,age:3} # 设置一个user:1 对象，值为接送字符串来保存一个对象

set user:{id}:{field} val # 这里的key是一个巧妙的设计

getset key val # 如果不存在值，设置值，并返回nil，如果存在值，返回值，并设置新值


# string 类型的使用场景:value 除了是我们的数字还可以是字符串
```

#### 2. list

```bash
# 将值插入列表头部，从左边放入队列
lpush key value 

# 取出队列的一部分数据
lrange key 0 -1 

# 将值插入到列表尾部，从右边放入队列
rpush key value  

# 从右边取出值
rpop key  

# 从左边取出值
lpop key  

# 按下标(index)获取队列的某一个值
lindex key index 

# 获取队列的长度
llen key  

# 移除指定的值,移除key中count个val
lrem key count val   
# 例如： lrem list 2 abc # 移除list中的2个abc

# 截取队列，从start开始，end结束
ltrim key start end  
# 例如： ltrim list 2 5 # 截取list，从下标2开始，5结束

# 从第一个队列的右边取出值，放入第二个队列中
rpoplpush list1 list2 

# 判断list是否存在
exists list  

# 更新操作，将list中下标为index的值改为newValue,修改的值不存在，就报错
lset list index newValue  

# 更新值，将0个元素更新为item
lset list 0 item 

# 在list中在值value前面插入新值newValue
linsert list before value newValue

# 在list中在值value后面插入新值newValue
linsert list after value newValue  

# 小结
# 如果key不存在，创建新的链表
# 如果key存在，新增内容
# 如果移除了所有值，空链表，也代表不存在
# 在两边插入或改动值，效率最高
```

#### 3. set

```bash
 # 往集合中插入元素
 sadd set value 
 
 # 查看集合中的所有元素
 smembers set  
 
 # 判断某一个值是不是在集合中
 sismember set value  
 
 # 获取set集合中的元素个数
 scard set  
 
 # 移除set集合中的某一个元素
 srem set value 
 
 # 随机抽出一个元素
 srandmember set 
 
 # 随机抽出count个元素
 srandmember set count  
 
 # 随机删除集合中的元素
 spop set 
 
 # 两个集合的差集
 sdiff set1 set2
 
 # 两个集合的交集
 sinter set1 set2 
 
 # 并集
 sunion set1 set2  
```

#### 4. hash

```bash
# 设置值
hset hash field value 

# 获取值
hget hash field 

# 设置多个值
hmset hash f1 v1 f2 v2 

# 获取多个值
hmget hash f1 f2 

# 删除指定的值
hdel hash field  
```

#### 5. zset

```bash
# 增加数据和权重
zadd set source value

# 显示全部的用户，升序排序 inf:无穷
zrangebyscore set -inf +inf [withscores] 

# 获取集合中所有的值
zset set 0 -1 

# 移除zset中的元素
zrem set value 

# 查看集合中元素个数
zcard set 

# 从大到小排序
zrevrange set 0 -1

# 获取指定区间的元素
zcount set start end  
```

### redis 配置文件

```bash
#  绑定IP
bind 127.0.0.1

# 守护进程运行
daemonize yes 

# 如果以后台方式运行，指定pid 文件
pidfile /var/run/redis_6379.pid 

# 数据库数量
databases 16 

# 日志文件的位置
logfile '' 

# 日志级别
loglevel notice  

# 快照，在规定的时间内，执行了多少次操作，则会持久化文件
save 900 1 # 如果900s内有一个修改，就持久化
save 300 10 # 300s内至少修改了10
save 60 10000 # 60s内修改了10000次

stop-writes-on-bgsave-error yes # 持久化出错，是否还需要工作
rdbcompression yes # 是否压缩rdb文件，需要消耗一些CPU资源
rdbchecksum yes # 保持rdb文件时，进行错误检查
dir ./ # rdb文件保存目录

# 如果有密码时，登录
auto pwd # 用密码登录

appendonly no # 默认是不开启aof模式的
appendfilename 'appendoonly.aof' # 持久化的文件名
appendsync everysec # 每秒执行一次，sync,可能会丢失这1s的数据
always # 每次修改都会 sync
no # 不同步
```


## m0ongoDB

#### 1.  什么是mongoDB ?

基于分布式文件存储的开源数据库系统

在高负载的情况下，添加更多的节点，可以保证服务器性能

#### 2.  mongoDB特点？

1. MongoDB 是一个面向文档存储的数据库，操作起来比较简单和容易。
2. 你可以在MongoDB记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序。
3. 你可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
4. 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。
5. Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。
6. MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。
7. Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。
8. Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。
9. Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。
10. GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
11. MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。

#### 3. mongoDB与SQL术语说明

![image-20210826181906899](./mongoDB.assets/image-20210826181906899.png)

#### 4. mongoDB连接？

```bash
mongodb: # [username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
 # mongodb:// 这是固定的格式，必须要指定
 # username:password@ 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登录这个数据库
 # host1 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制  		集，请指定多个主机地址。
 # portX 可选的指定端口，如果不填，默认为27017
 # /database 如果指定username:password@，连接并验证登录指定数据库。若不指定，默认打开 test 数据库。
 # ?options 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开
 
 mongodb://localhost # 使用默认端口连接
 
 $ ./mongo  # 通过shell连接
MongoDB shell version: 4.0.9
connecting to: test
... 

# 使用用户 admin 使用密码 123456 连接到本地的 MongoDB 服务上
> mongodb://admin:123456@localhost/ 
... 

# mongodb://admin:123456@localhost/test
mongodb://admin:123456@localhost/test 

# 连接 replica set 三台服务器 (端口 27017, 27018, 和27019)
mongodb://localhost,localhost:27018,localhost:27019 

# 连接 replica set 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器。
mongodb://host1,host2,host3/?slaveOk=true

# 直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器。
mongodb://host1,host2,host3/?connect=direct;slaveOk=true
```

#### 5. 创建数据库

```bash
> use runoob # 创建数据库
switched to db runoob
> db
runoob
> 
> show dbs # 查看所有数据库
admin   0.000GB
config  0.000GB
local   0.000GB
> 
# 删除数据库
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
runoob  0.000GB
> db.dropDatabase()
{ "dropped" : "runoob", "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```



#### 6. mongoDB 语法

```bash
# 创建集合语法
db.createCollection(name, options)
	# name : 要创建的集合名称
	# options : 可选参数，指定有关内存大小及索引的选项。options参数如下图：
```

![image-20210827143002565](mongoDB.assets/image-20210827143002565.png)

```sql
# 查看已有集合
> show collections
runoob
system.indexes
# 创建固定集合 mycol，整个集合空间大小 6142800 B, 文档最大个数为 10000 个。
> db.createCollection("mycol", { capped : true, autoIndexId : true, size : 
   6142800, max : 10000 } )
{ "ok" : 1 }
>

# 在 MongoDB 中，你不需要创建集合。当你插入一些文档时，MongoDB 会自动创建集合。
> db.mycol2.insert({"name" : "菜鸟教程"})
> show collections
mycol2

# 删除集合
db.collection.drop()
# 删除集合 mycol2 :
>db.mycol2.drop()
true

	
```

##### 1. 插入

````sql
# 插入文档
db.collection_name.insert(document)
或
db.collection_name.save(document)
 # save()：如果 _id 主键存在则更新数据，如果不存在就插入数据。该方法新版本中已废弃，可以使用 db.collection.insertOne() 或 db.collection.replaceOne() 来代替。
 # insert(): 若插入的数据主键已经存在，则会抛 org.springframework.dao.DuplicateKeyException 异常，提示主键重复，不保存当前数据。

# db.collection.insertOne() 用于向集合插入一个新文档，语法格式如下：
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)

# db.collection.insertMany() 用于向集合插入一个多个文档，语法格式如下：
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
# 参数说明：
	# document：要写入的文档。
	# writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
	# ordered：指定是否按顺序写入，默认 true，按顺序写入。
````



##### 2. 更新

```sql
	# 更新操作
	 # update() 方法用于更新已存在的文档,默认会使用新对象替换旧对象，updateOne(), updateMany(),updateReplace()
	 db.collection.update(
     <query>,
     <update>,
     {
       upsert: <boolean>,
       multi: <boolean>,
       writeConcern: <document>
     }
	 )
	 # 参数说明：
	 	# query : update的查询条件，类似sql update查询内where后面的。
	 	# update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
	 	# upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是alse，不插入。
	 	# multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
	 	# writeConcern :可选，抛出异常的级别。
	 	
	# 如果你要修改多条相同的文档，则需要设置 multi 参数为 true。
	>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
	
	# 只更新第一条记录：
	db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );
	
	# 全部更新：
	db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );
	
	# 只添加第一条：
	db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
	
	# 全部添加进去:
	db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );
	 
	# 全部更新：
	db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );
	 
	# 只更新第一条记录：
	db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );
```

##### 3.  删除

```sql
# 删除
# 如删除集合下全部文档：
db.inventory.deleteMany({})

# 删除 status 等于 A 的全部文档：
db.inventory.deleteMany({ status : "A" })

# 删除 status 等于 D 的一个文档：
db.inventory.deleteOne( { status: "D" } )

# 删除一个或多个, 默认情况下删除多个， remove(query, justOne), 第二个参数传入true，只会删除一个
db.inventory.remove({status: "B"})
```

##### 4. 查询

````sql
# 查询语法
db.collection.find(query, projection)
# query ：可选，使用查询操作符指定查询条件
# projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略)

# 如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：
>db.col.find().pretty() # pretty() 方法以格式化的方式来显示所有文档。
> db.col.find().pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
# 除了 find() 方法之外，还有一个 findOne() 方法，它只返回一个文档。

# 如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：
````

![image-20210827165847650](mongoDB.assets/image-20210827165847650.png)

 ````sql
 # MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。
 >db.col.find({key1:value1, key2:value2}).pretty() 
 
 # 类似于 WHERE 语句：WHERE by='菜鸟教程' AND title='MongoDB 教程'
 db.col.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()
 
 # MongoDB OR 条件语句使用了关键字 $or,语法格式如下：
 >db.col.find(
    {
       $or: [
          {key1: value1}, {key2:value2}
       ]
    }
 ).pretty()
 # or 查询
 >db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
 
 # AND和OR联合使用，类似常规SQL语句为：'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'
 >db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
 ````

##### 5. $type 操作符

```sql
# 如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：
db.col.find({"title" : {$type : 2}})
或
db.col.find({"title" : {$type : 'string'}})
```

![image-20210827170946368](mongoDB.assets/image-20210827170946368.png)

##### 6. limit 与 skip 方法

```sql
# limit()方法基本语法如下所示：
>db.COLLECTION_NAME.find().limit(NUMBER)

# 以下实例为显示查询文档中的两条记录：
> db.col.find({},{"title":1,_id:0}).limit(2)
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
# 注：如果没有指定limit()方法中的参数则显示集合中的所有数据。

# 使用skip()方法来跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数。
>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)

# 以下实例只会显示第二条文档数据
>db.col.find({},{"title":1,_id:0}).limit(1).skip(1)
{ "title" : "Java 教程" }
# 注:skip()方法默认参数为 0 。
```

##### 7. 排序

````sql
# sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。
>db.COLLECTION_NAME.find().sort({KEY:1})

# col 集合中的数据按字段 likes 的降序排列：
>db.col.find({},{"title":1,_id:0}).sort({"likes":-1})
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
{ "title" : "MongoDB 教程" }
````

##### 8. 索引

```sql
# MongoDB使用 createIndex() 方法来创建索引。
>db.collection.createIndex(keys, options)
# 语法中 Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。

# 实例
>db.col.createIndex({"title":1})

# createIndex() 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引)
>db.col.createIndex({"title":1,"description":-1})

```

createIndex() 接收可选参数，可选参数列表如下：

![image-20210827172551112](mongoDB.assets/image-20210827172551112.png)

```sql
# 在后台创建索引：
db.values.createIndex({open: 1, close: 1}, {background: true})
# 通过在创建索引时加 background:true 的选项，让创建工作在后台执行

# 查看集合索引
db.col.getIndexes()

# 查看集合索引大小
db.col.totalIndexSize()

# 删除集合所有索引
db.col.dropIndexes()

# 删除集合指定索引
db.col.dropIndex("索引名称")
```

##### 9.聚合

聚合(aggregate)主要用于处理数据(诸如统计平均值，求和等)，并返回计算后的数据结果。有点类似 SQL 语句中的 **count(\*)**。

MongoDB中聚合的方法使用$\textcolor{Green}{aggregate()} $。

```SQL
# aggregate() 方法的基本语法格式如下所示
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

# 计算每个作者所写的文章数，使用aggregate()计算结果如下：
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "runoob.com",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
# 以上实例类似于SQL语句：
select by_user, count(*) from mycol group by by_user
```

![image-20210827180315337](mongoDB.assets/image-20210827180315337.png)



**管道的概念**

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

聚合框架中常用的几个操作：

- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $match：用于过滤数据，只输出符合条件的文档。\$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。
- $geoNear：输出接近某一地理位置的有序文档。

```sql
# $project实例
db.article.aggregate(
    { $project : {
        title : 1 ,
        author : 1 ,
    }}
 );
# 这样的话结果中就只还有_id,tilte和author三个字段，默认情况下_id字段是被包含的，如果要想不包含_id话可以这样:
db.article.aggregate(
    { $project : {
        _id : 0 ,
        title : 1 ,
        author : 1
    }});


# $match实例
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
# $match用于获取分数大于70小于或等于90记录，然后将符合条件的记录送到下一阶段$group管道操作符进行处理。

# $skip实例 , 前五个文档被"过滤"掉。
db.article.aggregate(
    { $skip : 5 });

```

#### 7. 复制(副本集)

复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。

复制还允许您从硬件故障和服务中断中恢复数据。

##### 1. 什么是复制？

- 保障数据的安全性
- 数据高可用性 (24*7)
- 灾难恢复
- 无需停机维护（如备份，重建索引，压缩）
- 分布式读取数据

##### 2. 复制原理？

复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。

mongodb各个节点常见的搭配方式为：一主一从、一主多从。

主节点记录在其上的所有操作==oplog==，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

MongoDB复制结构图如下所示：

![image-20210831162251886](mongoDB.assets/image-20210831162251886.png)

以上结构图中，客户端从主节点读取数据，在客户端写入数据到主节点时， 主节点与从节点进行数据交互保障数据的一致性。

###### 副本集特征：

- N 个节点的集群
- 任何节点可作为主节点
- 所有写入操作都在主节点上
- 自动故障转移
- 自动恢复

###### MongoDB副本集设置

```bash
# 通过指定 --replSet 选项来启动mongoDB。--replSet 基本语法格式如下：
mongod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"

# 启动一个名为rs0的MongoDB实例，其端口号为27017。
mongod --port 27017 --dbpath "D:\set up\mongodb\data" --replSet rs0

# 添加副本集的成员，我们需要使用多台服务器来启动mongo服务。进入Mongo客户端，并使用rs.add()方法来添加副本集的成员。
>rs.add(HOST_NAME:PORT)

# 假设你已经启动了一个名为mongod1.net，端口号为27017的Mongo服务
>rs.add("mongod1.net:27017")
```

#### 8. 分片

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。

##### 1. 为什么使用分片？

- 复制所有的写入操作到主节点

- 延迟的敏感数据会在主节点查询

- 单个副本集限制在12个节点

- 当请求量巨大时会出现内存不足。

- 本地磁盘不足

- 垂直扩展价格昂贵

  

MongoDB中使用分片集群结构分布：

![image-20210831163552014](mongoDB.assets/image-20210831163552014.png)

上图中主要有如下所述三个主要组件：

- Shard:

  用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障

- Config Server:

  mongod实例，存储了整个 ClusterMetadata，其中包括 chunk信息。

- Query Routers:

  前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。

##### 2. 分片实例

```bash
# 分片结构端口分布如下：
Shard Server 1：27020
Shard Server 2：27021
Shard Server 3：27022
Shard Server 4：27023
Config Server ：27100
Route Process：40000

# 步骤一：启动Shard Server
[root@100 /]# mkdir -p /www/mongoDB/shard/s0
[root@100 /]# mkdir -p /www/mongoDB/shard/s1
[root@100 /]# mkdir -p /www/mongoDB/shard/s2
[root@100 /]# mkdir -p /www/mongoDB/shard/s3
[root@100 /]# mkdir -p /www/mongoDB/shard/log
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27020 --dbpath=/www/mongoDB/shard/s0 --logpath=/www/mongoDB/shard/log/s0.log --logappend --fork
....
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27023 --dbpath=/www/mongoDB/shard/s3 --logpath=/www/mongoDB/shard/log/s3.log --logappend --fork

# 步骤二： 启动Config Server
[root@100 /]# mkdir -p /www/mongoDB/shard/config
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27100 --dbpath=/www/mongoDB/shard/config --logpath=/www/mongoDB/shard/log/config.log --logappend --fork
# 注意：这里我们完全可以像启动普通mongodb服务一样启动，不需要添加—shardsvr和configsvr参数。因为这两个参数的作用就是改变启动端口的，所以我们自行指定了端口就可以。

# 步骤三： 启动Route Process
/usr/local/mongoDB/bin/mongos --port 40000 --configdb localhost:27100 --fork --logpath=/www/mongoDB/shard/log/route.log --chunkSize 500
# mongos启动参数中，chunkSize这一项是用来指定chunk的大小的，单位是MB，默认大小为200MB.

# 步骤四： 配置Sharding
# 接下来，我们使用MongoDB Shell登录到mongos，添加Shard节点
[root@100 shard]# /usr/local/mongoDB/bin/mongo admin --port 40000
MongoDB shell version: 2.0.7
connecting to: 127.0.0.1:40000/admin
mongos> db.runCommand({ addshard:"localhost:27020" })
{ "shardAdded" : "shard0000", "ok" : 1 }
......
mongos> db.runCommand({ addshard:"localhost:27029" })
{ "shardAdded" : "shard0009", "ok" : 1 }
mongos> db.runCommand({ enablesharding:"test" }) #设置分片存储的数据库
{ "ok" : 1 }
mongos> db.runCommand({ shardcollection: "test.log", key: { id:1,time:1}})
{ "collectionsharded" : "test.log", "ok" : 1 }

# 步骤五： 程序代码内无需太大更改，直接按照连接普通的mongo数据库那样，将数据库连接接入接口40000
```

#### 9.数据备份与恢复

##### 1. 备份

==mongodump==命令可以通过参数指定导出的数据量级转存的服务器。

语法如下：

```sql
>mongodump -h dbhost -d dbname -o dbdirectory
```

- -h：

  MongoDB 所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017

- -d：

  需要备份的数据库实例，例如：test

- -o：

  备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个test目录，这个目录里面存放该数据库实例的备份数据。

##### 2. 恢复

使用 ==mongorestore== 命令来恢复备份的数据。

语法如下：

```sql
>mongorestore -h <hostname><:port> -d dbname <path>
```

- --host <:port>, -h <:port>：

  MongoDB所在服务器地址，默认为： localhost:27017

- --db , -d ：

  需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2

- --drop：

  恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用哦！

- <path>：

  mongorestore 最后的一个参数，设置备份数据所在位置，例如：c:\data\dump\test。

  你不能同时指定 <path> 和 --dir 选项，--dir也可以设置备份目录。

- --dir：

  指定备份的目录

  你不能同时指定 <path> 和 --dir 选项。 

#### 10. 监控

MongoDB中提供了==mongostat== 和 ==mongotop== 两个命令来监控MongoDB的运行情况。

启动你的Mongod服务，进入到你安装的MongoDB目录下的bin目录， 然后输入mongostat命令，如下所示：

```sql
bin>mongostat

bin>mongotop
```

#### 11.数据库引用

MongoDB 引用有两种：

- 手动引用（Manual References）
- DBRefs、

```sql
# DBRef的形式：
{ $ref : , $id : , $db :  }
# $ref：集合名称
# $id：引用的id
# $db:数据库名称，可选参数

# 实例中用户数据文档使用了 DBRef, 字段 address：
{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "runoob"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin"
}
```

#### 12. 覆盖索引查询

- 所有的查询字段是索引的一部分
- 所有的查询返回字段在同一个索引中

```sql
# 测试覆盖索引查询，使用以下 users 集合:
{
   "_id": ObjectId("53402597d852426020000002"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "gender": "M",
   "name": "Tom Benzamin",
   "user_name": "tombenzamin"
}

# 在 users 集合中创建联合索引，字段为 gender 和 user_name :
>db.users.ensureIndex({gender:1,user_name:1})

# 该索引会覆盖以下查询：
>db.users.find({gender:"M"},{user_name:1,_id:0})

# 实例没有排除_id，查询就不会被覆盖：
>db.users.find({gender:"M"},{user_name:1})

# 如果是以下的查询，不能使用覆盖索引查询：
 # 1. 所有索引字段是一个数组
 # 2. 所有索引字段是一个子文档
```

#### 13. 查询分析

查询分析常用函数有：==explain()== 和 ==hint()==。

##### 1. 使用 explain()

```sql
#  users 集合中创建 gender 和 user_name 的索引：
>db.users.ensureIndex({gender:1,user_name:1})

# 查询语句中使用 explain ：
>db.users.find({gender:"M"},{user_name:1,_id:0}).explain()

#  explain() 查询返回如下结果：
{
   "cursor" : "BtreeCursor gender_1_user_name_1",
   "isMultiKey" : false,
   "n" : 1,
   "nscannedObjects" : 0,
   "nscanned" : 1,
   "nscannedObjectsAllPlans" : 0,
   "nscannedAllPlans" : 1,
   "scanAndOrder" : false,
   "indexOnly" : true,
   "nYields" : 0,
   "nChunkSkips" : 0,
   "millis" : 0,
   "indexBounds" : {
      "gender" : [
         [
            "M",
            "M"
         ]
      ],
      "user_name" : [
         [
            {
               "$minElement" : 1
            },
            {
               "$maxElement" : 1
            }
         ]
      ]
   }
}
# indexOnly: 字段为 true ，表示我们使用了索引。
# cursor：因为这个查询使用了索引，MongoDB 中索引存储在B树结构中，所以这是也使用了 BtreeCursor 类型的游标。如果没有使用索引，游标的类型是 BasicCursor。这个键还会给出你所使用的索引的名称，你通过这个名称可以查看当前数据库下的system.indexes集合来得到索引的详细信息。
# n：当前查询返回的文档数量。
# nscanned/nscannedObjects：表明当前这次查询一共扫描了集合中多少个文档，我们的目的是，让这个数值和返回文档的数量越接近越好。
# millis：当前查询所需时间，毫秒数。
# indexBounds：当前查询具体使用的索引。
```

##### 2.  使用hint()

```sql
# 查询实例指定了使用 gender 和 user_name 索引字段来查询：
>db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1})

# 使用 explain() 函数来分析以上查询：
>db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1}).explain()
```

#### 14. 原子操作

==mongodb不支持事务==，但是mongodb提供了许多原子操作，比如文档的保存，修改，删除等，都是原子操作。

所谓原子操作就是要么这个文档保存到Mongodb，要么没有保存到Mongodb，不会出现查询到的文档没有保存完整的情况。

```sql
# 原子操作常用命令

# $set 用来指定一个键并更新键值，若键不存在并创建。
{ $set : { field : value } }

# $unset 用来删除一个键。
{ $unset : { field : 1} }

# $inc 可以对文档的某个值为数字型（只能为满足要求的数字）的键进行增减的操作。
{ $inc : { field : value } }

# $push 把value追加到field里面去，field一定要是数组类型才行，如果field不存在，会新增一个数组类型加进去。
{ $push : { field : value } }

# $pushAll 同$push,只是一次可以追加多个值到一个数组字段内。
{ $pushAll : { field : value_array } }

# $pull 从数组field内删除一个等于value值。
{ $pull : { field : _value } }

# $addToSet 增加一个值到数组内，而且只有当这个值不在数组内才增加。

# $pop 删除数组的第一个或最后一个元素
{ $pop : { field : 1 } }

# $rename 修改字段名称
{ $rename : { old_field_name : new_field_name } }

# $bit 位操作，integer类型
{$bit : { field : {and : 5}}}
```

#### 15. 索引限制

##### 1. 额外开销

每个索引占据一定的存储空间，在进行插入，更新和删除操作时也需要对索引进行操作。所以，如果你很少对集合进行读取操作，建议不使用索引。

##### 2. 内存(RAM)使用

由于索引是存储在内存(RAM)中,你应该确保该索引的大小不超过内存的限制。

如果索引的大小大于内存的限制，MongoDB会删除一些索引，这将导致性能下降。

##### 3. 查询限制

索引不能被以下的查询使用：

- 正则表达式及非操作符，如 $nin, \$not, 等。
- 算术运算符，如 $mod, 等。
- $where 子句

所以，检测你的语句是否使用索引是一个好的习惯，可以用explain来查看。

##### 4. 索引键限制

从2.6版本开始，如果现有的索引字段的值超过索引键的限制，MongoDB中不会创建索引。

##### 5. 插入文档超过索引键限制

如果文档的索引字段值超过了索引键的限制，MongoDB不会将任何文档转换成索引的集合。与mongorestore和mongoimport工具类似。

##### 6. 最大范围

- 集合中索引不能超过64个
- 索引名的长度不能超过128个字符
- 一个复合索引最多可以有31个字段


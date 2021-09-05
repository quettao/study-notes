## mongoDB

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
	 # update() 方法用于更新已存在的文档
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
	 	# update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后			面的
	 	# upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是				false，不插入。
	 	# multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来			多条记录全部更新。
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

MongoDB中聚合的方法使用$\textcolor{Red}{aggregate()} $。

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


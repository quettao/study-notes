# 

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

#### ![image-20210826181906899](mongoDB.assets/image-20210826181906899.png)

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

#### 创建数据库

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



#### mongoDB 语法

```bash
# 创建集合语法
db.createCollection(name, options)
	# name : 要创建的集合名称
	# options : 可选参数，指定有关内存大小及索引的选项。options参数如下图：
```

![image-20210827143002565](mongoDB.assets/image-20210827143002565.png)

```bash
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

# 
```

 
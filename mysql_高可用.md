### 1. mysql双机热备的实现

Mysql数据库没有增量备份的机制，当数据量太大的时候备份是一个很大的问题。还好mysql数据库提供了一种主从备份的机制，其实就是把主数据库的所有的数据同时写到备份的数据库中。实现mysql数据库的热备份。

#### 1. Mysql 建立主－从服务器双机热备配置步骤

##### [环境描述]

```
A服务器（主服务器Master）：59.151.15.36

B服务器（从服务器Slave）：218.206.70.146

主从服务器的Mysql版本皆为5.5.17
```

##### 1. 主服务器Master配置

###### 1. 创建同步用户

进入mysql操作界面，在主服务器上为从服务器建立一个连接帐户，该帐户必须授予REPLICATION SLAVE权限。因为从mysql版本3.2以后就可以通过REPLICATION对其进行双机热备的功能操作。

操作指令如下：

```mysql
mysql> grant replication slave on *.* to 'replicate'@'218.206.70.146' identified by '123456';

mysql> flush privileges;
```

创建好同步连接帐户后，我们可以通过在从服务器（Slave）上用replicat帐户对主服务器（Master）数据库进行访问下，看下是否能连接成功。

在从服务器（Slave）上输入如下指令：

```mysql
[root@YD146 ~]# mysql -h59.151.15.36 -ureplicate -p123456
```

如果出现下面的结果，则表示能登录成功，说明可以对这两台服务器进行双机热备进行操作。

![image-20220705172131135](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705172131135.png)

###### 2. 修改mysql配置文件

首先找到mysql配置所有在目录，一般在安装好mysql服务后，都会将配置文件复制一一份出来放到/ect目录下面，并且配置文件命名为：my.cnf。即配置文件准确目录为/etc/my.cnf

````mysql
[mysqld]

server-id = 1
log-bin=mysql-bin                //其中这两行是本来就有的，可以不用动，添加下面两行即可
binlog-do-db = test
binlog-ignore-db = mysql
````

###### 3. 重启mysql服务

修改完配置文件后，保存后，重启一下mysql服务，如果成功则没问题。

![image-20220705172757673](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705172757673.png)

###### 4. 查看主服务器状态

进入mysql服务后，可通过指令查看Master状态，输入如下指令：

![image-20220705172837965](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705172837965.png)

注意看里面的参数，特别前面两个File和Position，在从服务器（Slave）配置主从关系会有用到的。

注：这里使用了锁表，目的是为了产生环境中不让进新的数据，好让从服务器定位同步位置，初次同步完成后，记得解锁。

![image-20220705172910615](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705172910615.png)



##### 2. 从服务器配置

###### 1.修改配置文件

因为这里面是以主－从方式实现mysql双机热备的，所以在从服务器就不用在建立同步帐户了，直接打开配置文件my.cnf进行修改即可，道理还是同修改主服务器上的一样，只不过需要修改的参数不一样而已。如下：

````mysql
[mysqld]

server-id = 2
log-bin=mysql-bin
replicate-do-db = test
replicate-ignore-db = mysql,information_schema,performance_schema
````

###### 2. 重启服务

修改完配置文件后，保存后，重启一下mysql服务，如果成功则没问题。

![image-20220705173131881](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173131881.png)

###### 3. 用change master 语句指定同步位置

这步是最关键的一步了，在进入mysql操作界面后，输入如下指令：

```mysql
mysql>stop slave;          //先停步slave服务线程，这个是很重要的，如果不这样做会造成以下操作不成功。
mysql>change master to
>master_host='59.151.15.36',master_user='replicate',master_password='123456',
> master_log_file=' mysql-bin.000016 ',master_log_pos=107;
```

注：master_log_file, master_log_pos由主服务器（Master）查出的状态值中确定。也就是刚刚叫注意的。master_log_file对应File, master_log_pos对应Position。Mysql 5.x以上版本已经不支持在配置文件中指定主服务器相关选项。

遇到的问题，如果按上面步骤之后还出现如下情况：

![image-20220705173321022](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173321022.png)

则要重新设置slave。指令如下

```mysql
mysql>stop slave;
mysql>reset slave;
```

之后停止slave线程重新开始。成功后，则可以开启slave线程了。

```mysql
mysql>start slave;
```

###### 4. 查看从服务器(slave)状态

用如下指令进行查看

```mysql
mysql> show slave status\G;
```

![image-20220705173521178](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173521178.png)

查看下面两项值均为Yes，即表示设置从服务器成功。

```mysql
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```



##### 3. 测试同步

之前开始已经说过了在数据库test只有一个表tb_mobile没有数据，我们可以先查看下两服务器的数据库是否有数据：

Master:59.151.15.36

![image-20220705173729739](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173729739.png)

Slave:218.206.70.146

![image-20220705173744513](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173744513.png)

好了，现在可以在Master服务器中插入数据看下是否能同步。

Master:59.151.15.36

![image-20220705173808514](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173808514.png)

Slave:218.206.70.146

![image-20220705173825090](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705173825090.png)

可以从上面两个截图上看出，在Master服务器上进行插入的数据在Slave服务器可以查到，这就表示双机热备配置成功了。



#### 2. Mysql 建立主－主服务器双机热备配置步骤

##### 1. 创建同步用户

同时在主从服务器建立一个连接帐户，该帐户必须授予REPLIATION SLAVE权限。这里因为服务器A和服务器B互为主从，所以都要分别建立一个同步用户。

服务器A：

```mysql
mysql> grant replication slave on *.* to 'replicate'@'218.206.70.146' identified by '123456';

mysql> flush privileges;
```

服务器B：

```mysql
mysql> grant replication slave on *.* to 'replicate'@'59.151.15.36' identified by '123456';

mysql> flush privileges;
```

##### 2. 修改配置文件 my.cnf

服务器A

```mysql
[mysqld]

server-id = 1
log-bin=mysql-bin 
binlog-do-db = test
binlog-ignore-db = mysql

#主－主形式需要多添加的部分
log-slave-updates
sync_binlog = 1
auto_increment_offset = 1
auto_increment_increment = 2
replicate-do-db = test
replicate-ignore-db = mysql,information_schema
```

服务器B：

```mysql
[mysqld]

server-id = 2
log-bin=mysql-bin 
master-slave need
replicate-do-db = test
replicate-ignore-db = mysql,information_schema,performance_schema

#主－主形式需要多添加的部分
binlog-do-db = test
binlog-ignore-db = mysql
log-slave-updates
sync_binlog = 1
auto_increment_offset = 2
auto_increment_increment = 2
```

##### 3. 分别重启A服务器和B服务器上的mysql服务

重启服务器方式和上面的一样

##### 4. 分别查A服务器和B服务器作为主服务器的状态

服务器A：

![image-20220705174511963](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705174511963.png)

服务器B：

![image-20220705174525346](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705174525346.png)



##### 5.分别在A服务器和B服务器上用change master to 指定同步位置

服务器A：

```mysql
mysql>change master to
>master_host='218.206.70.146',master_user='replicate',master_password='123456',
> master_log_file=' mysql-bin.000011 ',master_log_pos=497;
```

服务器B：

```mysql
mysql>change master to
>master_host='59.151.15.36',master_user='replicate',master_password='123456',
> master_log_file=' mysql-bin.000016 ',master_log_pos=107;
```



##### 6. 分别在A和B服务器上重启从服务线程

```mysql
mysql>start slave;
```



##### 7. 分别在A和B服务器上查看从服务器状态

```mysql
mysql>show slave status\G;
```

查看下面两项值均为Yes，即表示设置从服务器成功。

Slave_IO_Running: Yes

Slave_SQL_Running: Yes



##### 8. 测试主－主同步例子

测试服务器A：

在服务器A上插入一条语句如下图所示：

![image-20220705174800995](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705174800995.png)

之后在服务器B上查看是否同步如下图所示：

![image-20220705174814626](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705174814626.png)

测试服务器B：

在服务器B上插入一条语句如下图所示：

![image-20220705174830790](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705174830790.png)

然后在从服务器A上查看是否有同步数据如下图所示：

![image-20220705174851204](mysql_%E9%AB%98%E5%8F%AF%E7%94%A8.assets/image-20220705174851204.png)

最后从结果可以看出主－主形式的双机热备是能成功实现的。



#### 3. 配置参数说明

```mysql
Server-id

# ID值唯一的标识了复制群集中的主从服务器，因此它们必须各不相同。Master_id必须为1到232－1之间的一个正整数值，slave_id值必须为2到232－1之间的一个正整数值。

Log-bin

# 表示打开binlog，打开该选项才可以通过I/O写到Slave的relay-log，也是可以进行replication的前提。

Binlog-do-db

# 表示需要记录二进制日志的数据库。如果有多个数据可以用逗号分隔，或者使用多个binlog-do-dg选项。

Binglog-ingore-db

# 表示不需要记录二进制日志的数据库，如果有多个数据库可用逗号分隔，或者使用多binglog-ignore-db选项。

Replicate-do-db

# 表示需要同步的数据库，如果有多个数据可用逗号分隔，或者使用多个replicate-do-db选项。

Replicate-ignore-db

# 表示不需要同步的数据库，如果有多个数据库可用逗号分隔，或者使用多个replicate-ignore-db选项。

Master-connect-retry

# master-connect-retry=n表示从服务器与主服务器的连接没有成功，则等待n秒（s）后再进行管理方式（默认设置是60s）。如果从服务器存在mater.info文件，它将忽略些选项。

Log-slave-updates

# 配置从库上的更新操作是否写入二进制文件，如果这台从库，还要做其他从库的主库，那么就需要打这个参数，以便从库的从库能够进行日志同步。

Slave-skip-errors

# 在复制过程，由于各种原因导致binglo中的sql出错，默认情况下，从库会停止复制，要用户介入。可以设置slave-skip-errors来定义错误号，如果复制过程中遇到的错误是定义的错误号，便可以路过。如果从库是用来做备份，设置这个参数会存在数据不一致，不要使用。如果是分担主库的查询压力，可以考虑。

Sync_binlog=1 Or N

# Sync_binlog的默认值是0，这种模式下，MySQL不会同步到磁盘中去。这样的话，Mysql依赖操作系统来刷新二进制日志binary log，就像操作系统刷新其他文件的机制一样。因此如果操作系统或机器（不仅仅是Mysql服务器）崩溃，有可能binlog中最后的语句丢失了。要想防止这种情况，可以使用sync_binlog全局变量，使binlog在每Ｎ次binlog写入后与硬盘同步。当sync_binlog变量设置为１是最安全的，因为在crash崩溃的情况下，你的二进制日志binary log只有可能丢失最多一个语句或者一个事务。但是，这也是最慢的一种方式（除非磁盘有使用带蓄电池后备电源的缓存cache,使得同步到磁盘的操作非常快）。

# 即使sync_binlog设置为１，出现崩溃时，也有可能表内容和binlog内容之间存在不一致性。如果使用InnoDB表，Mysql服务器处理COMMIT语句，它将整个事务写入binlog并将事务提交到InnoDB中。如果在两次操作之间出现崩溃，重启时，事务被InnoDB回滚，但仍然存在binlog中。可以用-innodb-safe-binlog选项来增加InnoDB表内容和binlog之间的一致性。（注释：在Mysql 5.1版本中不需要-innodb-safe-binlog；由于引入了XA事务支持，该选项作废了），该选项可以提供更大程度的安全，使每个事务的binlog(sync_binlog=1)和（默认情况为真）InnoDB日志与硬盘同步，该选项的效果是崩溃后重启时，在滚回事务后，Mysql服务器从binlog剪切回滚的InnoDB事务。这样可以确保binlog反馈InnoDB表的确切数据等，并使从服务器保持与主服务器保持同步（不接收回滚的语句）。

Auto_increment_offset和Auto_increment_increment

# Auto_increment_increment和auto_increment_offset用于主－主服务器（master-to-master）复制，并可以用来控制AUTO_INCREMENT列的操作。两个变量均可以设置为全局或局部变量，并且假定每个值都可以为1到65,535之间的整数值。将其中一个变量设置为0会使该变量为1。

# 这两个变量影响AUTO_INCREMENT列的方式：auto_increment_increment控制列中的值的增量值，auto_increment_offset确定AUTO_INCREMENT列值的起点。

# 如果auto_increment_offset的值大于auto_increment_increment的值，则auto_increment_offset的值被忽略。例如：表内已有一些数据，就会用现在已有的最大自增值做为初始值。
```

#### 如何解决MySQL主从同步错误的SQL

解决：

stop slave;

\#表示跳过一步错误，后面的数字可变

set global sql_slave_skip_counter =1;

start slave;

之后再用mysql> show slave status\G 查看：

Slave_IO_Running: Yes

Slave_SQL_Running: Yes
#### 查看磁盘大文件

```
find /pdata1/ -xdev -type f -size +50M -print | xargs ls -lh | sort -k5,5 -h -r
```

#### 查看磁盘删除未释放文件

```
lsof |grep delete | more
```

#### **查看当前系统打开的文件数量:**

```
cat /proc/sys/fs/file-nr
```

#### **查看当前进程的打开文件数量：**

````
lsof -p pid | wc -l   （lsof -p 1234 | wc -l ）
````

#### 判断是不是连接池中的连接有没有正确关闭

```
# cat /proc/{pid}/net/sockstat
sockets: used 109
TCP: inuse 4 orphan 0 tw 14 alloc 9 mem 2
UDP: inuse 5 mem 16
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0
# 如果 TCP inuse 很大，就很有可能是连接没有正确关闭。

sockets: used：已使用的所有协议套接字总量
TCP: inuse：正在使用（正在侦听）的TCP套接字数量。其值≤ netstat –lnt | grep ^tcp | wc –l
TCP: orphan：无主（不属于任何进程）的TCP连接数（无用、待销毁的TCP socket数）
TCP: tw：等待关闭的TCP连接数。其值等于netstat –ant | grep TIME_WAIT | wc –l
TCP：alloc(allocated)：已分配（已建立、已申请到sk_buff）的TCP套接字数量。其值等于netstat –ant | grep ^tcp | wc –l
TCP：mem：套接字缓冲区使用量（单位不详。用scp实测，速度在4803.9kB/s时：其值=11，netstat –ant 中相应的22端口的Recv-Q＝0，Send-Q≈400）
UDP：inuse：正在使用的UDP套接字数量
RAW：
FRAG：使用的IP段数量
```

#### linux查看网络并发连接

```
netstat -pnt | grep :80  # :80端口号
```

#### 查看打开句柄总数

```
lsof -n|awk '{print $2}'|wc -l
```

查看系统中进程占用的句柄数，根据打开文件句柄的数量降序排列，其中第二列为进程ID：

```
lsof -n|awk '{print $2}'|sort|uniq -c|sort -nr|more
```



linux查看网络并发连接数

```
netstat -pnt | grep :80 | wc -l
```

#### 查看CLOSE_WAIT是哪台服务器请求的，以及请求数

```
# 查看CLOSE_WAIT是哪台服务器请求的，以及请求数
netstat -a |grep "CLOSE_WAIT"|awk '{print $5}'|awk -F '.' '{print $1}' |sort | uniq -c | sort -nr

# 查看TIME_WAIT是哪台服务器请求的，以及请求数
netstat -a |grep "TIME_WAIT"|awk '{print $5}'|awk -F '.' '{print $1}' |sort | uniq -c | sort -nr
```



#### 报错： write: broken pipe

这个错误的字面意思为：客户端可能会在收到服务端响应之前断开连接

##### 连接数过大

想到的问题原因可能是以下两种原因：

- 服务端所在的服务器，超过了最大连接数
- 客户端在接收到服务端响应之前断开连接

- 使用以下命令查看当前最大连接数：

  ```
  # ulimit -n 
  1024
  ```



##### 调用者在接收到服务端响应之前断开连接

问题可能是因为：client端获取响应数据，突然异常退出或直接close连接。

client退出后，server将会发送两次数据，server第一次发送给client，client返回给server RST。第二次在这个RST的连接上，server继续发送，出现broken pipe。即如下错误：

```
2022/04/11 12:27:53 http: panic serving 172.31.34.109:45842: write tcp 172.31.6.64:9093->172.31.34.109:45842: write: broken pipe
```

那么此时就会出现两个问题，server端迟迟没有响应数据给client端，那么也有可能是以下两个原因：

- server端去节点请求数据的时候，节点一直没有返回
- 连接数过高，请求需要排队处理，可能有的请求还没有处理完，就已经请求超时了

```
关于此问题已向client端进行求证，目前连接server端设置的超时时间是：连接超时：0.5秒，响应超时5秒
```

###### 排查服务器上的连接数

- 查看系统连接数

```
# 获取当前socket连接状态统计信息
# cat /proc/net/sockstat
sockets: used 1138
TCP: inuse 21 orphan 0 tw 0 alloc 786 mem 220
UDP: inuse 4 mem 2
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0


# 统计当前各种状态的连接的数量的命令
# netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
CLOSE_WAIT 1020
ESTABLISHED 59
```

- 查看端口范围

````
# 允许系统打开的端口范围，用于向外链接的端口范围
# cat /proc/sys/net/ipv4/ip_local_port_range
32768	60999
````

- 查看文件打开数

```
# 查看当前打开文件数 
# 如果执行缓慢，那么执行结果显示系统当前打开文件数500w
# lsof | wc -l
4792741


# 把当前打开文件列表保存
# lsof>>/tmp/lsof.log  


# 查看 5-6万 行之间的数据
sed -n '50000,60000p' /tmp/lsof.log
```

结果，bsc，且没有关闭：

```
COMMAND      PID         PPID      USER       PGID      FD          DEVICE       SIZE     NODE     NAME
进程名称 进程标识符 父进程标识符 进程所有者 进程所属组 文件描述符   指定磁盘名称     文件大小   索引节点  打开文件的确切名称
bsc-wallet  2105       2125        root      1872u      sock       0,7           0t0    2800355 protocol: TCPv6
bsc-wallet  2105       2125        root      1873u      sock       0,7           0t0    2802021 protocol: TCPv6
bsc-wallet  2105       2125        root      1874u      sock       0,7           0t0    2765499 protocol: TCPv6
bsc-wallet  2105       2125        root      1875u      sock       0,7           0t0    2803112 protocol: TCPv6
bsc-wallet  2105       2125        root      1876u      sock       0,7           0t0    2769274 protocol: TCPv6
bsc-wallet  2105       2125        root      1877u      sock       0,7           0t0    2803114 protocol: TCPv6
bsc-wallet  2105       2125        root      1878u      sock       0,7           0t0    2770305 protocol: TCPv

```

###### 查看连接状态为CLOSE_WAIT的连接情况

- 统计当前各种状态的连接的数量的命令

```
#  netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
CLOSE_WAIT 1169
ESTABLISHED 84
```

- 查看CLOSE_WAIT是哪台服务器请求的，以及请求数

```
# netstat -a |grep "CLOSE_WAIT"|awk '{print $5}'|awk -F '.' '{print $1}' |sort | uniq -c | sort -nr
1382 ip-172-31-34-109

```

##### 延时测试

可以看到情况为：另外一台服务器上面的服务（客户端）请求到bsc-clinet（服务端）时，由于在他设置的超时时间内未返回请求数据，客户端主动关闭了连接，导致服务端所在的服务器出现大量关于该服务端的CLOSE_WAIT的连接状态，最后服务端无法提供服务，报错： write: broken pipe

服务端未出现 CLOSE_WAIT 时的响应时间及延时

- 服务端目前没有CLOSE_WAIT连接状态

```
#  netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
ESTABLISHED 21
```

- 获取服务端请求响应时间

````
# curl -o /dev/null -s -w %{time_namelookup}::%{time_connect}::%{time_starttransfer}::%{time_total}::%{speed_download}"\n" -d "number=16619272&sign=b8aaab58e4d73753430c473ba29756d3" http://127.0.0.1:9093/api/block/blockInfo

0.000::0.000::0.088::0.088::1572245.000
````

```
-o：把curl 返回的html、js 写到垃圾回收站[ /dev/null]
-s：去掉所有状态
-w：按照后面的格式写出rt
time_namelookup：DNS 解析域名www.36nu.com的时间
time_commect：client和server端建立TCP 连接的时间
time_starttransfer：从client发出请求；到web的server 响应第一个字节的时间
time_total：client发出请求；到web的server发送会所有的相应数据的时间
speed_download：下周速度 单位 byte/s

上面这条命令及返回结果可以这么理解：
0.000: DNS 服务器解析www.36nu.com 的时间单位是s
0.000: client发出请求，到c/s 建立TCP 的时间；里面包括DNS解析的时间
0.088: client发出请求；到s响应发出第一个字节开始的时间；包括前面的2个时间
0.088: client发出请求；到s把响应的数据全部发送给client；并关闭connect的时间
1572245.000 ：下载数据的速度
建立TCP连接到server返回client第一个字节的时间：0.088s – 0.000s = 0.088s
server把响应数据发送给client的时间：0.088s – 0.088s = 0.000s
```

- 模拟客户端超时时间：连接超时：1秒，响应超时5秒

使用CURL时，有两个超时时间：一个是连接超时时间，另一个是数据传输的最大允许时间。

连接超时时间用–connect-timeout参数来指定，数据传输的最大允许时间用-m参数来指定。

```
# curl --connect-timeout 1 -m 5 -d "number=16619272&sign=b8aaab58e4d73753430c473ba29756d3" http://127.0.0.1:9093/api/block/blockInfo

```

##### 服务端出现 CLOSE_WAIT 时的响应时间及延时

- 服务端目有大量有CLOSE_WAIT连接状态

```
# netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
CLOSE_WAIT 1185
ESTABLISHED 31
```

- 获取服务端请求响应时间

```
# curl -o /dev/null -s -w %{time_namelookup}::%{time_connect}::%{time_starttransfer}::%{time_total}::%{speed_download}"\n" -d "number=16619272&sign=b8aaab58e4d73753430c473ba29756d3" http://127.0.0.1:9093/api/block/blockInfo

```

### awk

awk工作原理：均是一行一行的读取，处理；awk将一行分成数个字段来处理

#### awk语法

```linux
awk ‘BEGIN {commands} pattern {commands}END{commands}' file1
BEGIN:处理数据前执行的命令
END：处理数据后执行的命令
pattern：模式，每一行都执行的命令
BEGIN和END里的命令只是执行一次
pattern里的命令会匹配每一行去处理

# 例如 
awk -F ":" '{print $1,$2,$5}' /etc/passwd | head -5

# -F ":"  ：  awk选项，指定输入分割符为：，
# '{print}'    : 固定语法
# $1,$2,$5  :输出第一个，第二个，第五个字段
# ，  ： 是输出分隔符，如果不加默认是没有分隔符的。
```


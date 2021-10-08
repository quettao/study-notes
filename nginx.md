### nginx

#### 配置文件结构

main block : 主配置段，也即全局配置段
event {...} : 事件驱动相关的配置；
http {...} : http/httpd协议相关的配置段；
mail {…}  : 邮件代理模块
stream {...} 

#### main配置段常见的配置指令：

分类：
    正常运行必备的配置
    优化性能相关的配置
    用于调试及定位问题相关的配置
    事件驱动相关的配置

##### 1. 正常运行必备的配置

```nginx
# 定义工作进程使用的用户和组，如果省略组，则使用名称等于用户名的组
user

# 指定存储nginx主进程进程号码的文件路径。
pid /PATH/TO/PID_FILE;

# 指明包含进来的其他配置文件片段
include file | mask ;

# 指明要装载的动态模块
load_module file
```

##### 2. 性能优化相关的配置

```nginx
# worker进程的数量；通常应该等于小于当前主机的cpu的物理核心数； auto：当前主机物理cpu核心数；
worker_processes number | auto;

# worker_cpu_affinity auto [cpumask];
# CPU MASK:
#		00000000:表示8颗cpu核心
#		00000001:0号cpu
#		00000010:1号cpu
worker_cpu_affinity cpumask ...

# 指定worker进程的nice值，设定worker进程的优先级 优先级范围：[-20,20]
worker_priority number 

# worker进程能够打开的文件数量上限；
worker_rlimit_nofile number
```

##### 3. 调试、定位问题

```nginx
# 是否以守护进程方式运行Nginx；
daemon on | off

# 是否以master/worker模型运行nginx; 默认为on;
master_process on | off;

# 定义错误日志路径与级别；
error_log file [level];
```

##### 4. 事件驱动相关的配置

```nginx
# 每个worker进程所能打开的最大并发连接数量；总的最大并发响应数量：worker_reocesses * worker_connections
worker_connections number

# 指明并发连接请求的处理方法 例如： use epoll;
use method

# 处理新的连接请求的方法
# on：意味着由各worker轮流处理新请求
# off：意味着每个新请求的到达都会通知所有的worker进程
accept——mutex on | off：
```

#### 定义nginx运行的用户和用户组

```nginx
# 改为特殊的用户和组
user nginx nginx; 

# nginxworker进程数，即处理请求的进程（熟称负责接客的服务员）

# 初始可设置为CPU总核数
worker_processes 8; 

# 一定要设置warn级别以上
error_log logs/error.log error; 

# 用于管理nginx进程
pid logs/nginx.pid; 

# 最大打开文件数
worker_rlimit_nofile 65535; 

# nginx最大连接数 = worker连接数 * worker进程数
worker_connections 10240;
```

#### http模块设置部分

```nginx
# 隐藏响应header和错误通知中的版本号
server_tokens off;

# 文件扩展名与文件类型映射表
include mime.types;

# 默认文件类型
default_type application/octet-stream;

# 服务域名的最大hash表大小
server_names_hash_max_size 512;

# 服务域名的hash表大小
server_names_hash_bucket_size 128;

# 开启高效文件传输模式，实现内核零拷贝
sendfile on;

# 激活tcp_nopush参数可以允许把httpresponse header和文件的开始放在一个文件里发布，积极的作用是减少网络报文段的数量
tcp_nopush on;

# 激活tcp_nodelay，内核会等待将更多的字节组成一个数据包，从而提高I/O性能
tcp_nodelay on;

# 连接超时时间，单位是秒
keepalive_timeout 120;

# 目录列表访问参数，合适http下载，默认关闭。
autoindex off;

# 读取客户端请求头的超时时间
client_header_timeout 15s;

# 读取客户端请求主体的超时时间
client_body_timeout 60s;

# 设定读取客户端请求主体的最大大小
client_max_body_size 8m;

# 设置服务器端传送http响应信息到客户端的超时时间
send_timeout 60s;
```

##### FastCGI参数是和动态服务器交互起作用的参数

~~~nginx
# 设定Nginx服务器和后端FastCGI服务器连接的超时时间
fastcgi_connect_timeout 60; 

# 设定Nginx允许FastCGI服务端返回数据的超时时间
fastcgi_send_timeout 60; 

# 设定Nginx从FastCGI服务端读取响应信息的超时时间
fastcgi_read_timeout 60; 

# 设定用来读取从FastCGI服务端收到的第一部分响应信息的缓冲区大小
fastcgi_buffer_size 64k; 

# 设定用来读取从FastCGI服务端收到的响应信息的缓冲区大小以及缓冲区数量
fastcgi_buffers 4 64k; 

# 设定系统很忙时可以使用的fastcgi_buffers大小，推荐大小为fastcgi_buffers *2。
fastcgi_busy_buffers_size 128k; 

# fastcti临时文件的大小，可设置128-256K
fastcgi_temp_file_write_size 128k; 
~~~

#####  gzip压缩模块部分（此部分对于网站优化极其重要）

```nginx
# 开启gzip压缩功能。
gzip on; 

# 设置允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取。默认值是0，表示不管页面多大都进行压缩。建议设置成大于1K。如果小于1K可能会越压越大。
gzip_min_length 1k; 

# 压缩缓冲区大小。表示申请4个单位为16K的内存作为压缩结果流缓存，默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果。
gzip_buffers 4 16k; 

# 压缩版本（默认1.1，前端为squid2.5时使用1.0）用于设置识别HTTP协议版本，默认是1.1，目前大部分浏览器已经支持GZIP解压，使用默认即可。
gzip_http_version 1.1; 

# 压缩比率。用来指定GZIP压缩比，1压缩比最小，处理速度最快；9压缩比最大，传输速度快，但处理最慢，也比较消耗cpu资源。
gzip_comp_level 2; 

# 用来指定压缩的类型，“text/html”类型总是会被压缩，这个就是HTTP原理部分讲的媒体类型。
gzip_types text/plain application/x-javascript text/css application/xml; 

# vary header支持。该选项可以让前端的缓存服务器缓存经过GZIP压缩的页面，例如用Squid缓存经过Nginx压缩的数据。
gzip_vary on; 
```

##### 反向代理负载均衡设定部分（可选）

```nginx
# upstream表示负载服务器池，定义名字为blog.oldboyedu.com的服务器池
upstream test {
  #server是服务器节点起始标签，其后是节点地址，可为域名或IP，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
  ip_hash;#调度算法，默认是rr轮询。
  server 172.16.1.7:80 weight=1;
  server 172.16.1.8:80 weight=1;
  server 172.16.1.9:80 weight=1 backup;#backup表示热备
}
```

##### 设定基于域名的虚拟主机部分

```nginx
server {
  listen       80;#监听的端口，也可以是172.16.1.7:80形式
  server_name www.oldboyedu.com oldboyedu.com;#域名
  root   html/blog;#站点根目录，即网站程序放的目录
  location / {#默认访问的location标签段
  index  index.php index.html index.htm;#首页排序

}
  
location ~.*.(php|php5)?$ {
    #符合php扩展名的请求调度到fcgi server
		fastcgi_pass 127.0.0.1:9000;#抛给本机的9000端口(php fastcgi server)
		fastcgi_index index.php;#设定动态首页
		include fastcgi.conf;#设定和fastcgi交互的相关参数包含文件
}
  
#将符合静态文件的图片视频流媒体等设定expries缓存参数，要求浏览器缓存。
location~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
  expires      10y;#客户端缓存上述静态数据10年
}

#将符合js,css文件的等设定expries缓存参数，要求浏览器缓存。
location~ .*\.(js|css)?$ {
  expires 30d;#客户端缓存上述js,css数据30天
}

#根据日志格式记录用户访问的日志
access_log /app/logs/www_access.log  main;
```

##### 反向代理负载均衡配置（代理blog.oldboyedu.com服务）

```nginx
server {

listen       80; # 监听的端口，也可以是172.16.1.7:80形式

server_name  blog.oldboyedu.com; # 代理的服务域名

  location / {
     # 将访问blog.oldboyedu.com的所有请求都发送到upstream定义的服务器节点池。
       proxy_pass http://blog.oldboyedu.com;
     # 在代理向后端服务器发送的http请求头中加入host字段信息，用于当后端服务器配置有多个虚拟主机时，可以识别代理的是哪个虚拟主机。这是节点服务器多虚拟主机时的关键配置。
       proxy_set_headerHost  $host;
     # 在代理向后端服务器发送的http请求头中加入X-Forwarded-For字段信息，用于后端服务器程序、日志等接收记录真实用户的IP，而不是代理服务器的IP。
       proxy_set_header X-Forwarded-For $remote_addr;
     # 设定反向代理与后端节点服务器连接的超时时间，即发起握手等候响应的超时时间。
       proxy_connect_timeout 60;
     # 设定代理后端服务器的数据回传时间
       proxy_send_timeout 60;
     # 设定Nginx从代理的后端服务器获取信息的时间
       proxy_read_timeout 60;
     # 设定缓冲区的大小
       proxy_buffer_size 4k;
     # 设定缓冲区的数量和大小。nginx从代理的后端服务器获取的响应信息，会放置到缓冲区。
       proxy_buffers 4 32k;
     # 设定系统很忙时可以使用的proxy_buffers大小
       proxy_busy_buffers_size 64k;
     # 设定proxy缓存临时文件的大小
       proxy_temp_file_write_size 64k;
       # 对于以上参数的详细理解可见本文开头图解。
  }
  # 反向代理如果并发大，务必要关闭日志，否则IO吃紧。
  access_log off;
}
```

##### 设定查看Nginx状态的地址

```nginx
location /status {
   stub_status on;#开启状态功能
   access_log off;#关闭记录日志
   auth_basic “Oldboy Server Status”;#设置基本认证提示
   auth_basic_user_file conf/htpasswd;#校验密码文件
}

```


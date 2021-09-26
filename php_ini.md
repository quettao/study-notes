### PHP 配置文件

##### php ini

```ini
# 用于设定单个PHP进程可以使用的系统内存最大值；default:128M
memory_limit

# 获取内存的内存使用量函数
memory_get_peak_usage()

# 设置单个 PHP 进程在终止之前最长可运行时间 default:30s; 在 PHP 脚本中可以调用 set_limit_time()函数覆盖这个设置
max_execution_time

# 除了提示外包含所有的错误默认值是 E_ALL & ~ E_notice。设置错误报告的级别。该参数可以是一个任意的表示二进制位字段的整数，或者常数名称;在程序运行时，还可以通过 error_reporting() 函数进行设置
error_reporting = E_ALL & ~E_NOTICE

# 许将错误消息标记为与其他的文本不同的颜色
error_prepend_string = [""]

# 允许包含或需要这些目录中的文件即可;这些目录一般是你文档的根目录;多个目录以冒号分隔：/usr/local/apache/htdocs:/usr/local/lib
include_path = [DIR]

# 如果指定路径，每个 PHP 文件的开头必须自动 include() 。包含路径设置适用。
auto-prepend-file = [path/to/file]

# 如果指定路径，每个 PHP 文件的结尾必须自动include()。除非你通过使用 exit() 函数来避免。包含路径设置适用。
auto-append-file = [path/to/file]

# 如果你使用安全模式或如果要在你站点部分启用 PHP，在此处设置此值（比如，仅在您网页根目录的一个子目录中）
doc_root = [DIR]

# 如果使用 PHP 脚本上传文件则打开此标志。
file_uploads = [on/off]

# 如果您明白 HTTP 上传的具体影响，请注释这条！
upload_tmp_dir = [DIR]

# 该配置是设置：最大上传文件大小
upload_max_filesize

# 允许同时上传的最大文件数量
max_file_uploads

# 设置允许的发布数据的最大大小。 该设置也会影响文件上传。 要上传大文件，该值必须大于upload_max_filesize。
# 如果配置脚本启用了内存限制，memory_limit也会影响文件上传。 一般来说，memory_limit应该大于post_max_size。 
# 如果post数据的大小大于post_max_size，那么$ _POST和$ _FILES超全局变量就是空的。
# 这可以通过各种方式进行跟踪，例如， 通过将$ _GET变量传递给处理数据的脚本，
# 即<form action =“edit.php？processed = 1”>，然后检查是否设置了$ _GET ['processed']。
post_max_size

# 设置客户端断开连接时是否中断脚本的执行
ignore_user_abort = [On/Off]

# 如果不指定任何其他主机时，服务器主机默认使用最初连接到的数据库服务器的主机
mysql.default_host = hostname

# 如果未指定主机名，默认使用最初连接的服务器名
mysql.default_user = username

# 如果不指定设置密码则默认使用最初连接到的服务器密码
mysql.default_password = password

# 本选项激活了 URL 形式的 fopen 封装协议使得可以访问 URL 对象例如文件
allow_url_fopen

# 本选项用于配置：是否可以使用 include require include_once require_once 进行远程包含文件
allow_url_include

# 在未设定环境变量时用于所有日期／时间函数的默认时区。
date.timezone=Asia/Shanghai

# 该选项设置是否将错误信息作为输出的一部分显示到屏幕，或者对用户隐藏而不显示
display_errors

# 出于网站安全，禁止显示php的版本号，防止别人针对特定php版本漏洞攻击网
expose_php = On|off

# php扩展安装目录
extension_dir = "D:\php\ext"

# 将数字30修改为300或1200。作用是每个脚本执行的最大时间，默认是30秒，解决可能因为网速和服务器的地址（如国外主机）可能会总是连接超时的问题
# 当经常出现502错误时可以尝试更改此选项。
max_execution_time = 30

# 把前面的分号去掉，并把数字1改为0。0的意思就是关闭重定向执行php文件，出于安全考虑防止别人上传木马执行如：你的网站url/as=你的网站url/sdf/muma.php，这样的重定向PHP文件是可执行的，将这个配置改为0之后这类型的重定向PHP文件就不会执行了
cgi.force_redirect = 1

# 将分号去掉并将数字1改为0。作用是禁止解析非法php文件，如/a.jpg/1.php这样的图片下的一个php文件属于非法的，设置为0就是禁止执行。这种将木马伪装成图片上传的文件存在已久，禁止这类文件运行，即使被上传了木马，由于设置了不允许运行，所以没有用
cgi.fix_pathinfo = 1

#  将前面的分号去掉。作用是iis或nginx使用的是fastcgi方式解析php文件，不开启就不能运行php程序，Apache则不用开启
fastcgi.impersonate = 1

# 在关闭display_errors后开启PHP错误日志（路径在php-fpm.conf中配置））
log_errors = On

```

##### php-fpm 配置

```ini
# 设置错误日志的路径
error_log = /usr/local/php/logs/php-fpm.log

# 引入www.conf文件中的配置（默认已设置）
include=/usr/local/php7/etc/php-fpm.d/*.conf

# pid设置，默认在安装目录中的var/run/php-fpm.pid，建议开启
pid = run/php-fpm.pid

# 错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice
log_level = notice

# 表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值
emergency_restart_threshold = 60
emergency_restart_interval = 60s

# 设置子进程接受主进程复用信号的超时时间. 可用单位: s(秒), m(分), h(小时), 或者 d(天) 默认单位: s(秒). 默认值: 0
process_control_timeout = 0

# 后台执行fpm,默认值为yes，如果为了调试可以改为no。在FPM中，可以使用不同的设置来运行多个进程池。 这些设置可以针对每个进程池单独设置
daemonize = yes

# fpm监听端口，即nginx中php处理的地址，一般默认值即可。可用格式为: 'ip:port', 'port', '/path/to/unix/socket'. 每个进程池都需要设置
listen = 127.0.0.1:9000 

# backlog数，-1表示无限制，由操作系统决定，此行注释掉就行
listen.backlog = -1

# 允许访问FastCGI进程的IP，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。默认值是any。每个地址是用逗号分隔. 如果没有设置或者为空，则允许任何服务器请求连接
listen.allowed_clients = 127.0.0.1

# 启动进程的用户和组
user = www
group = www 

# 对于专用服务器，pm可以设置为static.
# dynamic表示php-fpm进程数是动态的，最开始是pm.start_servers指定的数量，如果请求较多，则会自动增加，保证空闲的进程数不小于pm.min_spare_servers，如果进程数较多，也会进行相应清理，保证多余的进程数不多于pm.max_spare_servers
# static表示php-fpm进程数是静态的, 进程数自始至终都是pm.max_children指定的数量，不再增加或减少
pm = dynamic

# 如何控制子进程，选项有static和dynamic。如果选择static，则由pm.max_children指定固定的子进程数。如果选择dynamic，则由下开参数决定:
# 子进程最大数, 静态方式下开启的php-fpm进程数量
# 如果pm为static, 那么其实只有pm.max_children这个参数生效。系统会开启设置数量的php-fpm进程
# 这个值原则上是越大越好，php-cgi的进程多了就会处理的很快，排队的请求就会很少。
# 一般来说一台服务器正常情况下每一个php-cgi所耗费的内存在20M左右，假设“max_children”设置成100个，20M*100=2000M，
#  也就是说在峰值的时候所有PHP-CGI所耗内存在2000M以内。假设“max_children”设置的较小，比如5-10个，那么php-cgi就会“很累”，处理速度也很慢，等待的时间也较长。
# 如果长时间没有得到处理的请求就会出现504 Gateway Time-out这个错误，而正在处理的很累的那几个php-cgi如果遇到了问题就会出现502 Bad gateway这个错误。
pm.max_children = 300;

# 启动时的进程数, 动态方式下的起始php-fpm进程数量 
# 如果pm为dynamic, 那么pm.max_children参数失效，后面3个参数生效。系统会在php-fpm运行开始的时候启动pm.start_servers个php-fpm进程，然后根据系统的需求动态在pm.min_spare_servers和pm.max_spare_servers之间调整
# pm.start_servers的默认值为2。并且php-fpm中给的计算方式也为：
# {（cpu空闲时等待连接的php的最小子进程数） + （cpu空闲时等待连接的php的最大子进程数 - cpu空闲时等待连接的php的最小子进程数）/ 2}；
# 用配置表示就是：min_spare_servers + (max_spare_servers - min_spare_servers) / 2；
# 一般而言，设置成10-20之间的数据足够满足需求了。
pm.start_servers = 20;

# 保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程, 动态方式下的最小php-fpm进程数量
pm.min_spare_servers = 5;

# 保证空闲进程数最大值，如果空闲进程大于此值，此进行清理, 动态方式下的最大空闲php-fpm进程数量
pm.max_spare_servers = 35;

# 设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0.
# 502，是后端 PHP-FPM 不可用造成的，间歇性的502一般认为是由于 PHP-FPM 进程重启造成的.
pm.max_requests = 1000

# FPM状态页面的网址. 如果没有设置, 则无法访问状态页面. 默认值: none. munin监控会使用到
pm.status_path = /status

# FPM监控页面的ping网址. 如果没有设置, 则无法访问ping页面. 该页面用于外部检测FPM是否存活并且可以响应请求. 请注意必须以斜线开头 (/)
ping.path = /ping

# 用于定义ping请求的返回相应. 返回为 HTTP 200 的 text/plain 格式文本. 默认值: pong.
ping.response = pong

# 设置单个请求的超时中止时间. 该选项可能会对php.ini设置中的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用. 设置为 '0' 表示 'Off'.当经常出现502错误时可以尝试更改此选项
request_terminate_timeout = 0

# 当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'
request_slowlog_timeout = 10s

# 慢请求的记录日志,配合request_slowlog_timeout使用
slowlog = log/$pool.log.slow

# 设置文件打开描述符的rlimit限制. 默认值: 系统定义值默认可打开句柄是1024，可使用 ulimit -n查看，ulimit -n 2048修改
rlimit_files = 1024

# 设置核心rlimit最大限制值. 可用值: 'unlimited' 、0或者正整数. 默认值: 系统定义值.
rlimit_core = 0

# 启动时的Chroot目录. 所定义的目录需要是绝对路径. 如果没有设置, 则chroot不被使用
chroot = 

# 设置启动目录，启动时会自动Chdir到该目录. 所定义的目录需要是绝对路径. 默认值: 当前目录，或者/目录（chroot时）
chdir = 

# 重定向运行过程中的stdout和stderr到主要的错误日志文件中. 如果没有设置, stdout 和 stderr 将会根据FastCGI的规则被重定向到 /dev/null . 默认值: 空
catch_workers_output = yes

# nginx php-fpm配置过程中最大问题是内泄漏出问题：服务器的负载不大，但是内存占用迅速增加，很快吃掉内存接着开始吃交换分区，系统很快挂掉！
# 其实根据官方的介绍，php-cgi不存在内存泄漏，每个请求完成后php-cgi会回收内存，但是不会释放给操作系统，这样就会导致大量内存被php-cgi占用;
# 官方的解决办法是降低PHP_FCGI_MAX_REQUESTS的值，如果用的是php-fpm，对应的php-fpm.conf中的就是max_requests，
# 该值的意思是发送多少个请求后会重启该线程，我们需要适当降低这个值，用以让php-fpm自动的释放内存，不是大部分网上说的51200等等，
# 实际上还有另一个跟它有关联的值max_children，这个是每次php-fpm会建立多少个进程，
# 这样实际上的内存消耗是max_children*max_requests*每个请求使用内存，根据这个我们可以预估一下内存的使用情况，就不用再写脚本去kill了
pm.max_requests = 10240;

# 最大执行时间, 在php.ini中也可以进行配置(max_execution_time)
request_terminate_timeout = 30;

# 开启慢日志
request_slowlog_timeout = 2;

# 慢日志路径
slowlog = log/$pool.log.slow;

# 增加php-fpm打开文件描述符的限制
rlimit_files = 1024;
```

参数调优

### 问题

当一个 PHP-CGI 进程处理的请求数累积到 max_requests 个后，自动重启该进程。

502，是后端 PHP-FPM 不可用造成的，间歇性的502一般认为是由于 PHP-FPM 进程重启造成的.

##### 但是为什么要重启进程呢？

- 如果不定期重启 PHP-CGI 进程，势必造成内存使用量不断增长（比如第三方库有问题等）。因此 PHP-FPM 作为 PHP-CGI 的管理器，提供了这么一项监控功能，对请求达到指定次数的 PHP-CGI 进程进行重启，保证内存使用量不增长。

- 正是因为这个机制，在高并发中，经常导致 502 错误

- 目前我们解决方案是把这个值尽量设置大些，减少 PHP-CGI 重新 SPAWN 的次数，同时也能提高总体性能。PS：刚开始我们是500导致内存飙高，现在改成5120，当然可以再大一些，10240等，这个主要看测试结果，如果没有内存泄漏等问题，可以再大一些。

### 内存|CPU排查方法

##### top

命令格式：

```bash
top [-] [d] [p] [q] [c] [C] [S]    [n] 
# 参数说明
# d： 指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。
# p： 通过指定监控进程ID来仅仅监控某个进程的状态。
# q：该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。
# S： 指定累计模式
# s ： 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。
# i： 使top不显示任何闲置或者僵死进程。
# m：切换显示内存信息。
# t：切换显示进程和CPU状态信息。
# c：切换显示命令名称和完整命令行。
# M： 根据驻留内存大小进行排序。
# P：根据CPU使用百分比大小进行排序。
# T：根据时间/累计时间进行排序。
```

##### sar

执行sar -P ALL 1 100。-P ALL表示监控所有核心，1表示每1秒采集，100表示采集100次。

### 开启慢日志

配置输出php-fpm慢日志，阀值为2秒：

```conf
request_slowlog_timeout = 2
slowlog = log/$pool.log.slow
```

利用sort/uniq命令分析汇总php-fpm慢日志：

```bash
grep -v “^$” www.log.slow.tmp | cut -d ” ” -f 3,2 | sort | uniq -c | sort -k1,1nr | head -n 50
# 参数解释
# sort: 对单词进行排序
# uniq -c: 显示唯一的行，并在每行行首加上本行在文件中出现的次数
# sort -k1,1nr: 按照第一个字段，数值排序，且为逆序
# head -10: 取前10行数据
```

### 用strace跟踪进程

利用nohup将strace转为后台执行，直到attach上的php-fpm进程死掉为止：

```bash
nohup strace -T -p 13167 > 13167-strace.log &
# 参数说明:
# -c 统计每一系统调用的所执行的时间,次数和出错的次数等.
# -d 输出strace关于标准错误的调试信息.
# -f 跟踪由fork调用所产生的子进程.
# -o filename,则所有进程的跟踪结果输出到相应的filename
# -F 尝试跟踪vfork调用.在-f时,vfork不被跟踪.
# -h 输出简要的帮助信息.
# -i 输出系统调用的入口指针.
# -q 禁止输出关于脱离的消息.
# -r 打印出相对时间关于,,每一个系统调用.
# -t 在输出中的每一行前加上时间信息.
# -tt 在输出中的每一行前加上时间信息,微秒级.
# -ttt 微秒级输出,以秒了表示时间.
# -T 显示每一调用所耗的时间.
# -v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
# -V 输出strace的版本信息.
# -x 以十六进制形式输出非标准字符串
# -xx 所有字符串以十六进制形式输出.
# -a column
# 设置返回值的输出位置.默认为40.
# -e execve 只记录 execve 这类系统调用
# -p 主进程号
# 也可以用利用-c参数让strace帮助汇总
# strace -cp pid
```

### 返回502的几种情况？

1. max_children
2. request_terminate_timeout    max_exeecution_time
3. 数据库
4. 网关服务是否启动如php-fpm

### 返回504的情况及解决办法

错误主要查看nginx.conf关于网关如fastcgi的配置

==request_terminate_timeout设置0或者过长问题(502)==

如果设置为0或者过长的时间，可能会引起file_get_contents的资源问题。

- 如果file_get_contents请求的远程资源如果反应过慢，file_get_contents就会一直卡在那里不会超时

- 我们知道php.ini 里面max_execution_time 可以设置 PHP 脚本的最大执行时间，

- 但是，在 php-cgi(php-fpm) 中，该参数不会起效。真正能够控制 PHP 脚本最大执行时间的是 php-fpm.conf 配置文件中的==request_terminate_timeout==参数。

- 修改该参数，设置 PHP 脚本最大执行时间是必要的，但是，治标不治本。例如改成 30s，如果发生 file_get_contents() 获取网页内容较慢的情况，这就意味着 150 个 php-cgi 进程，每秒钟只能处理 5 个请求，WebServer 同样很难避免”502 Bad Gateway”。

- ==解决办法是request_terminate_timeout设置为10s或者一个合理的值，或者给file_get_contents加一个超时参数。==

max_requests参数配置不当，可能会引起间歇性502错误


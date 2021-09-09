### php

### php7新特性

#### 1. 标量类型声明

```php
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));

/**
Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)
*/
```

#### 2. null合并运算符

```php
# null合并运算符 (??) 这个语法糖。如果变量存在且值不为null， 它就会返回自身的值，否则返回它的第二个操作数。
$username = $_GET['user'] ?? 'nobody';
```

#### 3. 太空舱操作符

```php
# 当$a小于、等于或大于$b时它分别返回-1、0或1。
// 整数
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1
```

#### 4. 通过 define() 定义常量数组

```php
<?php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // 输出 "cat"
?>
```

#### 5. 匿名类

通过`new class` 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。

```php
$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});
```

#### 6.  namespace 导入的类、函数和常量现在可以通过单个 use语句 一次性导入了。

```php
use some\namespace\{ClassA, ClassB, ClassC as C};
```

#### 7. 生成器可以返回表达式

```php
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;
// 结果
1
2
3
```

#### 8. 整数除法函数intdiv()

```php
var_dump(intdiv(10, 3)); // int(3)
```

#### 9. 可为空(Nullable)类型

参数以及返回值的类型现在可以通过在类型前加上一个==问号==使之允许为空。 当启用这个特性时，传入的参数或者函数返回的结果要么是给定的类型，要么是 null 。

```php
function testReturn(): ?string
{
    return 'elePHPant';
}

function testReturn(): ?string
{
    return null;
}
```

#### 10. void 函数

回值声明为 void 类型的方法要么干脆省去 return 语句，要么使用一个空的 return 语句。 对于 void 函数来说，**`null`** 不是一个合法的返回值。

```php
function swap(&$left, &$right) : void
{
    if ($left === $right) {
        return;
    }

    $tmp = $left;
    $left = $right;
    $right = $tmp;
}

$a = 1;
$b = 2;
var_dump(swap($a, $b), $a, $b);
// 结果
null
int(2)
int(1)
```

试图去获取一个 void 方法的返回值会得到 **`null`** ，并且不会产生任何警告。这么做的原因是不想影响更高层次的方法。

#### 11. 类常量可见性

```php
class ConstDemo
{
    const PUBLIC_CONST_A = 1;
    public const PUBLIC_CONST_B = 2;
    protected const PROTECTED_CONST = 3;
    private const PRIVATE_CONST = 4;
}
```

#### 12. 多异常捕获处理

一个catch语句块现在可以通过管道字符(`|`)来实现多个异常的捕获。 这对于需要同时处理来自不同类的不同异常时很有用。

```php
try {
    // some code
} catch (FirstException | SecondException $e) {
    // handle first and second exceptions
}
```

#### 13. list()现在支持键名

```php
$data = [
    ["id" => 1, "name" => 'Tom'],
    ["id" => 2, "name" => 'Fred'],
];

// list() style
list("id" => $id1, "name" => $name1) = $data[0];
```

#### 14. 异步信号处理

一个新的名为==pcntl_async_signals()==的方法现在被引入， 用于启用无需 ticks （这会带来很多额外的开销）的异步信号处理。

```php
pcntl_async_signals(true); // turn on async signals

pcntl_signal(SIGHUP,  function($sig) {
    echo "SIGHUP\n";
});

posix_kill(posix_getpid(), SIGHUP); 
// 结果： SIGHUP

```

#### 15. 属性添加限定类型

```php
# 下面的例子中，会强制要求 $user->id 只能为 int 类型，同时 $user->name 只能为 string 类型。
class User {
    public int $id;
    public string $name;
}
```

#### 16. 箭头函数

```php
$factor = 10;
$nums = array_map(fn($n) => $n * $factor, [1, 2, 3, 4]);
// $nums = array(10, 20, 30, 40);
```

#### 17. 空合并运算符赋值

```php
$array['key'] ??= computeDefault();
// 等同于以下旧写法
if (!isset($array['key'])) {
    $array['key'] = computeDefault();
}
```

#### 18. 数组展开操作

```php
$parts = ['apple', 'pear'];
$fruits = ['banana', 'orange', ...$parts, 'watermelon'];
// ['banana', 'orange', 'apple', 'pear', 'watermelon'];
```

#### 19. 数值文字分隔符

```php
6.674_083e-11; // float
299_792_458;   // decimal
0xCAFE_F00D;   // hexadecimal
0b0101_1111;   // binary
```

#### 20. 允许从 __toString() 抛出异常

现在允许从 ==__toString()==抛出异常。之前的版本，将会导致一个致命错误。新版本中，之前发生致命错误的代码，已经被转换为 Error 异常。

### php8 新特性

#### 1. 命名参数

==命名参数允许根据参数名而不是参数位置向函数传参==。这使得参数的含义自成体系，参数与顺序无关，并允许任意跳过默认值。

命名参数通过在参数名前加上冒号来传递。允许使用保留关键字作为参数名。参数名必须是一个标识符，不允许动态指定。

```php
myFunction(paramName: $value);
array_foobar(array: $value);

// NOT supported.
function_name($variableStoringParamName: $value);

// 使用顺序传递参数：
array_fill(0, 100, 50);

// 使用命名参数：
array_fill(start_index: 0, count: 100, value: 50);

array_fill(value: 50, count: 100, start_index: 0);
```

#### 2. 构造器属性提升

构造器的参数也可以相应提升为类的属性。

```php
class Point {
    protected int $x;
    protected int $y;

    public function __construct(int $x, int $y = 0) {
        $this->x = $x;
        $this->y = $y;
    }
}

// 等同于上面的例子
class Point {
    public function __construct(protected int $x, protected int $y = 0) {
    }
}
```

#### 3. 联合类型

联合类型接受多个不同的类型做为参数。声明联合类型的语法为 `T1|T2|...`。

#### 4. match表达式

```php
# match, 这个关键字的作用跟switch有点类似。
# switch
switch ($input) {
    case "true":
        $result = 1;
    break;
    case "false":
        $result = 0;
    break;
    case "null":
        $result = NULL;
    break;
}

# match 等同于上面
$result = match($input) {
        "true" => 1,
        "false" => 0,
        "null" => NULL,
};

# 并且，类似switch的多个case一个block一样，match的多个条件也可以写在一起，比如:
$result = match($input) {
    "true", "on" => 1,
    "false", "off" => 0,
    "null", "empty", "NaN" => NULL,
};

# 需要注意的和switch不太一样的是，以前我们用switch可能会经常遇到这种诡异的问题:
$input = "2 person";
switch ($input) {
    case 2:
        echo "bad";
    break;
}
# 你会发现，bad竟然被输出了，这是因为switch使用了宽松比较(==)。match就不会有这个问题了, 它使用的是严格比较(===)，就是值和类型都要完全相等。

# 还有就是，当input并不能被match中的所有条件满足的时候，match会抛出一个UnhandledMatchError exception:
$input = "false";
$result = match($input) {
        "true" => 1,
};
// 结果
Fatal error: Uncaught UnhandledMatchError: Unhandled match value of type string
```

#### 5. Nullsafe方法和属性

属性和方法可以通过 "nullsafe" 操作符访问：==?->==

此操作的结果，类似于在每次访问前使用 is_null() 函数判断方法和属性是否存在，但更加简洁。

```php
// 自 PHP 8.0.0 起可用
$result = $repository?->getUser(5)?->name;

// 上边那行代码等价于以下代码
if (is_null($repository)) {
    $result = null;
} else {
    $user = $repository->getUser(5);
    if (is_null($user)) {
        $result = null;
    } else {
        $result = $user->name;
    }
}
```



### 问题

#### 1. php array 底层原理？

array底层基于散列表实现

#### 2. 获取HTTP头文件？

```php
// 获取全部（客户端）HTTP请求头信息
   #1 array apache_request_headers(void)
   #2：通过$_SERVER获取，每个http请求头信息都以"HTTP_"开头，
//	在$_SERVER键中获取if_modified_since的请求信息
	$_SERVER['HTTP_IF_MODIFIED_SINCE']
   # url 请求的服务器的URL地址 
   # format 0:返回的头部信息以索引数字形式，1:返回头部信息以关联数组形式
   $head_arr = get_headers("https://www.baidu.com");　　
   $head_arr_index = get_headers("https://www.baidu.com",1);
```

#### 3. cookie会话攻击防护?

什么样的Cookie信息可以被攻击者利用
    1. Cookie中包含了不应该让除开发者之外的其他人看到的其他信息，如USERID=1000
       USERSTATUS=ONLINE，ACCOUNT_ID=xxx等等这些信息。
       2. Cookie信息进行了加密，但是很容易被攻击者进行解密
       3. 在对Cookie信息的时候没有进行输入验证
       如何防范利用Cookie进行的攻击
       1. 不要在Cookie中保存敏感信息
       2. 不要在Cookie中保存没有经过加密的或者容易被解密的敏感信息
       3. 对从客户端取得的Cookie信息进行严格校验
       4. 记录非法的Cookie信息进行分析，并根据这些信息对系统进行改进。
       5. 使用SSL/TLS来传递Cookie信息
       cookie和session的区别：
       1）cookie数据存放在客户的浏览器上，session数据放在服务器上。
       2）cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗考虑到安全应当使用session
       3）session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能考虑到减轻服务器性能方面，应当使用COOKIE。
       4）单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie

#### 4. PHP扩展文件安装过程？

```bash
# phpize安装
#下载libevent扩展文件压缩包（在当前系统哪个目录下载随意）
	~# wget http://pecl.php.net/get/libevent-0.1.0.tgz
	# 解压文件
	~# tar -zxvf libevent-0.1.0.tgz
	# 进入源码目录
	~# cd libevent-0.1.0/
	# 如 /usr/local/php7/bin/phpize //运行phpize命令，写全phpize的路径
	~# ./configure --with-php-config=/usr/local/php/bin/php-config
	# 运行configure命令，配置时 要将php-config的路径附上
	~# make
	~# make test
	~# sudo make install
	# 修改php.ini，结尾加入：extension=libevent.so
	# 重启对应的php-fpm
```

#### 5. 一个客户端http请求从服务器server到nginx到php响应返回整个流程？

HTTP 事务执行过程 
	客户端（浏览器）做出请求操作（输入网址、点击链接、提交表单）。
	客户端对域名进行解析，向设定的 DNS 服务器请求 IP 地址。
	客户端根据 DNS 服务器返回 IP 地址采用三次握手与服务端建立 TCP/IP 连接。
	TCP/IP 连接成功后，客户端向服务端发送 HTTP 请求。
	服务端的 Web Server 会判断 HTTP 请求的资源类型，进行内容分发处理；如果请求的资源为 PHP 文		件，服务端软件会启动对应的 CGI 程序进行处理，并返回处理结果。
	服务端将 Web Server 的处理结果响应给客户端
	客户端接收服务端的响应，并渲染处理结果，如果响应内容需要请求其他静态资源，通过 CDN 加速访问所需资源。
	客户端将渲染好的视图呈现出来并断开 TCP/IP 连接

#### 6. CGI、FastCGI、PHP-CGI和PHP-FPM原理区别？

CGI：是公共网关接口 Web Server 与 Web Application 之间数据交换的一种协议。
FastCGI：FastCGI就像是一个常驻（long-live）型的CGI程序，它可以一直运行着。同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似。
PHP-CGI：是 PHP （Web Application）对 Web Server 提供的 CGI 协议的接口程序。
PHP-FPM：是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，额外还提供了相对智能一些任务管理。

#### 7. 爬虫模拟登陆，如何跳过验证码？

1. 爬取网站时经常会遇到需要登录的问题，这是就需要用到模拟登录的相关方法。python提供了强大的url库，想做到这个并不难。

2. 首先得明白cookie的作用，cookie是某些网站为了辨别用户身份、进行session跟踪而储存在用户本地终端上的数据。因此我们需要用Cookielib模块来保持网站的cookie。
3. 这个是要登陆的地址 1 和验证码地址 2
4. 可以发现这个验证码是动态更新的每次打开都不一样，一般这种验证码和cookie是同步的。其次想识别验证码肯定是吃力不讨好的事，因此我们的思路是首先访问验证码页面，保存验证码、获取cookie用于登录，然后再直接向登录地址post数据
5. 首先通过抓包工具或者火狐或者谷歌浏览器分析登录页面需要post的request和header信息。模拟登录
        验证码地址和post地址
       将cookies绑定自动管理
       使用用户名和密码
       用代码访问验证码地址,获取cookie
       保存验证码到本地
       打开保存的验证码图片输入
       根据抓包信息 构造表单
       根据抓包信息 构造headers
       生成post数据 ?key1=value1&key2=value2的形式
       构造request请求
       打印登录后的页面
       登录成功后便可以利用该cookie访问其他需要登录才能访问的页面。

#### 8. $a=[0,1,2,3]; $b=[1,2,3,4,5]; $a+=$b; echo json_encode($a) ？

答：[0,1,2,3,5] // array＋array合并数组则会把最先出现的值作为最终结果返回

#### 9. 以下代码执行结果是？

```php
$count = 5;
   	function get_count(){
		static $count = 0;
		return $count++;
	}
	++$count;get_count();echo get_count();
//  答：1  static静态类型.这种的值永远都是静态的,第一次调用声明等于0，并且自增等于1。第二次调用，1再自增就等于2


$a=[1,2,3]; 
	foreach($a as &$v){} 
	foreach($a as $v){} 
	echo json_encode($a);
# 答：[1,2,2] // 在 PHP 中，foreach 结束后，循环中的索引值（index）及內容（value）並不会被重置。
	    // 所以最后的 $v还指向最后一个元素，再次循环，就会把最后个元素的值修改掉了。
	    // 解决的办法是，循环完毕之后，用unset($v);
```

#### 10. php执行过程的顺序正确的是？

 扫描->解析->编译->执行->输出
 PHP简化执行过程： 
	1.扫描(scanning) ,将index.php内容变成一个个语言片段(token) 
	2.解析(parsing) , 将一个个语言片段变成有意义的表达式 
	3.编译(complication),将表达式编译成中间码(opcode) 
	4.执行(execution),将中间码一条一条的执行 
	5.输出(output buffer),将要输出的内容输出到缓冲区

#### 11. HTTP中GET与POST的区别有哪些？

GET在浏览器回退时是无害的，而POST会再次提交请求。

 GET产生的URL地址可以被Bookmark(书签)，而POST不可以。

GET请求会被浏览器主动cache，而POST不会，除非手动设置。

 GET请求只能进行url编码，而POST支持多种编码方式。

 GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。

  GET请求在URL中传送的参数是有长度限制的，而POST没有。

 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。

 GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。

 GET参数通过URL传递，POST放在Request body中。

#### 12. 为什么大型网站要使用消息队列？

消息队列常见的使用场景有很多，但是比较核心的有 3 个：解耦、异步、削峰 

#### 13. Memcache和redis区别？

Memcache
  该产品本身特别是数据在内存里边的存储，如果服务器突然断电，则全部数据就会丢失
  单个key（变量）存放的数据有1M的限制
  存储数据的类型都是String字符串类型
  本身没有持久化功能
  可以使用多核（多线程）
 Redis
  数据类型比较丰富:String、List、Set、Sortedset、Hash
  有持久化功能，可以把数据随时存储在磁盘上
  本身有一定的计算功能
  单个key（变量）存放的数据有1GB的限制

#### 14. 如何设计一个高并发系统？

   1.系统拆分 将一个系统拆分为多个子系统，用 dubbo 来搞。然后每个系统连一个数据库，这样本来就一个库，现在多个数据库，不也可以扛高并发么。
   2.缓存 缓存，必须得用缓存。大部分的高并发场景，都是读多写少，那你完全可以在数据库和缓存里都写一份，然后读的时候大量走缓存不就得了。毕竟人家 redis 轻轻松松单机几万的并发。所以你可以考虑考虑你的项目里，那些承载主要请求的读场景，怎么用缓存来抗高并发。
   3.MQ MQ，必须得用 MQ。可能你还是会出现高并发写的场景，比如说一个业务操作里要频繁搞数据库几十次，增删改增删改，疯了。那高并发绝对搞挂你的系统，你要是用 redis 来承载写那肯定不行，人家是缓存，数据随时就被 LRU 了，数据格式还无比简单，没有事务支持。所以该用
   4.mysql 还得用 mysql 啊。那你咋办？用 MQ 吧，大量的写请求灌入 MQ 里，排队慢慢玩儿，后边系统消费后慢慢写，控制在 mysql 承载范围之内。所以你得考虑考虑你的项目里，那些承载复杂写业务逻辑的场景里，如何用 MQ 来异步写，提升并发性。MQ 单机抗几万并发也是 ok 的，这个之前还特意说过。
   5.分库分表 分库分表，可能到了最后数据库层面还是免不了抗高并发的要求，好吧，那么就将一个数据库拆分为多个库，多个库来扛更高的并发；然后将一个表拆分为多个表，每个表的数据量保持少一点，提高 sql 跑的性能。
   6.读写分离 读写分离，这个就是说大部分时候数据库可能也是读多写少，没必要所有请求都集中在一个库上吧，可以搞个主从架构，主库写入，从库读取，搞一个读写分离。读流量太多的时候，还可以加更多的从库。
   7.ElasticSearch Elasticsearch，简称 es。es 是分布式的，可以随便扩容，分布式天然就可以支撑高并发，因为动不动就可以扩容加机器来扛更高的并发。那么一些比较简单的查询、统计类的操作，可以考虑用 es 来承载，还有一些全文搜索类的操作，也可以考虑用 es 来承载。

#### 15. PHP-FPM 的运行方式？

static(静态) ：表示在 php-fpm 运行时直接 fork 出 pm.max_children 个子进程，
   dynamic(动态)：表示，运行时 fork 出 start_servers 个进程，随着负载的情况，动态的调整，最多不超过 max_children 个进程。
   一般推荐用 static ，优点是不用动态的判断负载情况，提升性能；缺点是多占用些系统内存资源。

#### 16. PHP-FPM 子进程数量，是不是越多越好？

当然不是，pm.max_chindren，进程多了，增加进程管理的开销以及上下文切换的开销。更核心的是，能并发执行的 php-fpm 进程不会超过 cpu 个数。如何设置，取决于你的代码。如果代码是 CPU 计算密集型的，pm.max_chindren 不能超过 CPU 的内核数。如果不是，那么将 pm.max_chindren 的值大于 CPU 的内核数，是非常明智的。国外技术大拿给出适用于 dynamic 方式的公式： 在 N + 20% 和 M / m 之间。
   .N 是 CPU 内核数量。
   .M 是 PHP 能利用的内存数量。
   .m 是每个 PHP 进程平均使用的内存数量。
   *static方式的公式：M / (m 1.2)**
   当然，还有一种保险的方式，来配置 max_children。 先把 max_children 设置成一个比较大的值。稳定运行一段时间后，观察 php-fpm 的 status 里的 max active processes 是多少，然后把 max_children 配置比它大一些就可以了。
   pm.max_requests：指的是每个子进程在处理了多少个请求数量之后就重启。这个参数，理论上可以随便设置，但是为了预防内存泄漏的风险，还是设置一个合理的数比较好。

#### 17. JSON Web Token令牌(JWT)的原理?

   1.jwt原理：服务器认证以后，生成一个JSON对象，发回给用户，用户与服务器通信的时候，都要发回这个JSON对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。
   2.jwt与session的区别？
	1)session存储在服务端占用服务器资源，而JWT存储在客户端
	2）session存储在Cookie中，存在伪造跨站请求伪造攻击的风险
	3）session只存在一台服务器上，那么下次请求就必须请求这台服务器，不利于分布式应用
	4）存储在客户端的JWT比存储在服务器的session更具有扩展性
   3.JWT流程说明？
	1，浏览器发起请求登陆，携带用户名和密码；
	2，服务端验证身份，根据算法，将用户标识符打包生成 token,
	3，服务器返回JWT信息给浏览器，JWT不包含敏感信息；
	4，浏览器发起请求获取用户资料，把刚刚拿到的 token一起发送给服务器；
	5，服务器发现数据中有 token，验明正身；
	6，服务器返回该用户的用户资料；
   4.JWT的优缺点？
	1、JWT默认不加密，但可以加密。生成原始令牌后，可以使用改令牌再次对其进行加密。
	2、当JWT未加密方法时，一些私密数据无法通过JWT传输。
	3、JWT不仅可用于认证，还可用于信息交换。善用JWT有助于减少服务器请求数据库的次数。
	4、JWT的最大缺点是服务器不保存会话状态，所以在使用期间不可能取消令牌或更改令牌的权限。也就是说，一旦JWT签发，在有效期内将会一直有效。
	5、JWT本身包含认证信息，因此一旦信息泄露，任何人都可以获得令牌的所有权限。为了减少盗用，JWT的有效期不宜设置太长。对于某些重要操作，用户在使用时应该每次都进行进行身份验证。
	6、为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，而是使用加密的HTTPS协议进行传输。
   5.JWT的数据结构？
	1.JWT的消息构成？
	   一个token分3部分，按顺序：头部(header)、载荷(payload)、签证(signature)
	   对象为一个很长的字符串，字符之间用“.”分隔符分为三个子串
	   1.JWT的头部承载两部分信息：1.声明类型，这里是jwt,2.声明加密算法，通常为SHA256
	   2.JWT载荷部分也是一个JSON对象，可增加自定义信息，官方定义了7个字段：
		iss (issuer)：签发人
		exp (expiration time)：过期时间
		sub (subject)：主题
		aud (audience)：受众
		nbf (Not Before)：生效时间
		iat (Issued At)：签发时间
		jti (JWT ID)：编号
	   3.签名部分是对前两部分的签名，防止数据篡改
		首先，需要指定一个密匙（secret），这个密匙只有服务器才知道，不能泄露给用户，然后使用
		header里面签名算法，按照下面的公式产生签名：
	HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
   	// Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。
    6.JWT的用法？
	客户端接收服务器返回的JWT，将其存储在Cookie或localStorage中，此后，客户端将在于服务器交互中都会带JWT。如果将它存储在Cookie中，就可以自动发送，但是不会跨域，因此一般是将它放入HTTP请求的header Authorization字段中。
	Authorization: Bearer <token>
	当跨域时，也可以将JWT被放置于POST请求的数据主体中。

#### 18. php判断是爬虫在访问还是用户浏览器在访问?

主要就是判断$_SERVER['HTTP_USER_AGENT'];里面的内容有没有爬虫的标志

#### 19. $_SERVER参数说明？

```php
	$_SERVER["SCRIPT_NAME"] => "/index.php"，// 当前脚本路径
	$_SERVER["REQUEST_URI"] => "/index.php?id=1"，// 访问的页面URI，包含查询字符串
	$_SERVER["QUERY_STRING"] => "id=1"，// 查询字符串，不存在为" "
	$_SERVER["REQUEST_METHOD"] => "GET"，// 请求方法，如"POST"、"PUT"等
	$_SERVER["SERVER_PROTOCOL"] => "HTTP/1.1"，// 通信协议的名称和版本
	$_SERVER["GATEWAY_INTERFACE"] => "CGI/1.1"，// 服务器使用的CGI 规范的版本
	$_SERVER["REMOTE_PORT"] => "60599"，// 用户连接服务器使用的端口
	$_SERVER["SCRIPT_FILENAME"] => "E:/WWW/example/index.php"，// 当前脚本的绝对路径
	$_SERVER["DOCUMENT_ROOT"] => "E:/WWW/example/"，// 当前脚本文档根目录的绝对路径
	$_SERVER["REMOTE_ADDR"] => "127.0.0.1"，// 用户的IP地址
	$_SERVER["SERVER_PORT"] => "80"，// 服务器使用的端口
	$_SERVER["SERVER_ADDR"] => "127.0.0.1"，// 服务器的IP地址
	$_SERVER["SERVER_NAME"] => "www.example.com"，// 服务器的主机名
	$_SERVER["SERVER_SOFTWARE"] => "Apache/2.4.23 (Win32) OpenSSL/1.0.2j mod_fcgid/2.3.9"，// 响应头中Server的内容
	$_SERVER["SERVER_SIGNATURE"] => ""，// 包含了服务器版本和虚拟主机名的字符串
	$_SERVER["HTTP_HOST"] => "www.example.com"，// 请求头中Host项的内容
	$_SERVER["HTTP_CONNECTION"] => "keep-alive"，// 请求头中Connection项的内容
	$_SERVER["HTTP_PRAGMA"] => "no-cache"，// 请求头中Pragma项的内容
	$_SERVER["HTTP_CACHE_CONTROL"] => "no-cache"，// 请求头中Cache-Control项的内容
	$_SERVER["HTTP_UPGRADE_INSECURE_REQUESTS"] => "1"，// 请求头中Upgrade-Insecure-Requests项的内容
	$_SERVER["HTTP_USER_AGENT"] => "Mozilla/5.0 (Windows NT 10.0; Win64; x64) 	AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36"，// 请求头中User-Agent项的内容
	$_SERVER["HTTP_ACCEPT"] => "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,/;q=0.8"，// 请求头中Accept项的内容
	$_SERVER["HTTP_ACCEPT_ENCODING"] => "gzip, deflate"，// 请求头中Accept-Encoding项的内容
$	_SERVER["HTTP_ACCEPT_LANGUAGE"] => "zh-CN,zh;q=0.8"，// 请求头中Accept-Language项的内容
	$_SERVER["PHP_SELF"] => "/index.php"，// 当前执行脚本的文件名
	$_SERVER["REQUEST_TIME_FLOAT"] => 1510112348.8084，// 请求开始的时间戳，微秒级别精准度
	$_SERVER["REQUEST_TIME"] => 1510112348，// 请求开始的时间戳
```

#### 20. PHP上传文件时 $_FILES 为空，可能的原因及解决方法?

出现这个问题的原因主要有两个：表单原因或者php设置原因：
	1) 上传文件的表单编码类型必须设置成 enctype="multipart/form-data"
	2) php配置问题
	   (1) 在 php.ini 里查找 max_execution_time 默认脚本最大执行30秒，这里可以改为 max_execution_time = 0 ; 表示没有时间限制，或在php文件头部添加：ini_set('max_execution_time',0);
	   (2) 修改 post_max_size 设定 POST 数据所允许的最大大小。此设定也影响到文件上传， 在php.ini查找 post_max_size 改为 post_max_size = 50M
	   (3) 很多人都会改了第二步.但上传文件时最大仍然为 8M. 这是为什么呢.我们还要改一个参数
	   upload_max_filesize 表示所上传的文件的最大大小。 在php.ini查找 upload_max_filesize ,默认为8M改为upload_max_filesize = 20M ，另外要说明的是，post_max_size 大于 upload_max_filesize 为佳。

#### 21. 单链表数据反转？

例如：1—>2—>3—>4 反转成：4—>3—>2—>1
    方式一：遍历节点，反转每个节点，也叫头插法，因为节点依次插入了新链表的头部
	因为单链表只有指向下一个节点的指针，没有指向上个节点的指针。所以我们可以定义个指针指向上个节点，这样我们遍历链表，把每个指向下个节点的指针，指向上个节点，这样每个节点都指向了上个节点，实现了反转
    方式二：借助栈的特性，先进后出，实现单链表的反转


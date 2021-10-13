### PHP设计模式

##### 单例模式

确保某==一个类只有一个实例==，而且自行实例化并向整个系统提供这个实例。通俗理解(单例模式一般也就是强调类的静态调用,一个进程对一个类的多次调用只产生一个类对象)

###### **单例模式有那些好处呢？**

PHP应用主要用于数据库应用，因此会存在大量的数据库操作，使用单例模式可以避免大量的 new 操作消耗的资源。

实例：

```php
  /*
 * PHP生成对象之设计模式—单例模式
 * 用于保存基本信息的单例类 存储URL目录、文件路径等数据
 * 1、Preferences对象可以被系统中中任何对象使用
 * 2、Preferences对象不被存储在会被覆写的全局变量中
 */
class Preferences {
	private $props = [];
  // 保存实例在此属性中
  private static $instance;
  // 构造函数声明为private，防止直接创建对象
  private function __construct() {}
  // 单例方法
  public static function getInstance() {
    if (empty(self::$instance)) {
      self::$instance = new Preferences();
    }
    return self::$instance;
  }
  
  // 设置属性和属性值
  public function setProperty($key, $val) {
    $this->props[$key] = $val;
  }
  
  // 读取值
  public function getProperty($key) {
    return $this->props[$key];
  }
}

//得到Preferences类的单例对象
$pref=Preferences::getInstance();

//设置一个属性 name 的值 为matt
$pref->setProperty("name","matt");

//移除引用
unset($pref);

//得到Preferences类的单例对象
$pref2=Preferences::getInstance();

//输出属性name 的属性值 该属性值未丢失 
print $pref2->getProperty("name");
```

##### 工厂模式

###### 什么是工厂方法模式？

​	动态的根据传递的数据，新建相应的类的对象

###### 在什么情况下使用工厂模式？

​	只有运行时才知道需要构造那种类型的对象。

​	可以轻松添加一种新类型

​	每种类型都需要不同的方法

###### 优缺点

优点：工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品。

缺点：客户可能仅仅为了创建一个特定的ConcreteProduct对象，就不得不创建一个Creator子类

实例：

```php
// 抽象工厂类
abstract class AbstractUser {
  abstract function getUsername();
  abstract function getUserID();
  abstract function getUserIntegral();
}

// 工厂实现
class UserInfo extends AbstractUser {
  // 姓名
  function getUserName() {
    return "test_name";
  }
  
  // ID 
  function getUserID() {
    return 1;
  }
  
  // 积分
  function getUserIntegral() {
    return 10;
  }
}

echo UserInfo::getUserName()."<br>";
echo UserInfo::getUserId()."<br>";
echo UserInfo::getUserIntegral();
```

##### 适配器模式

==适配器设计模式只是将某个对象的接口适配为另外一个对象所期望的接口==

实例：

```php
// 早先设计的类
class UserInfo {
  /**
 	 * 根据用户UID获取用户信息
 	 * @param inti $uid 用户ID 
 	 * @return array $userinfo 返回用户信息
 	 */
  public function getUserInfo($uid) {
    // 相关处理
    // DB层，从数据库查询用户信息
    $userinfo = [
      'uid' 			=> 1,
      'username' 	=> 'test_name'
    ];
    return $userinfo;
  }
}

// 适配器类，目的是在新需求增加的情况，不修改以前公共接口类，通过对适配器userInfoIntegral用户积分类的扩展
class UserInfoIntegral extends UserInfo {
  public function getUserIntegral($uid) {
    $integral = [
      'integral1'		=> 2,
      'integral2'		=> 3
    ];
    return $integral;
  }
  
  public function getUser($uid) {
    $userall = [
      'userinfo' 			=> $this->getUserInfo($uid),
      'userintegral'	=> $this->getUserIntegral($uid)
    ];
    return $userall;
  }
}

// 获取用户信息的客户端类
class myObject {
  public function write($uid) {
    $userInfoApp = new UserInfoIntegral();
    return $userInfoApp->getUser($uid);
  }
}

$l = new myObject();
print_r($l->write(1));
```

##### 建造者模式

建造者设计模式定义了处理其他对象的复杂构建的对象设计。

用一个简单的mysql、mongo链接类说明：

```php
/**
 * 数据库链接类 - 此类只是一个简单的说明实例，如需使用，请加以简单完善修改后在使用
 */
class connect {
  private $localhost;//主机 sql服务器
 	private $dbuser;//数据库用户名
 	private $password;//数据库密码
 	private $database;//数据库名
 	private $charset;//编码
 	private $pconnect;//是否持久链接
 	
 	private $port; //数据库端口
 	private $mongo;//mongo 对象
 	private $db;//db mongodb对象数据库
  
  public function __construct($config){
		$this->localhost=$config['localhost'];
 		$this->dbuser=$config['dbuser'];
 		$this->password=$config['password'];
 		$this->database=$config['database'];
 		$config['charset']?$this->charset=$config['charset']:'utf8';//默认为utf8字符集
 		$config['pconnect']?$this->pconnect=$config['pconnect']:0;//默认为utf8字符集
 		$config['port']?$this->port=$config['port']:3360;//端口
 		
 		//MongoDB
 		if (!$config['option']){
      $config['option'] = ['connect' => true];
    } 
	}
  
  /**
	 * MYSQL连接
	 * @return obj
	 */
 	public function db_connect(){
 		$conn = ($config['pconnect'] == 0) ? mysql_connect($this->localhost, $this->dbuser, $this->password, true) 
      																 : mysql_pconnect($this->localhost, $this->dbuser, $this->password);
	    if(!$conn){  
	       die('Could not connect: ' . mysql_error());  
	    }
	    mysql_query('SET NAMES ' . $this->database, $conn);  //设置数据库字符集
	    $dbname=@mysql_select_db($this->database, $conn);  
	    if(!$dbname){  
	        die ("Can\'t use test_db : " . mysql_error());  
	    }  
	    return $conn;  
 	}
  
  /**
 	 * NoSql mongoDB链接
 	 */
 	public function mongodb(){	
 		$server = 'mongodb://' . $this->localhost . ':' . $this->port;
 		//链接mongodb 数据库
 		$this->mongo = new Mongo($server, $this->options);
 		//选择数据库
		$this->db = $this->mongo->selectDB($this->database);
		//用户名 密码
		$this->db->authenticate($this->dbuser, $this->password);
		//$mongo->connect();  //调用connect方法，来保持连接。  
		//$mongo->close();  //调用close方法，关闭数据库连接。  
		//$alldb= $this->mongo->listDBs(); //调用listDBs方法，返回$alldb这个多维数组。显示所有的数据库。 
 	}
}

/**
  * 建造者模式类  目的是处理数据库链接对象的复杂构建的对象设计
  * 这只是一个简单说明类，如需分装一个完整的正式环境下用的数据库多种类型链接类，请稍微加以修改即可用。
  * 当然此类你也可以直接拿去用，只是感觉不完美。
  */
class mySqlDb {
  protected $obj;
  protected $sqltype;
  public function __construct($config) {
    $this->obj = new connect($config);
    $htis->sqltype = $config['sqltype'];
  }
  
  public function buildDb() {
    if ($this->sqltype == 'mysql') {
      $this->obj->db_connect();
    } else {
      $this->obj->mongodb();
    }
  }
}

/* 创建过程被封装了，方便切换使用mysql 或 mongo数据库链接 
 */  
$config = array( 
	'sqltype'=>'mysql', 
    'localhost' => '127.0.0.1',  
    'dbuser' => 'root',  
    'password' => '123456' ,
    'database'=>'weike',
    );  
 $sql=new MysqlDb($config);
 $sql->buildDb();
 
 // 用sql查询测试 
 $sql='select * from pre_brand';
 $result =mysql_query($sql);
 $row=mysql_fetch_array($result);
  print_r($row);
  mysql_free_result($result);//释放与之关联的资源 （脚本执行完毕后会自动释放内存）
```

##### 策略模式

策略模式，==将一组特定的行为和算法封装成类，以适应某些特定的上下文环境==
使用场景：电商网站根据用户类型，推荐展示不同的类目、商品信息

```php
// 用户抽象接口
interface UserStrategy
{
    function showCate(); // 美食
    function showInfo(); // 信息
}

// 男性 
class MaleUser implements UserStrategy
{
    public function showCate()
    {
        echo "show male cate!!!\r\n";
    }

    public function showInfo()
    {
        echo "show male info!!!\r\n";
    }
}

// 女性
class FemalUser implements UserStrategy
{
    public function showCate()
    {
        echo "show female cate!!!\r\n";
    }

    public function showInfo()
    {
        echo "show female info!!!\r\n";
    }
}

// 返回页信息
class Page
{
    public $strategy;

    public function setStrategy($obj)
    {
        $this->strategy = $obj;
    }

    public function show()
    {
        $this->strategy->showCate();
        $this->strategy->showInfo();
    }
}

// 根据不同的登录用户返回不同的页信息
$page = new Page();

$type = 'male';
if ($type == 'male') {
    $strategy = new MaleUser();
} else if ($type == 'femal') {
    $strategy = new FemalUser();
}
$page->setStrategy($strategy);
$page->show();
```

##### 观察者模式

描述：==当一个对象状态发生变化时，依赖它的对象全部收到通知，并自动更新==
 场景：一个事件发生后，要执行一连串更新操作，传统的编程方式，就是在时间得代码之后直接加入处理的逻辑，当更新的逻辑增多之后，代码会变的难以维护，这种方式是耦合的，侵入式的，增加新的逻辑需要修改事件的主体代码，观察者模式实现了低耦合，非侵入式的通知和更新机制。

```php
**
 * @desc 观察者抽象接口
 * Class Observer
 */
interface Observer
{
    function update();
}

// 事件类
class Event
{
    private $observers = [];
		
  	// 添加观察者
    public function addObserver(Observer $observer)
    {
        $this->observers[] = $observer;
    }
		// 通知接口
    public function notify()
    {
        foreach ($this->observers as $observer) {
            $observer->update();
        }
    }
		
  	// 触发事件
    public function trigger()
    {
        echo "Event trigger!!!\r\n";
    }
}

// 观察者1
class Observer1 implements Observer
{
    public function update()
    {
        echo "observer1 update\r\n";
    }
}

// 观察者2
class Observer2 implements Observer
{
    public function update()
    {
        echo "observer2 update\r\n";
    }
}

// 将观察者注入到事件类中，触发事件后，逐个通知
$event = new Event();
$event->addObserver(new Observer1());
$event->addObserver(new Observer2());
$event->trigger();
$event->notify();
```



##### 控制反转

==层层的依赖关系，反转到调用的起点。通过调整注入的对象，来控制程序的行为==

```php
//传统依赖写法
class A {
    public function action()
    {
        echo "class A\r\n";
    }
}

class B {
    public function action()
    {
        $a = new A();
        $a->action();
    }
}

class C {
    public function action()
    {
        $b = new B();
        $b->action();
    }
}

//传统依赖写法,类耦合在一起，层层调用
$c = new C();
$c->action();

//控制反转模式
class NewA {

    public function action()
    {
        echo "class newA\r\n";
    }

}

class NewB {

    public $obj;

    public function __construct(NewA $a)
    {
        $this->obj = $a;
    }

    public function action() {
        echo "class newB\r\n";
        $this->obj->action();
    }
}

class NewC {

    public $obj;

    public function __construct(NewB $b)
    {
        $this->obj = $b;
    }

    public function action()
    {
        echo "class newC\r\n";
        $this->obj->action();
    }
}

$obj = new NewC(new NewB(new NewA()));
$obj->action();
```



##### 装饰器

描述：装饰器模式，==可以动态地添加修改类的功能==

```php
/**
 * 装饰器接口
 * Interface Decorator
 */
interface Decorator
{
    public function before();

    public function after();
}

// 颜色类 实现 装饰器接口
class Color implements Decorator
{
    public function before()
    {
        echo "Color before\r\n";
    }

    public function after()
    {
        echo "Color after\r\n";
    }
}

// 尺寸类 实现 装饰类接口
class Size implements Decorator
{
    public function before()
    {
        echo "Size before\r\n";
    }

    public function after()
    {
        echo "Size after\r\n";
    }
}

// 展示类
class Show
{
    public $decorator = [];

  	// 添加装饰类
    public function addDecorator(Decorator $decorator)
    {
        $this->decorator[] = $decorator;
    }

  	// 装饰之前的样子
    public function before()
    {
        foreach ($this->decorator as $item) {
            $item->before();
        }
    }

  	// 装饰之后的样子
    public function after()
    {
        foreach ($this->decorator as $item) {
            $item->after();
        }
    }
		
  	// 展现
    public function index()
    {
        //before
        $this->before();

        //核心处理
        echo "Show index\r\n";

        //after
        $this->after();
    }
}

// 使用
$show = new Show();
$show->addDecorator(new Color());
$show->addDecorator(new Size());
$show->index();
```

##### 注册模式

==通过将对象实例注册到一棵全局的对象树上，需要的时候从对象树上采摘的模式设计方法==

下面让三种模式做个小小的结合。单纯创建一个实例对象远远没有这么复杂，但运用于大型项目的话，便利性便不言而喻了。

```php
//创建单例
class Single{
    public $hash;
    static protected $ins=null;
    final protected function __construct(){
        $this->hash=rand(1,9999);
    }

    static public function getInstance(){
        if (self::$ins instanceof self) {
            return self::$ins;
        }
        self::$ins=new self();
        return self::$ins;
    } 
}

//工厂模式
class RandFactory{
    public static function factory(){
        return Single::getInstance();
    }
}

//注册树
class Register{
    protected static $objects;
    public static function set($alias,$object){
        self::$objects[$alias]=$object;
    }
    public static function get($alias){
        return self::$objects[$alias];
    }
    public static function _unset($alias){
        unset(self::$objects[$alias]);
    }
}

Register::set('rand',RandFactory::factory());

$object=Register::get('rand');

print_r($object);
```

##### 依赖注入模式

**依赖注入（Dependency Injection）**是**控制反转（Inversion of Control）**的一种实现方式。

要实现控制反转，通常的解决方案是将创建被调用者实例的工作交由 IoC 容器来完成，然后在调用者中注入被调用者（通过构造器/方法注入实现），这样我们就实现了调用者与被调用者的解耦，该过程被称为依赖注入。

```php
// 超能力接口
interface SuperModuleInterface{
    /**
     * 超能力激活方法
     *
     * 任何一个超能力都得有该方法，并拥有一个参数
     *@param array $target 针对目标，可以是一个或多个，自己或他人
     */
    public function activate(array $target);
}

/**
 * 终极炸弹 ，超能力之一， 实现超能力接口
 */
class UltraBomb implements SuperModuleInterface
{
    public function activate(array $targets)
    {
        $str = '';
        foreach ($targets as $target) {
            $str .= "对" . $target . "使用 UltraBomb\n";
        }
        return $str;
    }
}

/**
 * X-超能量 ， 超能力之一， 实现超能力接口
 */
class XPower implements SuperModuleInterface
{
    public function activate(array $targets)
    {
        $str = '';
        foreach ($targets as $target) {
            $str .= "对" . $target . "使用 XPower\n";
        }
        return $str;
    }
}

// 超人
class Superman
{
    protected $module;

    public function __construct(SuperModuleInterface $module)
    {
        $this->module = $module;
    }

    public function fire(array $targets){
        return $this->module->activate($targets);
    }
}

// 超能力模组
$superModule = new XPower();
// 初始化一个超人，并注入一个超能力模组依赖
$superMan = new Superman($superModule);
$result1 = $superMan->fire(['user1', 'user2']);
$this->assertEquals("对user1使用 XPower\n对user2使用 XPower\n", $result1);


// 超能力模组
$superModule2 = new UltraBomb();
// 初始化一个超人，并注入一个超能力模组依赖
$superMan2 = new Superman($superModule2);
$result2 = $superMan2->fire(['user3']);
$this->assertEquals("对user3使用 UltraBomb\n", $result2);
```

##### 原型模式

原型模式的核心思想是，先创建好一个原型对象，然后通过clone原型对象来创建新的对象。

这样就免去了类创建是重复的初始化操作。原型模式适用于对大对象的创建，大对象每次new消耗很大，原型模式仅需内存拷贝即可。

```php
/*抽象原型角色*/
abstract class Prototype {
    abstract function cloned();
}

/*具体原型角色*/
class Plane extends Prototype {
    public $color;

    public function Fly()
    {
        echo "飞机飞啊飞\n";
    }

    public function cloned()
    {
        return clone $this;
    }
}

/*客户角色*/
class Client {
    public static function main()
    {
        $plane1 = new Plane();

        $plane1->color = "blue";

        $plane2 = $plane1->cloned();

        $plane1->Fly();
        $plane2->Fly();

        echo "plane1的颜色为: {$plane1->color}\n";
        echo "plane2的颜色为: {$plane2->color}\n";
    }
}

Client::main();
```


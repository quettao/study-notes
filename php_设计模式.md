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

适配器设计模式只是将某个对象的接口适配为另外一个对象所期望的接口

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


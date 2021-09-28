### PHP --- 协程

#### 什么是协程？

==进程==就是二进制可执行文件在计算机内存里的一个运行实例，就好比你的.exe文件是个类，进程就是new出来的那个实例。

进程是计算机系统进行资源分配和调度的基本单位（调度单位这里别纠结线程进程的），每个CPU下同一时刻只能处理一个进程。

进程的切换需要进行系统调用，CPU要保存当前进程的各个信息，同时还会使CPUCache被废掉。

所以进程切换不到费不得已就不做。那么怎么实现『进程切换不到费不得已就不做』呢？

​	首先进程被切换的条件是：进程执行完毕、分配给进程的CPU时间片结束，系统发生中断需要处理，或者进程等待必要的资源（进程阻塞）等。你想下，前面几种情况自然没有什么话可说，但是如果是在阻塞等待，是不是就浪费了。

其实阻塞的话我们的程序还有其他可执行的地方可以执行，不一定要傻傻的等！所以就有了==线程==。线程简单理解就是一个『微进程』，专门跑一个函数（逻辑流）。所以我们就可以在编写程序的过程中将可以同时运行的函数用线程来体现了。线程有两种类型，一种是由内核来管理和调度。

我们说，只要涉及需要内核参与管理调度的，代价都是很大的。这种线程其实也就解决了当一个进程中，某个正在执行的线程遇到阻塞，我们可以调度另外一个可运行的线程来跑，但是还是在同一个进程里，所以没有了进程切换。

还有另外一种线程，他的调度是由程序员自己写程序来管理的，对内核来说不可见。这种线程叫做『用户空间线程』。

==协程==可以理解就是一种用户空间线程。

协程，有几个特点：

1. 协同，因为是由程序员自己写的调度策略，其通过协作而不是抢占来进行切换
2. 在用户态完成创建，切换和销毁
3. 从编程角度上看，协程的思想本质上就是控制流的主动让出（yield）和恢复（resume）机制
4. 迭代器经常用来实现协程

#### PHP实现协程

##### 可迭代对象

PHP5提供了一种定义对象的方法使其可以通过单元列表来遍历，例如用foreach语句。

你如果要实现一个可迭代对象，你就要实现  ==Iterator==  接口：

```PHP
class MyIterator implements Iterator 
{
    private $var = [];
    public function __construct($array) {
        if (is_array($array)) {
            $this->var = $array;
        }
    }
    
    public function rewind() {
        echo "rewinding\n";
        reset($this-var);
    }
    
    public function current() {
        $var = current($this->var);
        echo "current: $var\n";
        return $var;
    }
    
    public function key() {
        $var = key($this->var);
        echo "key: $var\n";
        return $var;
    }
    
    public function next() {
        $var = next($this->var);
        echo "next: $var\n";
        return $var;
    }
    
    public function valid() {
        $var = $this->current() !== false;
        echo "valid: $var\n";
        return $var;
    }   
}

$values = [1, 2, 3];
$it = new MyIterator($value);
foreach ($it as $k => $v) {
    print("$k: $v\n");
}
```

##### 生成器

可以说之前为了拥有一个能够被foreach遍历的对象，你不得不去实现一堆的方法，==yield==关键字就是为了简化这个过程。

生成器提供了一种更容易的方法来实现简单的对象迭代，相比较定义类实现Iterator接口的方式，性能开销和复杂性大大降低。

```php
function xrange($start, $end, $step = 1) {
    for ($i = $start; $i <= $end; $i += $step) {
        yield $i;
    }
}

foreach (xrange(1, 100000) as $num) {
    echo $num, "\n";
}
```

记住，一个函数中如果用了yield，他就是一个生成器，直接调用他是没有用的，不能等同于一个函数那样去执行！

所以，yield就是yield，下次别再说yield是协程.

##### PHP协程

###### 0）生成器正确使用

既然生成器不能像函数一样直接调用，那么怎么才能调用呢？

方法如下：

- foreach他
- send($value)
- current / next...

###### 1）Task实现

Task就是一个任务的抽象，刚刚我们说了协程就是用户空间协程，线程可以理解就是跑一个函数。

所以Task的构造函数中就是接收一个闭包函数，我们命名为coroutine。

```php
// task任务类
class Task
{
    protected $taskId;
    protected $coroutine;
    protected $beforeFirstYield = true;
    protected $sendValue;
    
    /**
      * Task constructor.
      * @param $taskId
      * @param Generator $coroutine
      */
    public function __construct($taskId, Generator $coroutine) {
        $this->taskId = $taskId;
        $this->coroutine = $coroutine;
    }
    
    /**
      * 获取当前的Task的ID
      * 
      * @return mixed
      */
    public function getTaskId() {
        return $this->taskId;
    }
    
    /**
      * 判断Task执行完毕了没有
      * 
      * @return bool
      */
    public function isFinished() {
        return !$this->coroutine->valid();
    }
    /**
      * 设置下次要传给协程的值，比如 $id = (yield $xxxx)，这个值就给了$id了
      * 
      * @param $value
      */
    public function setSendValue($value) {
        $this->sendValue = $value;
    }
    
     /**
      * 运行任务
      * 
      * @return mixed
      */
    public function run() {
        // 这里要注意，生成器的开始会reset，所以第一个值要用current 获取
        if ($this->beforeFirstYield) {
            $this->beforeFirstYield = false;
            return $this->coroutine->current();
        } else {
            // 我们说过了， 用send去调用一个生成器
            $retval = $this->coroutine->send($this->sendValue);
            $this->sendValue = null;
            return $retval;
        }
    }
}
```

###### 2）Scheduler实现

接下来就是Scheduler这个重点核心部分，他扮演着调度员的角色。

```php
https://zhuanlan.zhihu.com/p/94145603
```


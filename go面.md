## Go

### slice

#### Slice扩容机制

```
GO1.17版本及之前
当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容；
当原 slice 容量 < 1024 的时候，新 slice 容量变成原来的 2 倍；
当原 slice 容量 > 1024，进入一个循环，每次容量变成原来的1.25倍,直到大于期望容量。

GO1.18之后
当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容；
当原 slice 容量 < threshold(256) 的时候，新 slice 容量变成原来的 2 倍；
当原 slice 容量 > threshold(256)，进入一个循环，每次容量增加（旧容量+3*threshold）/4。

```

#### 切片与数组的区别

```
共同点：
①都是存储一系列相同类型的数据结构
②都可以通过下标来访问
③都有len和cap这种概念
不同点：
①数组是定长的，且大小不能更改，是值类型。比如在函数参数传入的时候，形参和实参类型必须一模一样的
②切片是不定长的，容量是可以自动扩容的。切片传入函数的时候是值类型
```

#### nil切片和空切片

```go
// nil切片：
var slice []int
slice := *new([]int)  //new前的*是解引用

// 空切片
slice := []int{}
slice := make([]int)

这两种方式的len和cap均为0
但是不同的是：
nil切片和nil的比较结果是true
空切片和nil的比较结果是false，且同一程序里面，任何类型的空切片的底层数组指针的都指向同一地址
```

#### 若截取旧切片的一小段给新切片，旧切片就不用了，那么共享数组的未截取的值会被释放内存吗？

```
答案是：不会被释放
```

#### 切片的底层原理以及扩容机制

##### 底层原理

```
切片的底层的数组内存空间是连续分配的，可以通过下标获取。但是是固定长度的，当append发生扩容的时候就会创建新的切片，并发生迁移复制。所以扩容操作是为切片分配新的内存空间并复制迁移原切片中元素的过程。
```

#####  扩容大小的计算

#### slice 为什么不是线程安全的

```
slice底层结构并没有使用加锁的方式,不支持并发读写
```

### map

#### map 底层原理

```c
map 是一个指针 占用8个字节(64位计算机),指向hmap结构体,hmap包含多个bmap数组(桶) 
  go的map使用的是哈希表，解决冲突的方式使用的是拉链法。
map底层结构体中维护了B和buckets字段（当然还有其他字段）：buckets是一个数组指针（对应拉链法的数组），2的B次方表示这个数组的长度，这个数组存放的类型是bmap结构体（该结构体对应拉链法每个下标所指向的单链表），也就是我们常说的”桶“，bmap结构体维护了tophash、key、value三个数组和overflow指针，三个数组的长度都是8，表示每个桶最多存放8个键值对。三个数组的下标是一一对应的关系，什么意思呢，比如，要查一个hash，和tophash下标为3的值一样，那么要查找的key和value对应的下标也就是3。bmap的内存模型呢，并不是一个key一个value依次存放，而是所有的key放在一个数组，所有的value放在一个数组，只要使其key与value相应的下标相同即可，这样做的目的是，某些情况下可以省略掉 pad字段，节省内存空间。overflow又是一个bmap类型的指针，表示溢出桶，溢出桶和正常桶结构一样，存储kv时，当正常桶的8个位置用完了，就会申请溢出桶，将kv存放在溢出桶。

type hmap struct { 
    count int  //元素个数，调用len(map)时直接返回 
    flags uint8  //标志map当前状态,正在删除元素、添加元素..... 
    B uint8  //单元(buckets)的对数 B=5表示能容纳32个元素  B随着map容量增大而变大
    noverflow uint16  //单元(buckets)溢出数量，如果一个单元能存8个key，此时存储了9个，溢出了，就需要再增加一个单元 
    hash0 uint32  //哈希种子 
    buckets unsafe.Pointer //指向单元(buckets)数组,大小为2^B，可以为nil 
    oldbuckets unsafe.Pointer //扩容的时候，buckets长度会是oldbuckets的两倍 
    nevacute uintptr  //指示扩容进度，小于此buckets迁移完成 
    extra *mapextra //与gc相关 可选字段 
}

type bmap struct { 
    tophash [bucketCnt]uint8 
} 
//实际上编译期间会生成一个新的数据结构  
type bmap struct { 
    topbits [8]uint8     //key hash值前8位 用于快速定位keys的位置
    keys [8]keytype     //键
    values [8]valuetype //值
    pad uintptr 
    overflow uintptr     //指向溢出桶 无符号整形 优化GC
}

```

#### map 扩容机制

```
扩容时机：向 map 插入新 key 的时候，会进行条件检测，符合下面这 2 个条件，就会触发扩容
扩容条件：
1.超过负载 map元素个数 > 6.5（负载因子） * 桶个数
2.溢出桶太多
当桶总数<2^15时，如果溢出桶总数>=桶总数，则认为溢出桶过多
当桶总数>2^15时，如果溢出桶总数>=2^15，则认为溢出桶过多

由于map扩容，需要将原来的kv复制到新的buckets上，如果同时有大量的kv复制，会影响性能。所以map的扩容动作并不是一次性完成的，而是渐进式的。当map进行插入或修改、删除 key 的时候，会先检查map是否处于扩容中的状态，如果还未搬迁完毕，就会触发搬迁 buckets 的工作。

扩容机制：
双倍扩容：针对条件1，新建一个buckets数组，新的buckets大小是原来的2倍，然后旧buckets数据搬迁到新的buckets。
等量扩容：针对条件2，并不扩大容量，buckets数量维持不变，重新做一遍类似双倍扩容的搬迁动作，把松散的键值对重新排列一次，使得同一个 bucket 中的 key 排列地更紧密，节省空间，提高 bucket 利用率，进而保证更快的存取。
渐进式扩容：
插入修改删除key的时候，都会尝试进行搬迁桶的工作，每次都会检查oldbucket是否nil，如果不是nil则每次搬迁2个桶，蚂蚁搬家一样渐进式扩容
```

#### key应该搬迁到哪个桶里面呢？

```
对于等量扩容而言，桶的数量并没有改变，只是将同一桶中的key，排布的更加紧密，所以key仍旧放置于原来的桶号；对于翻倍扩容，因为桶的数量翻倍了，即B加一，所以在进行key定位的时候，需要用hash的后B+1位，来计算迁移至哪一个桶中，所以导致同一桶中的key，会被分到两个不同的桶中。
```



#### map 遍历为什么无序

```go
map每次遍历,都会从一个随机的桶,再从其中随机的key开始遍历,并且扩容后,原来桶中的key会落到其他桶中,本身就会造成失序
如果想顺序遍历map,先把key放到切片排序,再按照key的顺序遍历map

var sl []int
for k := range m {
    sl = append(sl, k)
}
sort.Ints(sl)
for _,k:= range sl {
    fmt.Print(m[k])
}

```

#### map 为什么不是线程安全的

````
map设计就不是用来多个协程高并发访问的
多个协程同时对map进行并发读写,程序会panic
如果想线程安全,可以使用sync.RWLock 锁

sync.map
这个包里面的map实现了锁,是线程安全的

````

#### Map 如何查找

```
1.写保护机制
先查hmap的标志位flags,如果flags写标志位此时是1,说明其他协程正在写操作,直接panic
2.计算hash值
key经过哈希函数计算后,得到64bit(64位CPU)
10010111 | 101011101010110101010101101010101010 | 10010
3.找到hash对应的桶
上面64位后5(hmap的B值)位定位所存放的桶
如果当前正在扩容中,并且定位到旧桶数据还未完成迁移,则使用旧的桶
4.遍历桶查找
上面64位前8位用来在tophash数组查找快速判断key是否在当前的桶中,如果不在需要去溢出桶查找  
5.返回key对应的指针

```

#### map的删除

```
key，value清零
对应位置的tophash置为Empty
```

#### map 中删除一个 key，它的内存会释放么？

```
① 如果删除的键值对都是值类型（int，float，bool，string以及数组和struct），map的内存不会自动释放
② 如果删除的键值对中有（指针，slice，map，chan等），且该引用未被程序的其他位置使用，则该引用的内存会被释放，但是map中为存放这个类型而申请的内存不会被释放。
上述两种情况，map为存储键值所申请的空间，均不会被立即释放。等待GC到来，才会被释放。
③ 将map设置为nil后，内存被回收。
```

#### 运行下面程序发生什么？

```go
type Student struct {
	Name string
}

var list map[string]Student

func main() {

	list = make(map[string]Student)

	student := Student{"Aceld"}

	list["student"] = student
	list["student"].Name = "LDB"

	fmt.Println(list["student"])
}
编译失败
map[string]Student 的value是一个Student结构值，所以当list[“student”] = student,是一个值拷贝过程。而list[“student”]则是一个值引用。那么值引用的特点是只读。所以对list[“student”].Name = "LDB"的修改是不允许的。
应该改为list = make(map[string]*Student)
```



#### Map 冲突解决方式

```
GO采用链地址法解决冲突，具体就是插入key到map中时，当key定位的桶填满8个元素后，将会创建一个溢出桶，并且将溢出桶插入当前桶的所在链表尾部
```

#### Map 负载因子为什么是 6.5

```
负载因子 = 哈希表存储的元素个数 / 桶个数
Go 官方发现：装载因子越大，填入的元素越多，空间利用率就越高，但发生哈希冲突的几率就变大。
            装载因子越小，填入的元素越少，冲突发生的几率减小，但空间浪费也会变得更多，而且还会提高扩容操作的次数
Go 官方取了一个相对适中的值，把 Go 中的 map 的负载因子硬编码为 6.5，这就是 6.5 的选择缘由。
这意味着在 Go 语言中，当 map存储的元素个数大于或等于 6.5 * 桶个数 时，就会触发扩容行为。

```

#### Map 和 Sync.Map 哪个性能好

```
type Map struct {
    mu Mutex
    read atomic.Value
    dirty map[interface()]*entry
    misses int
}
对比原始map：
和原始map+RWLock的实现并发的方式相比，减少了加锁对性能的影响。它做了一些优化：可以无锁访问read map，而且会优先操作read map，倘若只操作read map就可以满足要求，那就不用去操作write map(dirty)，所以在某些特定场景中它发生锁竞争的频率会远远小于map+RWLock的实现方式
优点：
适合读多写少的场景
缺点：
写多的场景，会导致 read map 缓存失效，需要加锁，冲突变多，性能急剧下降

```

### channel

#### Channel 底层实现原理

```
通过var声明或者make函数创建的channel变量是一个存储在函数栈上的指针，占用8个字节，指向堆上的hchan结构体
channel的数据结构包括：底层存放缓冲的数据格式是一个循环数组
重要的字段（当然，不止这些字段）：
buf表示循环数组的指针
sendx表示循环数组中将要存放元素的位置索引（向通道发送）
recvx表示循环数组中将要读出元素的位置索引（从通道接收）
sendq表示等待发送元素的协程队列（双向链表），其节点的类型是sudog，表示一个协程的信息
recvq表示等待接收元素的协程队列（双向链表）
lock锁保证通道操作是原子的

type hchan struct {
    closed   uint32   // channel是否关闭的标志
    elemtype *_type   // channel中的元素类型
    // channel分为无缓冲和有缓冲两种。
    // 对于有缓冲的channel存储数据，使用了 ring buffer（环形缓冲区) 来缓存写入的数据，本质是循环数组
    // 为啥是循环数组？普通数组不行吗，普通数组容量固定更适合指定的空间，弹出元素时，普通数组需要全部都前移
    // 当下标超过数组容量后会回到第一个位置，所以需要有两个字段记录当前读和写的下标位置
    buf      unsafe.Pointer // 指向底层循环数组的指针（环形缓冲区）
    qcount   uint           // 循环数组中的元素数量
    dataqsiz uint           // 循环数组的长度
    elemsize uint16                 // 元素的大小
    sendx    uint           // 下一次写下标的位置
    recvx    uint           // 下一次读下标的位置
    // 尝试读取channel或向channel写入数据而被阻塞的goroutine
    recvq    waitq  // 读等待队列
    sendq    waitq  // 写等待队列
    lock mutex //互斥锁，保证读写channel时不存在并发竞争问题
}
等待队列：
双向链表，包含一个头结点和一个尾结点
每个节点是一个sudog结构体变量，记录哪个协程在等待，等待的是哪个channel，等待发送/接收的数据在哪里
type waitq struct {
    first *sudog
    last  *sudog
}

type sudog struct {
    g *g
    next *sudog
    prev *sudog
    elem unsafe.Pointer 
    c        *hchan 
    ...
}

创建时:
创建时会做一些检查:
-   元素大小不能超过 64K
-   元素的对齐大小不能超过 maxAlign 也就是 8 字节
-   计算出来的内存是否超过限制

创建时的策略:
-   如果是无缓冲的 channel，会直接给 hchan 分配内存
-   如果是有缓冲的 channel，并且元素不包含指针，那么会为 hchan 和底层数组分配一段连续的地址
-   如果是有缓冲的 channel，并且元素包含指针，那么会为 hchan 和底层数组分别分配地址

发送时:
-   如果 channel 的读等待队列存在接收者goroutine
-   将数据直接发送给第一个等待的 goroutine， 唤醒接收的 goroutine
-   如果 channel 的读等待队列不存在接收者goroutine
-   如果循环数组buf未满，那么将会把数据发送到循环数组buf的队尾
-   如果循环数组buf已满，这个时候就会走阻塞发送的流程，将当前 goroutine 加入写等待队列，并挂起等待唤醒

接收时:
-   如果 channel 的写等待队列存在发送者goroutine
-   如果是无缓冲 channel，直接从第一个发送者goroutine那里把数据拷贝给接收变量，唤醒发送的 goroutine
-   如果是有缓冲 channel（已满），将循环数组buf的队首元素拷贝给接收变量，将第一个发送者goroutine的数据拷贝到 buf循环数组队尾，唤醒发送的 goroutine
-   如果 channel 的写等待队列不存在发送者goroutine
-   如果循环数组buf非空，将循环数组buf的队首元素拷贝给接收变量
-   如果循环数组buf为空，这个时候就会走阻塞接收的流程，将当前 goroutine 加入读等待队列，并挂起等待唤醒


```

#### Channel 有什么特点

```
channel有2种类型：无缓冲、有缓冲
channel有3种模式：写操作模式（单向通道）、读操作模式（单向通道）、读写操作模式（双向通道）
写操作模式    make(chan<- int)
读操作模式    make(<-chan int)
读写操作模式    make(chan int)

```

channel 有 3 种状态：未初始化、正常、关闭

操作 \ 状态	未初始化					关闭																  正常
关闭				panic						panic																正常
发送				永远阻塞导致死锁	panic																阻塞或者成功发送
接收				永远阻塞导致死锁	缓冲区为空则为零值，否则可以继续读			阻塞或者成功接收

```
注意点：
一个 channel不能多次关闭，会导致panic
如果多个 goroutine 都监听同一个 channel，那么 channel 上的数据都可能随机被某一个 goroutine 取走进行消费
如果多个 goroutine 监听同一个 channel，如果这个 channel 被关闭，则所有 goroutine 都能收到退出信号

```

#### Go缓冲通道与无缓冲通道的区别

```
无缓冲通道，在初始化的时候，不用添加缓冲区大小；
无缓冲通道的发送与接收（或者接收与发送）是同步的；
发送者的发送操作将被阻塞，直到有接收者接收数据；接收者的接受操作将被阻塞，直到有发送者发送数据。

有缓冲通道，在初始化的时候，需要指定缓冲区大小；
有缓冲通道的发送与接收（或者接收与发送）是不同步的，也就是异步的；
有缓冲通道，缓冲区满后，再继续执行发送操作，会被阻塞。（其实无缓冲通道可以想象为是一个一直满的通道）
```



#### Channel 为什么是线程安全的

```
不同协程通过channel进行通信，本身的使用场景就是多线程，为了保证数据的一致性，必须实现线程安全
channel的底层实现中，hchan结构体中采用Mutex锁来保证数据读写安全。在对循环数组buf中的数据进行入队和出队操作时，必须先获取互斥锁，才能操作channel数据

```

#### Channel 发送和接收什么情况下会死锁？

```go
func deadlock1() {    //无缓冲channel只写不读
    ch := make(chan int) 
    ch <- 3 //  这里会发生一直阻塞的情况，执行不到下面一句
}
func deadlock2() { //无缓冲channel读在写后面
    ch := make(chan int)
    ch <- 3  //  这里会发生一直阻塞的情况，执行不到下面一句
    num := <-ch
    fmt.Println("num=", num)
}
func deadlock3() { //无缓冲channel读在写后面
    ch := make(chan int)
    ch <- 100 //  这里会发生一直阻塞的情况，执行不到下面一句
    go func() {
        num := <-ch
        fmt.Println("num=", num)
    }()
    time.Sleep(time.Second)
}
func deadlock3() {    //有缓冲channel写入超过缓冲区数量
    ch := make(chan int, 3)
    ch <- 3
    ch <- 4
    ch <- 5
    ch <- 6  //  这里会发生一直阻塞的情况
}
func deadlock4() {    //空读
    ch := make(chan int)
    // ch := make(chan int, 1)
    fmt.Println(<-ch)  //  这里会发生一直阻塞的情况
}
func deadlock5() {    //互相等对方造成死锁
    ch1 := make(chan int)
    ch2 := make(chan int)
    go func() {
        for {
        select {
        case num := <-ch1:
            fmt.Println("num=", num)
            ch2 <- 100
        }
    }
    }()
    for {
        select {
        case num := <-ch2:
            fmt.Println("num=", num)
            ch1 <- 300
        }
    }
}

```

#### 如何优雅的关闭channel

```go
关闭 channel 的基本原则
不要从接收端关闭 channel，也不要在有多个发送端时，主动关闭 channel
这个原则的来源就因为：
1. 不能向已关闭的 channel 发送数据
2. 不能重复关闭已关闭的 channel

关于 channel 的使用，有几点不方便的地方：
1. 在不改变 channel 自身状态的情况下，无法获知一个 channel 是否关闭。
2. 关闭一个 closed channel 会导致 panic。所以，如果关闭 channel 的一方在不知道 channel 是否处于关闭状态时就去贸然关闭 channel 是很危险的事情。
3. 向一个 closed channel 发送数据会导致 panic。所以，如果向 channel 发送数据的一方不知道 channel 是否处于关闭状态时就去贸然向 channel 发送数据是很危险的事情。

那到底应该如何优雅地关闭 channel？
根据 sender 和 receiver 的个数，分下面几种情况：
1. 一个 sender，一个 receiver
2. 一个 sender， M 个 receiver
3. N 个 sender，一个 reciver
4. N 个 sender， M 个 receiver

对于 1，2，只有一个 sender 的情况就不用说了，直接从 sender 端关闭就好了.

第 3 种情形下，优雅关闭 channel 的方法是：
解决方案就是增加一个传递关闭信号的 channel，receiver 通过信号 channel 下达关闭数据 channel 指令。senders 监听到关闭信号后，停止发送数据。代码如下：
func main() {
    rand.Seed(time.Now().UnixNano())

    const Max = 100000
    const NumSenders = 1000

    dataCh := make(chan int, 100)
    stopCh := make(chan struct{})

    // senders
    for i := 0; i < NumSenders; i++ {
        go func() {
            for {
                select {
                case <- stopCh:
                    return
                case dataCh <- rand.Intn(Max):
                }
            }
        }()
    }

    // the receiver
    go func() {
        for value := range dataCh {
            if value == Max-1 {
                fmt.Println("send stop signal to senders.")
                close(stopCh)
                return
            }

            fmt.Println(value)
        }
    }()

    select {
    case <- time.After(time.Hour):
    }
}

这里的 stopCh 就是信号 channel，它本身只有一个 sender，因此可以直接关闭它。senders 收到了关闭信号后，select 分支 “case <- stopCh” 被选中，退出函数，不再发送数据。

第 4 种情形下，优雅关闭 channel 的方法是：
这里有 M 个 receiver，如果直接还是采取第 3 种解决方案，由 receiver 直接关闭 stopCh 的话，就会重复关闭一个 channel，导致 panic。因此需要增加一个中间人，M 个 receiver 都向它发送关闭 dataCh 的“请求”，中间人收到第一个请求后，就会直接下达关闭 dataCh 的指令（通过关闭 stopCh，这时就不会发生重复关闭的情况，因为 stopCh 的发送方只有中间人一个）。另外，这里的 N 个 sender 也可以向中间人发送关闭 dataCh 的请求。
func main() {
    rand.Seed(time.Now().UnixNano())

    const Max = 100000
    const NumReceivers = 10
    const NumSenders = 1000

    dataCh := make(chan int, 100)
    stopCh := make(chan struct{})

    // It must be a buffered channel.
    toStop := make(chan string, 1)

    var stoppedBy string

    // moderator
    go func() {
        stoppedBy = <-toStop
        close(stopCh)
    }()

    // senders
    for i := 0; i < NumSenders; i++ {
        go func(id string) {
            for {
                value := rand.Intn(Max)
                if value == 0 {
                    select {
                    case toStop <- "sender#" + id:
                    default:
                    }
                    return
                }

                select {
                case <- stopCh:
                    return
                case dataCh <- value:
                }
            }
        }(strconv.Itoa(i))
    }

    // receivers
    for i := 0; i < NumReceivers; i++ {
        go func(id string) {
            for {
                select {
                case <- stopCh:
                    return
                case value := <-dataCh:
                    if value == Max-1 {
                        select {
                        case toStop <- "receiver#" + id:
                        default:
                        }
                        return
                    }

                    fmt.Println(value)
                }
            }
        }(strconv.Itoa(i))
    }

    select {
    case <- time.After(time.Hour):
    }

}
代码里 toStop 就是中间人的角色，使用它来接收 senders 和 receivers 发送过来的关闭 dataCh 请求。
这里将 toStop 声明成了一个 缓冲型的 channel。假设 toStop 声明的是一个非缓冲型的 channel，那么第一个发送的关闭 dataCh 请求可能会丢失。因为无论是 sender 还是 receiver 都是通过 select 语句来发送请求，如果中间人所在的 goroutine 没有准备好，那 select 语句就不会选中，直接走 default 选项，什么也不做。这样，第一个关闭 dataCh 的请求就会丢失。

如果，我们把 toStop 的容量声明成 Num(senders) + Num(receivers)，那发送 dataCh 请求的部分可以改成更简洁的形式：
...
toStop := make(chan string, NumReceivers + NumSenders)
...
            value := rand.Intn(Max)
            if value == 0 {
                toStop <- "sender#" + id
                return
            }
...
                if value == Max-1 {
                    toStop <- "receiver#" + id
                    return
                }
...
直接向 toStop 发送请求，因为 toStop 容量足够大，所以不用担心阻塞，自然也就不用 select 语句再加一个 default case 来避免阻塞。
```



#### 互斥锁实现原理

```
Go sync包提供了两种锁类型：互斥锁sync.Mutex 和 读写互斥锁sync.RWMutex，都属于悲观锁。
锁的实现一般会依赖于原子操作、信号量，通过atomic 包中的一些原子操作来实现锁的锁定，通过信号量来实现线程的阻塞与唤醒

在正常模式下，锁的等待者会按照先进先出的顺序获取锁。但是刚被唤起的 Goroutine 与新创建的 Goroutine 竞争时，大概率会获取不到锁，在这种情况下，这个被唤醒的 Goroutine 会加入到等待队列的前面。 如果一个等待的 Goroutine 超过1ms 没有获取锁，那么它将会把锁转变为饥饿模式。
Go在1.9中引入优化，目的保证互斥锁的公平性。在饥饿模式中，互斥锁会直接交给等待队列最前面的 Goroutine。新的 Goroutine 在该状态下不能获取锁、也不会进入自旋状态，它们只会在队列的末尾等待。如果一个 Goroutine 获得了互斥锁并且它在队列的末尾或者它等待的时间少于 1ms，那么当前的互斥锁就会切换回正常模式。

```

#### 互斥锁允许自旋的条件？

```
线程没有获取到锁时常见有2种处理方式：
-   一种是没有获取到锁的线程就一直循环等待判断该资源是否已经释放锁，这种锁也叫做自旋锁，它不用将线程阻塞起来， 适用于并发低且程序执行时间短的场景，缺点是cpu占用较高
-   另外一种处理方式就是把自己阻塞起来，会释放CPU给其他线程，内核会将线程置为「睡眠」状态，等到锁被释放后，内核会在合适的时机唤醒该线程，适用于高并发场景，缺点是有线程上下文切换的开销
Go语言中的Mutex实现了自旋与阻塞两种场景，当满足不了自旋条件时，就会进入阻塞
允许自旋的条件：
1.  锁已被占用，并且锁不处于饥饿模式。
2.  积累的自旋次数小于最大自旋次数（active_spin=4）。
3.  cpu 核数大于 1。
4.  有空闲的 P。
5.  当前 goroutine 所挂载的 P 下，本地待运行队列为空。

```

#### 读写锁实现原理

```
读写锁的底层是基于互斥锁实现的。
写锁需要阻塞写锁：一个协程拥有写锁时，其他协程写锁定需要阻塞；
写锁需要阻塞读锁：一个协程拥有写锁时，其他协程读锁定需要阻塞；
读锁需要阻塞写锁：一个协程拥有读锁时，其他协程写锁定需要阻塞；
读锁不能阻塞读锁：一个协程拥有读锁时，其他协程也可以拥有读锁。

```

#### 原子操作有哪些

```
Go atomic包是最轻量级的锁（也称无锁结构），可以在不形成临界区和创建互斥量的情况下完成并发安全的值替换操作，不过这个包只支持int32/int64/uint32/uint64/uintptr这几种数据类型的一些基础操作（增减、交换、载入、存储等）
当我们想要对某个变量并发安全的修改，除了使用官方提供的 `mutex`，还可以使用 sync/atomic 包的原子操作，它能够保证对变量的读取或修改期间不被其他的协程所影响。
atomic 包提供的原子操作能够确保任一时刻只有一个goroutine对变量进行操作，善用 atomic 能够避免程序中出现大量的锁操作。
常见操作：
-   增减Add     AddInt32 AddInt64 AddUint32 AddUint64 AddUintptr
-   载入Load    LoadInt32 LoadInt64    LoadPointer    LoadUint32    LoadUint64    LoadUintptr    
-   比较并交换CompareAndSwap    CompareAndSwapInt32...
-   交换Swap    SwapInt32...
-   存储Store    StoreInt32...

```

#### 原子操作和锁的区别

```
原子操作由底层硬件支持，而锁是基于原子操作+信号量完成的。若实现相同的功能，前者通常会更有效率
原子操作是单个指令的互斥操作；互斥锁/读写锁是一种数据结构，可以完成临界区（多个指令）的互斥操作，扩大原子操作的范围
原子操作是无锁操作，属于乐观锁；说起锁的时候，一般属于悲观锁
原子操作存在于各个指令/语言层级，比如“机器指令层级的原子操作”，“汇编指令层级的原子操作”，“Go语言层级的原子操作”等。
锁也存在于各个指令/语言层级中，比如“机器指令层级的锁”，“汇编指令层级的锁”，“Go语言层级的锁”等

```

#### goroutine 的底层实现原理

```go
g本质是一个数据结构,真正让 goroutine 运行起来的是调度器
type g struct { 
    goid int64  // 唯一的goroutine的ID 
    sched gobuf // goroutine切换时，用于保存g的上下文 
    stack stack // 栈 
    gopc // pc of go statement that created this goroutine 
    startpc uintptr  // pc of goroutine function ... 
} 
type gobuf struct {     //运行时寄存器
    sp uintptr  // 栈指针位置 
    pc uintptr  // 运行到的程序位置 
    g  guintptr // 指向 goroutine 
    ret uintptr // 保存系统调用的返回值 ... 
} 
type stack struct {     //运行时栈
    lo uintptr  // 栈的下界内存地址 
    hi uintptr  // 栈的上界内存地址 
}


```

### goroutine

#### goroutine 和线程的区别

```
内存占用:
创建一个 goroutine 的栈内存消耗为 2 KB，实际运行过程中，如果栈空间不够用，会自动进行扩容。创建一个 thread 则需要消耗 1 MB 栈内存。
创建和销毀:
Thread 创建和销毀需要陷入内核,系统调用。而 goroutine 因为是由 Go runtime 负责管理的，创建和销毁的消耗非常小，是用户级。
切换:
当 threads 切换时，需要保存各种寄存器,而 goroutines 切换只需保存三个寄存器：Program Counter, Stack Pointer and BP。一般而言，线程切换会消耗 1000-1500 ns,Goroutine 的切换约为 200 ns,因此，goroutines 切换成本比 threads 要小得多。


```

#### goroutine 泄露场景

```
原因：
Goroutine 是轻量级线程，需要维护执行用户代码的上下文信息。在运行过程中也需要消耗一定的内存来保存这类信息，而这些内存在目前版本的 Go 中是不会被释放的。
因此，如果一个程序持续不断地产生新的 goroutine、且不结束已经创建的 goroutine 并复用这部分内存，就会造成内存泄漏的现象。

造成泄露的大多数原因有以下三种:
Goroutine 内进行channel/mutex 等读写操作被一直阻塞。
Goroutine 内的业务逻辑进入死循环，资源一直无法释放。
Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待

泄露场景
channel 如果忘记初始化，那么无论你是读，还是写操作，都会造成阻塞。
channel 发送数量 超过 channel接收数量，就会造成阻塞
channel 接收数量 超过 channel发送数量，也会造成阻塞
http request body未关闭，goroutine不会退出
互斥锁忘记解锁
sync.WaitGroup使用不当

如何排查
单个函数：调用 `runtime.NumGoroutine` 方法来打印 执行代码前后Goroutine 的运行数量，进行前后比较，就能知道有没有泄露了。
生产/测试环境：使用`PProf`实时监测Goroutine的数量

解决方法：

是的，这个回答也是对的，提供了一些其它的处理 Goroutine 可能的内存泄露的方法：

使用 Channel 接收业务完成的通知：确保每一个 Goroutine 在完成它的任务后都能发送一个完成的信号，这样创建 Goroutine 的代码就知道何时它们完成了任务，并可以安全的结束它们。

业务执行阻塞超过设定的超时时间，就会触发超时退出：对一些可能阻塞的操作设置超时时间，防止 Goroutine 无限期的等待下去。

使用 pprof 排查：pprof 是 Go 提供的性能分析工具，它可以帮助我们找出程序中 CPU、内存使用高的地方，包括 Goroutine 的堆栈信息等，是一个强大的诊断内存泄露的工具。

使用 context 进行超时控制：context 、是 Go 提供的一种可以包含多个 Goroutine 之间可共享的数据和状态的结构，我们可以使用它提供的 WithTimeout、WithCancel 方法对一些可能运行长时间的 Goroutine 进行超时设置或者强制取消。

利用 Go 的垃圾回收机制：Go 提供垃圾回收机制，可以自动的回收 Goroutine 运行中产生但没有被引用的内存，减少因疏忽而发生的内存泄露。


```

#### goroutine 什么时候会被挂起

```
goroutine 会在以下情况下被挂起：

发生阻塞，例如等待 I/O 操作的完成或者发送或接收通道上的数据时没有可用的对等方。
发生调用 runtime.Gosched ()，让出 CPU 给其他 goroutine 执行。
发生同步操作，例如 sync.Mutex 或 sync.WaitGroup 的锁定和解锁操作。
发生垃圾回收（GC）。
发生错误，例如 panic 或者超时。

```

#### 同时启动了一万个 goroutine，会如何调度

```
一万个 G 会按照 P 的设定个数，尽量平均地分配到每个 P 的本地队列中。如果所有本地队列都满了，那么剩余的 G 则会分配到 GMP 的全局队列上。接下来便开始执行 GMP 模型的调度策略：

本地队列轮转：每个 P 维护着一个包含 G 的队列，不考虑 G 进入系统调用或 IO 操作的情况下，P 周期性的将 G 调度到 M 中执行，执行一小段时间，将上下文保存下来，然后将 G 放到队列尾部，然后从队首中重新取出一个 G 进行调度。
系统调用：P 的个数默认等于 CPU 核数，每个 M 必须持有一个 P 才可以执行 G，一般情况下 M 的个数会略大于 P 的个数，这多出来的 M 将会在 G 产生系统调用时发挥作用。当该 G 即将进入系统调用时，对应的 M 由于陷入系统调用而进被阻塞，将释放 P，进而某个空闲的 M 获取 P，继续执行 P 队列中剩下的 G。
工作量窃取：多个 P 中维护的 G 队列有可能是不均衡的，当某个 P 已经将 G 全部执行完，然后去查询全局队列，全局队列中也没有新的 G，而另一个 M 中队列中还有很多 G 待运行。此时，空闲的 P 会将其他 P 中的 G 偷取一部分过来，一般每次偷取一半。
```



#### 如何查看正在运行的 goroutine 数量

```go
package main

import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    for i := 0; i < 100; i++ {
        go func() {
            select {}
        }()
    }
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    select {}
}
// 执行程序之后，命令运行以下命令，会自动打开浏览器显示一系列目前还看不懂的图，提示Could not execute dot; may need to install graphviz.则需要安装graphviz，需要python环境
go tool pprof -http=:1248 http://127.0.0.1:6060/debug/pprof/goroutine

```

#### 如何控制并发的 goroutine 数量？

```
在开发过程中，如果不对goroutine加以控制而进行滥用的话，可能会导致服务整体崩溃。比如耗尽系统资源导致程序崩溃，或者CPU使用率过高导致系统忙不过来。
解决方案：
有缓冲channel:利用缓冲满时发送阻塞的特性
无缓冲channel:任务发送和执行分离，指定消费者并发协程数


```

#### Goroutine 是什么？ 

```
这是一个基础问题，用来检查你对 Goroutine 的理解。你应该能够解释 Goroutine 是 Golang 特有的并发编程模型，更轻量级也更易于管理的线程。
Goroutine 是 Go 语言中并发设计的核心。是一种更轻量级比线程（thread）的执行体，它使得同时运行多项任务（函数或者方法）变得非常简单，类似线程但管理是在 Go 运行时（runtime），而非操作系统。
```

#### goroutine 创建流程

```
Go 语言中，goroutine 的创建非常简单，只需要使用 go 关键字即可。go 关键字后面跟随一个函数调用（或者匿名函数），该函数就会在新的 goroutine 中并发执行。

创建一个 Goroutine 会涉及以下一些步骤：

Goroutine 的申请 首先，当你在代码中使用 go 关键字启动一个 Goroutine，Go 语言运行时就会去申请一个 G（也就是 Goroutine）。这个 G 实体包含必要的信息：栈信息、栈大小、运行的状态等等。

Goroutine 的调度 Goroutine 转为待运行状态，其实就是放入到本地运行队列中。本地运行队列对应一个 P（Processor），P 可以看作是 Golang 运行时的上下文环境。每个 P 可以关联一个 M（Machine，系统线程）进行运行。

Goroutine 的运行 一旦调度器发现有空闲的 P 和需要运行的 G，就会安排他们一起进行任务的执行。

值得注意的是，创建 Goroutine 的操作非常轻量级，因为在 Go 语言运行时中，每个 Goroutine 的初始栈大小相比操作系统线程要小很多，一般只有 2KB~8KB，所以在实际编程中，我们可以大量并且自由地创建 Goroutine，让 Go 语言运行时进行调度和管理。
但是，如果你的程序中存在频繁创建和销毁 goroutine 的情况，应该考虑使用 sync.Pool 等技术来优化内存分配和回收效率。

```



#### Goroutine 与线程的区别是什么？

```
 这个问题用来评估你对更深层级的并发编程概念理解如何，并且能否对比不同的并发模型。
Goroutines 和操作系统线程最大的不同是它们的大小和启动速度。Goroutines 极其轻量级，一个 goroutine 初始只占用 4KB 内存，稍大的 goroutine 栈可能会自动扩容以容纳更大的数据，而线程的栈通常为 2MB。此外，Goroutines 的调度和管理都由 Go 语言的运行时环境负责，使得 Goroutines 更易于开发和维护。
```

#### Goroutine 是如何被调度的？

```
 这个问题需要你对 Golang 的调度器有深入的理解。你应该了解 Golang 是如何在操作系统线程之间切换 Goroutines 的，并能解释 M（内核线程）, P（处理器）, G（goroutine）这三者的关系。

Golang 的任务调度模型是基于 M（OS 主线程）, P（处理器，上下文环境）, G（goroutine）的。每个 P 上都有一个本地 goroutine 队列和一个全局的队列。新创建的 goroutine 首先会被放到本地队列，当本地队列满了或者为空时，会和全局的队列进行协调。P 执行 G 特定时间后，会检查是否有更高优先级的 G 需要运行，如系统调用，网络 IO 等
```

#### go 语言协程调度原理，协程为什么快

```
goroutine 的理解：
​ 是一种轻量级的用户态线程（可以避免用户态到核心态的切换，节省资源开销），由 Go 运行时调度器进行管理！在 Go 中，每一格协程都会被分配一个较小的栈空间（默认为 2KB），并以非常低的成本创建和销毁。
​ 当一个协程阻塞时，调度器会自动将其与当前协程解除绑定，并将其转移到等待队列中，然后运行其它的协程，从而实现了高效地利用了 CPU 的时间片。

Go 中协程的优势：
​ 协程不像传统多线程编程那样必须避免线程竞争和加锁解锁等操作。事实上，在 Go 中通常通过 channel 实现协程之间的同步和通信。Go 的 channel 机制提供了一种非常有效且安全的协程同步方式，可以避免竞争条件和死锁等问题。同时，协程可以自我调节、自我管理，从而可以避免了频繁的线程切换带来的性能损耗。这使得协程同游非常快的相应速度和高效的并发处理能力。

​ 总而言之：Go 的协程是一种轻量级、高效率的并发编程方式，得益于其独特的调度机制和 channel 通信机制，可以有效地利用 CPU 时间片，并具有比传统多线程编程更加安全和高效的优点。
```



#### 当函数执行结束时，goroutine 自动终止，其状态和堆栈等资源也会被回收，这句话如何理解

```
背：
简单来说就是：当你通过 go 关键字开启的函数运行完，那个由此函数驱动的 Goroutine 任务就结束了，和它相关联的内存和状态信息会被 Go 语言的运行环境自动清理和回收。

扯：
理解这句话的关键是理解 Go 语言中的 Goroutine 执行模型。

一个 Goroutine 对应一个函数（或者方法）的执行。就像任何函数执行一样，当函数的指令全部执行完毕，达到结束点（如遇到 return 指令或者到达函数体末尾）时，函数执行就被认为是结束。

针对 Goroutine，当 Goroutine 对应的函数执行结束后，Go 语言运行时系统自动将其标记为结束。运行时系统会在适当的时间，如垃圾回收周期，自动回收（recycle）它，同时也会回收与其关联的资源。

关于 “状态和堆栈等资源也会被回收”，Goroutine 的状态包括它当前处于运行（running）、就绪（runnable）或阻塞（blocked）等状态，在 Goroutine 结束后，这些状态信息都没必要保留并会被回收。同时，Goroutine 在运行过程中会使用到自己私有的一段栈内存（用于保存局部变量，函数调用链等信息），在 Goroutine 结束后，这块栈内存也会被回收，以便再次利用。

需要注意的是，Goroutine 结束并不影响其他正在运行的 Goroutine，它们会继续运行直到各自的函数执行完。这就是 Goroutine 的并发性。执行完的 Goroutine 不会对其他 Goroutine 造成任何影响，除非在有明确的资源共享（如通过通道（Channel）等机制共享数据）的上下文环境中。

```



#### 什么是 Golang 的 GOMAXPROCS？

```
你应该了解这个参数的作用以及如何设置它。
GOMAXPROCS 是一个调整可运行 goroutine 的 OS 线程数量的环境变量，在 Go 1.5 之后默认为机器上的 CPU 核心数。通过设置这个值，我们可以调整 Go 并发程序的并行度。一般情况下，不需要手动调整，Go 语言运行环境会自动进行管理。 
```

#### 请解释下 Golang 中的 Channel，并描绘一下它们在 Goroutines 之间的通信中起什么作用？ 

```
这个问题旨在了解你是否理解 Golang 的 CSP 并发模型，也就是通过 Channels 在 Goroutines 之间传递数据以达到同步的目的。

Channel 在 Golang 里是一种数据类型，它是用来处理数据共享，实现多 Goroutine 之间的通信的一种重要途径。我们可以把它看作是 Goroutines 之间通信的管道。一方面，Goroutines 可以通过 Channel 发送和接收数据，实现数据的共享。另一方面，Channel 还可以用于实现锁的作用，保证并发环境下的数据安全。通过合理使用 Channel，我们可以高效的解决并发编程中的同步和通信问题。
```

### GMP

#### 什么是GMP模型？

```
先说一下GMP模型的作用：当我们写一个并发程序，操作系统会对其进行调度，线程是操作系统调度的最小单位，而不是协程，所以GMP模型就是想办法将用户创建的众多协程分配到线程上的这么一个过程
再分别说一下G、M、P是什么，以及各自干什么工作
```

#### P和M的数量以及何时被创建？

#### GO 线程模型如何实现

```
M个线程对应N个内核线程
优点：
-   能够利用多核
-   上下文切换成本低
-   如果进程中的一个线程被阻塞，不会阻塞其他线程，是能够切换同一进程内的其他线程继续执行
```

#### GMP 和 GM 模型

```
G：Goroutine  是 Go 的协程 / 轻量级线程，Go 通过他们实现并发操作。
M: 线程。代表 Machine，是系统线程，是运行 Goroutine 的实体。
P: Processor 本地队列。是 M 和 G 之间的调度器。

GM模型：
2012年前的调度器模型，使用了4年果断被抛弃，缺点如下：
1.  创建、销毁、调度G都需要每个M获取锁，这就形成了激烈的锁竞争。
2.  M转移G会造成延迟和额外的系统负载。比如当G中包含创建新协程的时候，M创建了G’，为了继续执行G，需要把G’交给M’执行，也造成了很差的局部性，因为G’和G是相关的，最好放在M上执行，而不是其他M'。
3.  系统调用(CPU在M之间的切换)导致频繁的线程阻塞和取消阻塞操作增加了系统开销。

GMP模型：
P的数量：
由启动时环境变量`$GOMAXPROCS`或者是由`runtime`的方法`GOMAXPROCS()`决定
M的数量:
go语言本身的限制：go程序启动时，会设置M的最大数量，默认10000.但是内核很难支持这么多的线程数
runtime/debug中的SetMaxThreads函数，设置M的最大数量
一个M阻塞了，会创建新的M。

P何时创建：在确定了P的最大数量n后，运行时系统会根据这个数量创建n个P。
M何时创建：没有足够的M来关联P并运行其中的可运行的G。比如所有的M此时都阻塞住了，而P中还有很多就绪任务，就会去寻找空闲的M，而没有空闲的，就会去创建新的M。

全场景解析：
1.P拥有G1，M1获取P后开始运行G1，G1创建了G2，为了局部性G2优先加入到P1的本地队列。
2.G1运行完成后，M上运行的goroutine切换为G0，G0负责调度时协程的切换。从P的本地队列取G2，从G0切换到G2，并开始运行G2。实现了线程M1的复用。
3.假设每个P的本地队列只能存4个G。G2要创建了6个G，前4个G（G3, G4, G5, G6）已经加入p1的本地队列，p1本地队列满了。
4.G2在创建G7的时候，发现P1的本地队列已满，需要执行负载均衡(把P1中本地队列中前一半的G，还有新创建G转移到全局队列)，这些G被转移到全局队列时，会被打乱顺序
5.G2创建G8时，P1的本地队列未满，所以G8会被加入到P1的本地队列。
6.在创建G时，运行的G会尝试唤醒其他空闲的P和M组合去执行。假定G2唤醒了M2，M2绑定了P2，并运行G0，但P2本地队列没有G，M2此时为自旋线程
7.M2尝试从全局队列取一批G放到P2的本地队列，至少从全局队列取1个g，但每次不要从全局队列移动太多的g到p本地队列，给其他p留点。
8.假设G2一直在M1上运行，经过2轮后，M2已经把G7、G4从全局队列获取到了P2的本地队列并完成运行，全局队列和P2的本地队列都空了,那m就要执行work stealing(偷取)：从其他有G的P哪里偷取一半G过来，放到自己的P本地队列。P2从P1的本地队列尾部取一半的G
9.G1本地队列G5、G6已经被其他M偷走并运行完成，当前M1和M2分别在运行G2和G8，M3和M4没有goroutine可以运行，M3和M4处于自旋状态，它们不断寻找goroutine。系统中最多有GOMAXPROCS个自旋的线程，多余的没事做线程会让他们休眠。
10.假定当前除了M3和M4为自旋线程，还有M5和M6为空闲的线程，G8创建了G9，G8进行了阻塞的系统调用，M2和P2立即解绑，P2会执行以下判断：如果P2本地队列有G、全局队列有G或有空闲的M，P2都会立马唤醒1个M和它绑定，否则P2则会加入到空闲P列表，等待M来获取可用的p。
11.G8创建了G9，假如G8进行了非阻塞系统调用。M2和P2会解绑，但M2会记住P2，然后G8和M2进入系统调用状态。当G8和M2退出系统调用时，会尝试获取P2，如果无法获取，则获取空闲的P，如果依然没有，G8会被记为可运行状态，并加入到全局队列,M2因为没有P的绑定而变成休眠状态

其他的一些小解释：
① P的数量和M的数量问题

P的最大个数是由GOMAXPROCS确定的，用户可以进行设置【P的局部队列最大不超过256】，程序启动所有的P便会创建。
M的最大个数也是可以进行设置的。
M在运行中的个数是不确定的，因为随着空闲忙碌与否，M会被创建以及被回收。
② 调度器的设计策略：线程复用，避免频繁的创建、销毁线程

work stealing机制：当本MP中没有G的时候，会去其他的MP组合中窃取一部分G，放入到自己的局部队列中。而不是将本M销毁。
hand off机制：当M中运行的G发生了阻塞的时候，M会主动释放掉P，让P去寻找空闲的M。
③ M0和G0

M0是程序启动后编号为0的主线程，负责初始化操作以及启动第一个G，之后便和其他的M一样了。
G0：每创建一个M都会启动一个G0，也就是每一个M都有属于自己的唯一的一个G0，G0仅用于该M上的G的调度。例如，M运行完G1之后，会先切换到G0，再有G0调度到下一个G2。
④ 自旋线程
定义的话如上边所示，那么为什么要这样设计？

自旋协程运行，肯定是占用一定的CPU资源，但是销毁再创建M也会消耗时间资源，我们希望当新的G到来的时候，能有M可以立即对其进行接收。
⑤ P会周期性的检查全局队列中有无G，防止里面的G被饿死
```

#### 描述一个协程的调度过程

```
① 通过go func创建一个协程G【这个G肯定是被某一个MP组合中正在运行的G创建的】。
② 有两种存储G的队列，一个是全局队列，一个是P维护的局部队列。新创建的G会先被加入创建它的MP组合的那个P的局部队列中（保证局部性），如果该P的局部队列已经满了，则会被放到全局队列。
③ 【正常】M会从关联的P中获取G（如果有G），放入到M中执行，G执行完毕之后，M在获取下一个G，如此循环。如果没有G，MP组合会去全局队列中获取一定数量的G，放到自己的P的局部队列中，调用执行。如果此时全局队列也没有G，该MP组合会去其他的MP组合的局部队列中窃取队列后半部分的一定数量的G，放到自己的P的局部队列中，调用执行。如果其他MP组合也没有G，那么该MP组合就会进入自旋线程【后序解释】，等待G的到来。
④ 【阻塞】假如M中执行的G发生了阻塞，M会释放所关联的P，P会去寻找一个休眠的M（休眠线程队列）关联，如果没有休眠线程，就会创建M。
⑤ 【苏醒】假如M中执行的G结束了④中的阻塞，又变为了可执行状态，这时M并没有关联P，所以从阻塞中恢复的协程，不能运行，只有关联了P才能运行。M在发生阻塞释放P的时候，会事先记录好是哪一个P处理器，当协程转为非阻塞状态后，M会先去找当时所关联的P，如果该P没有关联其他M，则M会重新绑定该P，并运行该非阻塞的G；如果该P已经绑定了其他M，会去找有没有空闲的P（没有绑定M的P），如果还是没有，该非阻塞的G会被放到全局队列，M会被放到休眠线程队列。
```



#### 怎么防止 G 全局队列饥饿

```
Golang 调度器采取了工作窃取（work-stealing）和手动调度（hand off）策略来防止 G 饥饿问题：

复用的 M 工作线程：当一个 G（goroutine）在执行 I/O 操作，等待某个条件或者是被阻塞，那么它将会释放其持有的 M 和 P，将 M 让给运行队列中的其他 G 使用，这样保证了 M 的利用效率。

手动调度： 当某个 G 在 M 上运行的时间太长，而其他 G 等待执行，调度器将会抢占这个 G 的执行，将执行权让给其他 G，这样可以保证所有的 G 都能均匀在 P 上执行，防止长期运行的 G 阻塞掉整个队列导致饥饿。

工作窃取：Golang 调度器还采用了一种叫做工作窃取（work-stealing）的策略。这是防止 G 饥饿的另一个重要的策略。当 goroutine 队列中的 G 都执行完成，而全局队列中还有等待执行的 G 时，M 就会从全局队列中（或者其他 P 中）“窃取” 一半的 G 到自己的队列中，这样保证了全局队列中的 G 得到执行，防止饥饿。

```



#### work stealing 机制？

```
当线程M⽆可运⾏的G时，尝试从其他M绑定的P偷取G，减少空转，提高了线程利用率（避免闲着不干活）。
当从本线程绑定 P 本地队列、全局G队列、netpoller都找不到可执行的 g，会从别的 P 里窃取G并放到当前P上面。
从netpoller 中拿到的G是_Gwaiting状态（ 存放的是因为网络IO被阻塞的G），从其它地方拿到的G是_Grunnable状态
从全局队列取的G数量：N = min(len(GRQ)/GOMAXPROCS + 1, len(GRQ/2)) （根据GOMAXPROCS负载均衡）
从其它P本地队列窃取的G数量：N = len(LRQ)/2（平分）
```

#### hand off 机制？

```
也称为P分离机制，当本线程 M 因为 G 进行的系统调用阻塞时，线程释放绑定的 P，把 P 转移给其他空闲的 M 执行，也提高了线程利用率（避免站着茅坑不拉shi）。
```

#### 如何查看运行时调度信息？

```
有 2 种方式可以查看一个程序的调度GMP信息，分别是go tool trace和GODEBUG
```





### 内存

#### 内存四区

```
代码区：存放代码
全局区：常量+全局变量。最终在进程退出时，由操作系统回收。
堆区：空间充裕，数据存放时间较久。一般由开发者分配，启动Golang的GC由GC清除机制自动回收。
栈区：空间较小，要求数据读写性能高，数据存放时间较短暂。由编译器自动分配和释放，存放函数的参数值、局部变量、返回值等、局部变量等(局部变量如果产生逃逸现象，可能会挂在在堆区)
```

#### 栈区和堆区的区别

```
栈区用于存储：系统分配的内存，例如，函数调用前后的上下文环境、函数参数、局部变量、函数返回值、函数返回地址等；堆区用于存储：用户通过malloc/new申请的内存。
栈的空间相较于堆小，栈的速度相较于堆快。
栈区是连续的，堆区是不连续的。
栈区的内存扩展方向是从高到低，堆区的内存扩展方向是由低到高。
```

#### go中变量分配在栈or堆

```
对于全部变量，值类型的全局变量分配在栈上，引用类型的全局变量分配在堆上；
对于局部变量，不能根据语句语义（var，new）去判断是分配到栈还是堆上，因为会发生内存逃逸（栈区的变量逃逸到堆上）；
当发现变量的作用域没有超出函数范围，就可以在栈上，反之则必须分配在堆上。
对于大对象（>32kb），是直接分配到堆区；
对于小对象（16B-32kb）以及微对象（<16B，且不含指针）很复杂，尚未了解；
```

#### 逃逸分析在何时进行

```
逸分析在编译阶段进行，由编译器完成。
```

#### 如何打印逃逸分析信息

```
go run -gcflags "-m -l" *.go
-m 设置打印信息
-l 禁止内联，内联编译，编译器会优化代码，可能导致不会发生逃逸。
```



#### 内存分配机制 



#### 内存逃逸机制

```
编译器会根据变量是否被外部引用来决定是否逃逸：
如果函数外部没有引用，则优先放到栈中；
如果函数外部存在引用，则必定放到堆中;
如果栈上放不下，则必定放到堆上;

案例：
指针逃逸：函数返回值为局部变量的指针，函数虽然退出了，但是因为指针的存在，指向的内存不能随着函数结束而回收，因此只能分配在堆上。
栈空间不足：当栈空间足够时，不会发生逃逸，但是当变量过大时，已经完全超过栈空间的大小时，将会发生逃逸到堆上分配内存。局部变量s占用内存过大，编译器会将其分配到堆上
变量大小不确定：编译期间无法确定slice的长度，这种情况为了保证内存的安全，编译器也会触发逃逸，在堆上进行分配内存
动态类型：动态类型就是编译期间不确定参数的类型、参数的长度也不确定的情况下就会发生逃逸
闭包引用对象：闭包函数中局部变量i在后续函数是继续使用的，编译器将其分配到堆上

总结：
1.  栈上分配内存比在堆中分配内存效率更高
2.  栈上分配的内存不需要 GC 处理，而堆需要
3.  逃逸分析目的是决定内分配地址是栈还是堆
4.  逃逸分析在编译阶段完成
因为无论变量的大小，只要是指针变量都会在堆上分配，所以对于小变量我们还是使用传值效率（而不是传指针）更高一点。

```

#### GO 内存对齐机制

```
什么是内存对齐
为了能让CPU可以更快的存取到各个字段，Go编译器会帮你把struct结构体做数据的对齐。所谓的数据对齐，是指内存地址是所存储数据大小（按字节为单位）的整数倍，以便CPU可以一次将该数据从内存中读取出来。编译器通过在结构体的各个字段之间填充一些空白已达到对齐的目的。存在内存空间的浪费，实际上是空间换时间
对齐原则：
1.  结构体变量中成员的偏移量必须是成员大小的整数倍
2.  整个结构体的地址必须是最大字节的整数倍

```

### GC

```
简答
三色标记算法将程序中的对象分成白色、黑色和灰色三类。

白色：不确定对象。
灰色：存活对象，子对象待处理。
黑色：存活对象。
标记开始时，所有对象加入白色集合（这一步需 STW ）。首先将根对象标记为灰色，加入灰色集合，垃圾搜集器取出一个灰色对象，将其标记为黑色，并将其指向的对象标记为灰色，加入灰色集合。重复这个过程，直到灰色集合为空为止，标记阶段结束。那么白色对象即可需要清理的对象，而黑色对象均为根可达的对象，不能被清理。

三色标记法因为多了一个白色的状态来存放不确定对象，所以后续的标记阶段可以并发地执行。当然并发执行的代价是可能会造成一些遗漏，因为那些早先被标记为黑色的对象可能目前已经是不可达的了。所以三色标记法是一个 false negative（假阴性）的算法。

三色标记法并发执行仍存在一个问题，即在 GC 过程中，对象指针发生了改变。比如下面的例子：

A (黑) -> B (灰) -> C (白) -> D (白)
正常情况下，D 对象最终会被标记为黑色，不应被回收。但在标记和用户程序并发执行过程中，用户程序删除了 C 对 D 的引用，而 A 获得了 D 的引用。标记继续进行，D 就没有机会被标记为黑色了（A 已经处理过，这一轮不会再被处理）。

A (黑) -> B (灰) -> C (白) 
  ↓
 D (白)
为了解决这个问题，Go 使用了内存屏障技术，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，类似于一个钩子。垃圾收集器使用了写屏障（Write Barrier）技术，当对象新增或更新时，会将其着色为灰色。这样即使与用户程序并发执行，对象的引用发生改变时，垃圾收集器也能正确处理了。

一次完整的 GC 分为四个阶段：

1）标记准备(Mark Setup，需 STW)，打开写屏障(Write Barrier)
2）使用三色标记法标记（Marking, 并发）
3）标记结束(Mark Termination，需 STW)，关闭写屏障。
4）清理(Sweeping, 并发)
```



#### GC 实现原理

```
在应用程序中会使用到两种内存，分别为堆（Heap）和栈（Stack），GC负责回收堆内存，而不负责回收栈中的内存
常用GC算法
1.引用计数：python,swift,php
2.分代收集：Java
3.标记清除：GO 三色标记法+混合屏障 停顿时间在0.5ms左右
```

#### 三色并发标记法，不使用stw，会有什么问题

```
1. 想要实现三色并发标记的协程和用户业务程序协程，并发的执行。有可能在执行三色标记的过程中，用户程序改变了变量间的引用关系，进而导致发生一些问题：
原本应该被垃圾回收的对象，被错误标记为了存活。
有一个对象已经被标记为了黑色，但是用户程序更改了指针，使得这个对象不再被引用，按道理，此时该对象应该被标记为白色，被回收的，但是这种情况下，由于是黑色，所以不会被回收。这种情况下，也不用担心，最多就是下次GC，对这个对象进行回收。

2. 原本应该存活的对象，被标记为了死亡。
有一个灰色的对象指向一个白色的对象，然后又有一个黑色的对象也指向了这个白色的对象，然后灰色对象断开了对白色对象的引用。此时按这种状态执行下去，由于灰色对象断开了对白色对象的引用，白色对象不再被标记为黑色，黑色对象引用了白色对象，但是黑色对象有已经被扫描过了，所以不会再去遍历白色对象了，也导致白色对象不会被标记为黑色，由于一直是白色，从而会被回收掉。这种情况，是非常严重的错误，对象直接丢失了。我们必须要避免这种情况。
为了解决上述问题，可以通过添加stw的方式，但是这会严重的影响GC性能。所以就引入下面的技术。
```

#### 屏障技术

```
垃圾收集中的屏障技术更像是一个钩子方法，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码。
比如说，程序要发生内存改动了，会先去调用内存屏障这个hook函数，对此次内存操作进行检查判断，看看接下来应该执行什么样子的操作。
```

#### 强弱三色不变式

```
强三色不变式：不允许黑色对象引用白色对象
弱三色不变式：黑色对象可以引用白色对象，但是白色对象的上游对象必须存在灰色对象
强弱三色式满足其中之一，即可解决上述问题。
```

#### 插入写屏障

```
具体操作：当A对象引用B对象的时候，B对象被标记为灰色
这样能满足强三色不变式，就不会存在黑色对象引用白色对象的情况存在了

进行垃圾回收的位置有堆和栈两种，由于栈空间小，又要求速度快，因为函数调用会造成频繁的入栈出栈，所以插入屏障不在栈空间使用，仅仅在堆空间使用。
那么栈空间该如何回收呢？在GC扫描完之后，不会直接删除白色对象，而是对栈空间加stw保护，再对栈空间重新进行三色标记扫描。
```

#### 删除写屏障

```
具体操作：前提：在起始的时候，会启动stw，把整个根部扫描一遍，将根部置为黑色，下一级为灰色，保证所有可达对象都在灰色对象的保护之下。如果从灰色对象和白色对象删除白色指针时，会将被删除的白色对象被标记为灰色。
满足弱三色不变式，保证了白色对象前面必有灰色对象。

存在的问题：回收精度会降低，一个对象即使被删除了最后一个指向它的指针，也依旧可以活过这一轮，在下一轮GC中被清理掉。
```

#### Go V1.8混合写屏障

```
针对插入写屏障和删除写屏障得短板：

插入写屏障：结束的时候，需要STW重新扫描栈。
删除写屏障：回收精度低，且GC开始的时候，需要STW扫描堆栈来记录初始快照，保护所有存活对象。
v1.8采用混合写屏障机制，避免了重新扫描的过程，极大减少了STW的时间。

混合写屏障规则
GC开始的时候，将栈上的对象全部扫描并标记为黑色；
GC期间，任何栈上创建的新对象，均为黑色；
被删除的堆对象标记为灰色；
被引用的堆对象标记为灰色；

需要注意的是：
① 混合写屏障，并不是不需要STW：混合写屏障是去除整体的STW 的一个改进，转而并发一个一个栈处理的方式(每个栈单独暂停)，从而消除了整机 STW 的影响，带来了吞吐的提升。
② 栈上不会触发写屏障，它只需要满足前两条即可。
③ 如果发生了栈对象引用了堆对象，是不会触发写屏障的，这个被引用的堆对象仍旧是白色。

```

#### 触发GC的条件

```
申请内存发出GC：
申请微对象（<16B）和小对象(16B-32KB)的时候，如果当前线程内存管理单元不存在空闲时；
申请大对象（>32KB），也会尝试触发GC。
后台定时检查触发GC：
两分钟一次。
手动触发GC：
用户程序会通过runtime.GC函数在程序运行期间主动通知运行时执行。

```



#### GC 如何调优

```
1.控制内存分配的速度，限制 Goroutine 的数量，提高赋值器 mutator 的 CPU 利用率（降低GC的CPU利用率）
2.少量使用+连接string
3.slice提前分配足够的内存来降低扩容带来的拷贝
4.避免map key对象过多，导致扫描时间增加
5.变量复用，减少对象分配，例如使用 sync.Pool 来复用需要频繁创建临时对象、使用全局变量等
6.增大 GOGC 的值，降低 GC 的运行频率 (不太用这个)
```

#### 如何查看 GC 信息

```
1. GODEBUG='gctrace=1' go run main.go
2. go tool trace trace.out
3. debug.ReadGCStats
4. runtime.ReadMemStats

```

#### Go 如何排查数据竞争问题？

```
go run -race main.go
```

### defer

#### defer作用

```
defer为延迟函数，为防止开发人员，在函数退出的时候，忘记释放资源或者执行某些收尾工作；比如，解锁、关闭文件、异常捕获等操作；
```

#### defer作用域

```
defer 作用域在当前函数和方法返回之前被调用
```

#### defer 执行顺序

```
defer 的执行顺序是采用栈（stack）的方式。当你使用多个 defer 语句时，它们会按照后进先出（LIFO）的顺序执行。在一个函数生命周期内，优先调用后面的 defer 
```

#### defer、return 谁先执行

```
return 比 defer 先执行
```

#### defer 影响主函数的具名返回值

```go
//上面讲了 return 比 defer 先执行。
//当主函数有返回值，且返回值有没有名字没有关系，defer 所作用的函数，即 defer 可能会影响主函数返回值。

func main() {
    fmt.Println(deferFuncReturn())
}

func deferFuncReturn() (j int) { // t初始化0且作用域为该函数全域
    i := 1
    defer func() {
       j++
    }()
    return i
}

$ go run main.go
2

// 结论：当主函数有返回值 ，会在函数初始化时赋值为 0，且其作用域为该函数全域，defer 会影响到该返回值。
```

#### defer 遇见 panic

```
① defer遇见panic，但是并不捕获异常的情况
和return一样，只不过panic前面的defer执行完之后，跳出函数，直接报异常
② defer遇见panic，并捕获异常
和上述不同的是，当运行的defer中捕获异常，并恢复之后，跳出函数，不会报异常，会继续执行。
但是需要注意的是，在发生panic的函数内，panic之后的程序都不会被执行。
```

#### defer 中含有 panic

```go
// panic 仅会被最后一个 revover 捕获
func main()  {
    defer func() {
       if err := recover(); err != nil{
           fmt.Println("err:", err)
       }else {
           fmt.Println("fatal")
       }
    }()

    defer func() {
        panic("defer panic2")
    }()

    panic("panic1")
}
$ go run main.go
err: defer panic2

// 在上面例子中，panic("panic1") 先 触发 defer 强制出栈，第一个执行触发 panic("defer panic2)" 异常，此时会覆盖前一个异常 panic，最后继续执行 defer， 最终被 recover() 捕获住。

```

###  interface相关问题

#### 介绍一下interface

```
① interface是go语言的一种类型。
② 它是一个方法的集合，但是并没有方法的实现，也没有数据字段。
③ 大白话讲就是，拿函数传参来说，如果该函数的形参是一个接口，无论实参是什么类型，只要实现了形参接口中的全部的方法，就能作为实参传入。也就是说，在函数内部，只在乎传入的实参有没有实现形参接口的全部方法，只要实现了，就能传入，没实现，就不能。
④ go语言是静态语言，在编译阶段就能检测出赋值给接口的值，有没有实现该接口全部的方法；而python动态语言，需要运行期间才能检测出来。
```

#### interface底层原理

```go
iface 和 eface 都是 Go 中描述接口的底层结构体，区别在于 iface 描述的接口包含方法，而 eface 则是不包含任何方法的空接口：interface{}。
从源码层面看一下：
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab
    hash   uint32 // copy of _type.hash. Used for type switches.
    bad    bool   // type does not implement interface
    inhash bool   // has this itab been added to hash?
    unused [2]byte
    fun    [1]uintptr // variable sized
}
iface 内部维护两个指针，tab 指向一个 itab 实体， 它表示接口的类型以及赋给这个接口的实体类型。data 则指向接口具体的值，一般而言是一个指向堆内存的指针。
再来仔细看一下 itab 结构体：_type 字段描述了实体的类型，包括内存对齐方式，大小等；inter 字段则描述了接口的类型。fun 字段放置和接口方法对应的具体数据类型的方法地址，实现接口调用方法的动态分派，一般在每次给接口赋值发生转换时会更新此表，或者直接拿缓存的 itab。

再看一下 interfacetype 类型，它描述的是接口的类型：
type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod
}
```



#### 值接收者和指针接收者（值调用者和指针调用者）

```
① 给用户自定义的类型添加新的方法的时候，与函数的区别是，需要给函数前添加一个接收者。这个接受者可以是自定义类型的值类型，也可以是自定义类型的指针类型。
② 在调用方法的时候，值类型既可以调用值接收者的方法，也可以调用指针接收者的方法；指针类型既可以调用指针接收者的方法，也可以调用值接收者的方法。
③ 在方法内部，如果对接收者进行了修改（例如对某一字段的值加一），无论是值类型调用还是指针类型调用，只有当接收者的类型为指针类型的时候，才会影响到接收者。（值类型的接收者，都是以“副本”的方式调用）
④ 对于自定义类型实现接口的方法的时候，需要注意了又，直接看表格：
```

![在这里插入图片描述](go面.assets/368a1f687a10429e8456206df45dc1a1.png)

#### 接口的类型检查

```go
① 断言
<目标类型的值>，<布尔参数> := <表达式>.( 目标类型 ) // 安全类型断言
<目标类型的值> := <表达式>.( 目标类型 )　　//非安全类型断言
type Student struct {
	Name string
	Age  int
}

func main() {
	stu := &Student{
		Name: "小有",
		Age:  22,
	}

	var i interface{} = stu
	s1 := i.(*Student) //断言成功，s1为*Student类型   不安全断言
	fmt.Println(s1)

	s2, ok := i.(Student) //断言失败，ok为false     安全型断言
	if ok {
		fmt.Println("success:",s2)
	}
	fmt.Println("failed:",s2)
}

② 如果接口类型可能有多种情况的话，采用Type Switch 方法。
func typeCheck(v interface{}){
	
//	switch v.(type) {       //只用判断类型，不需要值
	switch msg := v.(type) {   //值和判断类型都需要
		case int :
			...
		case string:
			...
		case Student:
			...
		case *Student:
			...
		default:
			...
	}
	
}

```

#### 空接口

```
空interface(interface{})不包含任何的method。
正因为如此，所有的类型都实现了空interface。
空interface对于描述起不到任何的作用(因为它不包含任何的method），但是空interface在我们需要存储任意类型的数值的时候相当有用，因为它可以存储任意类型的数值。
```

### 反射的相关问题

#### 什么是反射

```
反射是指计算机程序在运行时（Run time）可以访问、检测和修改它本身状态或行为的一种能力
```

#### go如何实现反射

```go
反射的三大定律
第一条是最基本的：反射是一种检测存储在 interface 中的类型和值机制。这可以通过 TypeOf 函数和 ValueOf 函数得到。
第二条实际上和第一条是相反的机制，它将 ValueOf 的返回值通过 Interface() 函数反向转变成 interface 变量。
前两条就是说 接口型变量 和 反射类型对象 可以相互转化，反射类型对象实际上就是指的前面说的 reflect.Type 和 reflect.Value。
第三条不太好懂：如果需要操作一个反射变量，那么它必须是可设置的。反射变量可设置的本质是它存储了原变量本身，这样对反射变量的操作，就会反映到原变量本身；反之，如果反射变量不能代表原变量，那么操作了反射变量，不会对原变量产生任何影响，这会给使用者带来疑惑。所以第二种情况在语言层面是不被允许的。
举一个经典例子：
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
执行上面的代码会产生 panic，原因是反射变量 v 不能代表 x 本身，为什么？因为调用 reflect.ValueOf(x) 这一行代码的时候，传入的参数在函数内部只是一个拷贝，是值传递，所以 v 代表的只是 x 的一个拷贝，因此对 v 进行操作是被禁止的。
可设置是反射变量 Value 的一个性质，但不是所有的 Value 都是可被设置的。
就像在一般的函数里那样，当我们想改变传入的变量时，使用指针就可以解决了。
var x float64 = 3.4
p := reflect.ValueOf(&x)
fmt.Println("type of p:", p.Type()) // type of p: *float64
fmt.Println("settability of p:", p.CanSet()) // settability of p: false

p 还不是代表 x，p.Elem() 才真正代表 x，这样就可以真正操作 x 了：
v := p.Elem()
v.SetFloat(7.1)
fmt.Println(v.Interface()) // 7.1
fmt.Println(x) // 7.1
关于第三条，记住一句话：如果想要操作原变量，反射变量 Value 必须要 hold 住原变量的地址才行。

反射的基本函数
reflect 包里定义了一个接口和一个结构体，即 reflect.Type 和 reflect.Value，它们提供很多函数来获取存储在接口里的类型信息。
reflect.Type 主要提供关于类型相关的信息，所以它和 _type 关联比较紧密；reflect.Value 则结合 _type 和 data 两者，因此程序员可以获取甚至改变类型的值。
reflect 包中提供了两个基础的关于反射的函数来获取上述的接口和结构体：
func TypeOf(i interface{}) Type 
func ValueOf(i interface{}) Value
TypeOf 函数用来提取一个接口中值的类型信息。由于它的输入参数是一个空的 interface{}，调用此函数时，实参会先被转化为 interface{}类型。这样，实参的类型信息、方法集、值信息都存储到 interface{} 变量里了。
看下源码：
func TypeOf(i interface{}) Type {
    eface := *(*emptyInterface)(unsafe.Pointer(&i))
    return toType(eface.typ)
}
这里的 emptyInterface 和上面提到的 eface 是一回事（字段名略有差异，字段是相同的），并且在不同的源码包：前者在 reflect 包，后者在 runtime 包。 eface.typ 就是动态类型。
type emptyInterface struct {
    typ  *rtype
    word unsafe.Pointer
}

至于 toType 函数，只是做了一个类型转换：
func toType(t *rtype) Type {
    if t == nil {
        return nil
    }
    return t
}

注意，返回值 Type 实际上是一个接口，定义了很多方法，用来获取类型相关的各种信息，而 *rtype 实现了 Type 接口。
type Type interface {
    // 所有的类型都可以调用下面这些函数

    // 此类型的变量对齐后所占用的字节数
    Align() int

    // 如果是 struct 的字段，对齐后占用的字节数
    FieldAlign() int

    // 返回类型方法集里的第 `i` (传入的参数)个方法
    Method(int) Method

    // 通过名称获取方法
    MethodByName(string) (Method, bool)

    // 获取类型方法集里导出的方法个数
    NumMethod() int

    // 类型名称
    Name() string

    // 返回类型所在的路径，如：encoding/base64
    PkgPath() string

    // 返回类型的大小，和 unsafe.Sizeof 功能类似
    Size() uintptr

    // 返回类型的字符串表示形式
    String() string

    // 返回类型的类型值
    Kind() Kind

    // 类型是否实现了接口 u
    Implements(u Type) bool

    // 是否可以赋值给 u
    AssignableTo(u Type) bool

    // 是否可以类型转换成 u
    ConvertibleTo(u Type) bool

    // 类型是否可以比较
    Comparable() bool

    // 下面这些函数只有特定类型可以调用
    // 如：Key, Elem 两个方法就只能是 Map 类型才能调用

    // 类型所占据的位数
    Bits() int

    // 返回通道的方向，只能是 chan 类型调用
    ChanDir() ChanDir

    // 返回类型是否是可变参数，只能是 func 类型调用
    // 比如 t 是类型 func(x int, y ... float64)
    // 那么 t.IsVariadic() == true
    IsVariadic() bool

    // 返回内部子元素类型，只能由类型 Array, Chan, Map, Ptr, or Slice 调用
    Elem() Type

    // 返回结构体类型的第 i 个字段，只能是结构体类型调用
    // 如果 i 超过了总字段数，就会 panic
    Field(i int) StructField

    // 返回嵌套的结构体的字段
    FieldByIndex(index []int) StructField

    // 通过字段名称获取字段
    FieldByName(name string) (StructField, bool)

    // FieldByNameFunc returns the struct field with a name
    // 返回名称符合 func 函数的字段
    FieldByNameFunc(match func(string) bool) (StructField, bool)

    // 获取函数类型的第 i 个参数的类型
    In(i int) Type

    // 返回 map 的 key 类型，只能由类型 map 调用
    Key() Type

    // 返回 Array 的长度，只能由类型 Array 调用
    Len() int

    // 返回类型字段的数量，只能由类型 Struct 调用
    NumField() int

    // 返回函数类型的输入参数个数
    NumIn() int

    // 返回函数类型的返回值个数
    NumOut() int

    // 返回函数类型的第 i 个值的类型
    Out(i int) Type

    // 返回类型结构体的相同部分
    common() *rtype

    // 返回类型结构体的不同部分
    uncommon() *uncommonType
}

讲完了 TypeOf 函数，再来看一下 ValueOf 函数。返回值 reflect.Value 表示 interface{} 里存储的实际变量，它能提供实际变量的各种信息。相关的方法常常是需要结合类型信息和值信息。例如，如果要提取一个结构体的字段信息，那就需要用到 _type (具体到这里是指 structType) 类型持有的关于结构体的字段信息、偏移信息，以及 *data 所指向的内容 —— 结构体的实际值。
源码如下：
func ValueOf(i interface{}) Value {
    if i == nil {
        return Value{}
    }

   // ……
    return unpackEface(i)
}

// 分解 eface
func unpackEface(i interface{}) Value {
    e := (*emptyInterface)(unsafe.Pointer(&i))

    t := e.typ
    if t == nil {
        return Value{}
    }

    f := flag(t.Kind())
    if ifaceIndir(t) {
        f |= flagIndir
    }
    return Value{t, e.word, f}
}
从源码看，比较简单：将先将 i 转换成 *emptyInterface 类型， 再将它的 typ 字段和 word 字段以及一个标志位字段组装成一个 Value 结构体，而这就是 ValueOf 函数的返回值，它包含类型结构体指针、真实数据的地址、标志位。
Value 结构体定义了很多方法，通过这些方法可以直接操作 Value 字段 ptr 所指向的实际数据：
	// 设置切片的 len 字段，如果类型不是切片，就会panic
 func (v Value) SetLen(n int)

 // 设置切片的 cap 字段
 func (v Value) SetCap(n int)

 // 设置字典的 kv
 func (v Value) SetMapIndex(key, val Value)

 // 返回切片、字符串、数组的索引 i 处的值
 func (v Value) Index(i int) Value

 // 根据名称获取结构体的内部字段值
 func (v Value) FieldByName(name string) Value

Value 字段还有很多其他的方法。例如：
// 用来获取 int 类型的值
func (v Value) Int() int64

// 用来获取结构体字段（成员）数量
func (v Value) NumField() int

// 尝试向通道发送数据（不会阻塞）
func (v Value) TrySend(x reflect.Value) bool

// 通过参数列表 in 调用 v 值所代表的函数（或方法
func (v Value) Call(in []Value) (r []Value) 

// 调用变参长度可变的函数
func (v Value) CallSlice(in []Value) []Value
```

#### Go的反射包怎么找到对应的方法

```
t := reflect.TypeOf(o) //获取类型
m1 := t.Method(0) //获取第几个方法
m2 := t.MethodByName(" funcName ") //根据方法名字获取方法
```

#### DeepEqual 的作用及原理

```go
如何比较两个对象完全相同
Go 语言中提供了一个函数可以完成此项功能：
func DeepEqual(x, y interface{}) bool
DeepEqual 函数的参数是两个 interface，实际上也就是可以输入任意类型，输出 true 或者 flase 表示输入的两个变量是否是“深度”相等。

如果是不同的类型，即使是底层类型相同，相应的值也相同，那么两者也不是“深度”相等。
type MyInt int
type YourInt int

func main() {
    m := MyInt(1)
    y := YourInt(1)

    fmt.Println(reflect.DeepEqual(m, y)) // false
}
上面的代码中，m, y 底层都是 int，而且值都是 1，但是两者静态类型不同，前者是 MyInt，后者是 YourInt，因此两者不是“深度”相等。

在源码里，有对 DeepEqual 函数的非常清楚地注释，列举了不同类型，DeepEqual 的比较情形，这里做一个总结：
类型												深度相等情形
Array										  相同索引处的元素“深度”相等
Struct										相应字段，包含导出和不导出，“深度”相等
Func											只有两者都是 nil 时
Interface									两者存储的具体值“深度”相等
Map												1、都为 nil；2、非空、长度相等，指向同一个 map 实体对象，
														或者相应的 key 指向的 value “深度”相等
Pointer										1、使用 == 比较的结果相等；2、指向的实体“深度”相等
Slice											1、都为 nil；2、非空、长度相等，首元素指向同一个底层数组的相同元素，
														即 &x[0] == &y[0] 或者 相同索引处的元素“深度”相等
numbers,bools,strings,channels		使用 == 比较的结果为真


```



### init函数

```
在包初始化的时候会调用init函数，不能被显示调用

适用场景
	初始化变量
	检查或者修复程序状态
	注册任务
	仅仅需要执行一次的情况
	
特征
	同一个包，可以有多个init函数
	包中的每个源文件中可以有多个init函数
	
执行顺序（可以自己实验验证）
	同一个源文件中的init函数，是按照先后顺序执行的（且在全局变量初始化之后）
	同一个包中的源文件的init函数，是按照源文件的字母顺序执行的
	不同包的init函数，按照包导入的依赖关系决定先后顺序
```

### sync.waitGroup相关

```
一个waitGroup对象，可以实现同一时间启动n个协程，并发执行，等n个协程全部执行结束后，在继续往下执行的一个功能。
通过Add()方法设置启动了多少个协程，在每一个协程结束的时候调用Done()方法，计数减一，同时使用wait()方法阻塞主协程，等待全部的协程执行结束。
```

#### 为什么要用sync.waitGroup

```
我们在日常开发中为了提高接口响应时间，有一些场景需要在多个goroutine中做一些互不影响的业务，这样可以节省不少时间，但是需要协调多个goroutine,但是我们每次使用都要保证主Goroutine最后从通道接收的次数需要与之前其他的Goroutine发送元素的次数相同，在这种场景下我们就可以选用sync.WaitGroup来帮助我们实现同步。
```



### 问题

#### 结构体不加 tag 可以转 json 字符串吗

```
如果变量首字母小写，则为 private。无论如何不能转，因为取不到反射信息。
如果变量首字母大写，则为 public。
不加 tag，可以正常转为 json 里的字段，json 内字段名跟结构体内字段原名一致。
加了 tag，从 struct 转 json 的时候，json 的字段名就是 tag 里的字段名，原字段名已经没用。
```

#### 字符串的小问题

```
①可以用==比较
②不可以通过下标的方式改变某个字符，字符串是只读的
③不能和nil比较
```

#### Go 支持什么形式的类型转换？

```
Go支持显示类型的转换，以满足严格的类型要求
```

#### 空结构体的作用

```
不包含任何字段的结构体叫做空结构体 struct{}
定义：
var et struct{}
et := struct{}{}
type ets struct {} / et := ets{} / var et ets
特性：

所有的空结构体的地址都是同一地址，都是zerobase的地址，且大小为0
使用场景：
用于保存不重复的元素的集合，Go的map的key是不允许重复的，用空结构体作为value，不占用额外空间。
用于channel中信号传输，当我们不在乎传输的信号的内容的时候，只是说只要用信号过来，通知到了就行的时候，用空结构体作为channel的类型
作为方法的接收者，然后该空结构体内嵌到其他结构体，实现继承
```

#### 单引号，双引号，反引号的区别？

```
单引号，表示byte或者rune类型，对应uint8和int32类型；默认直接赋值的话是rune类型。
双引号，字符串类型，不允许修改。实际上是字符数组，可以用下标索引其中的某个字节。
反引号，表示字符串字面量，反引号中的字符不支持任何转义，写什么就是什么。
```

#### 如何停止一个 Goroutine？

```
①for - select方法，采用通道，通知协程退出
②采用context包
```

#### Printf()，Sprintf()，FprintF() 都是格式化输出，有什么不同？

````
Printf()是标准输出，一般用于打印。
Sprintf()把格式化字符串输出到字符串，并返回
FprintF()把格式化字符串输出到实现了io.witer方法的类型，比如文件
````

#### golang 中 make 和 new 的区别？

```
共同点：都会分配内存空间（堆上）
不同点：
①作用变量不同，new可以为任意类型分配内存；但是make只能给切片、map、chan分配内存
②返回类型不同，new返回的是指向变量的指针；make返回的是上边三种变量类型本身
③new对分配的内存清零；make会根据你的设定进行初始化，比如在设置长度、容量的时候
```

#### for-range切片的时候，它的地址会发生变化么？

```
在for a,b := range slice的遍历中，a和b内存中的地址只有一份，每次循环遍历的时候，都会对其进行覆盖，其地址始终不会改变。对于切片遍历的话，b是复制的切片中的元素，改变b，并不会影响切片中的元素。
```

#### context 使用场景和用途？

```
context的主要作用：协调多个 groutine 中的代码执行“取消”操作，并且可以存储键值对。最重要的是它是并发安全的。
① 可以存储键值对，供上下文（协程间）读取【建议不要使用】
② 优雅的主动取消协程（Cancel）。主动取消子协程运行，用不到子协程了，回收资源。比如一个http请求，客户端突然断开了，就直接cancel，停止后续的操作；
③ 超时退出协程（Timeout），比如如果三秒之内没有执行结束，直接退出该协程；
④ 截止时间退出协程（Deadline），如果一个业务，2点到4点为业务活动期，4点截止结束任务（协程）
```

#### 常量计数器iota

```
用来干啥的：
go语言中用常量定义代替枚举类型，这时候就用到了iota常量计数器，具有自增的特点，可以简化有关于数字增长的常量的定义
再说一下特点：
① iota只能出现在const代码块中
② 不同const代码块中的iota互不影响
③ 从第一行开始算，ioat出现在第几行，它的值就是第几行减一 【所有注释行和空白行忽略；_代表一行，不能忽略；这一行即使没有iota也算一行。】
④ 没有表达式的常量定义复用上一行的表达式
```

#### 介绍下rune类型

```
rune是int32的别名，等同于int32，常用来处理unicode或utf-8字符，用来区分字符值和整数值
这里和byte进行对比，byte是uint8，常用来处理ascii字符
那么有什么不同呢？举个例子
```

```go
var str = "hello 世界"
 
	//golang中string底层是通过byte数组实现的，直接求len 实际是在按字节长度计算  所以一个汉字占3个字节算了3个长度
	fmt.Println("len(str):", len(str))
 
	//以下两种都可以得到str的字符串长度
 
	//golang中的unicode/utf8包提供了用utf-8获取长度的方法
	fmt.Println("RuneCountInString:", utf8.RuneCountInString(str))
 
	//通过rune类型处理unicode字符
	fmt.Println("rune:", len([]rune(str)))

输出为：12 8 8
golang中string底层是通过byte数组实现的。中文字符在unicode下占2个字节，在utf-8编码下占3个字节，而golang默认编码正好是utf-8。
所以len计算的时候，一个中文字符占用3个字节。
而转换为rune计算，就刚好能得到字符的实际个数，比如我们想取出‘界’，rune的方式就很友好

fmt.Println(string([]rune(str)[7:]))  //就能取出‘界’
```

#### go语言如何实现面对对象编程

```
面对对象编程的三个基本特征：封装、继承和多态
go通过结构体实现封装和继承，通过接口实现多态

封装：在结构体中，字段为成员变量（c++中的成员变量），字段名大写开头为可以导出，也就是外部可以访问；字段名为小写开头为不可以导出，也就是外部不可以访问（也就是说，在本package中，这两种情况是没有区别的，如果这个结构体在其他的package中使用的话，小写开头的不能直接被访问）；将结构体的类型作为函数的接收器，来为这个结构体写方法，也就是c++中的成员函数。

继承：通过在子结构体中，添加父结构体作为成员变量，以这种组合的方式来实现继承，这样子结构体声明的变量也就能访问父结构体中的变量和方法了。

多态：多态是通过接口实现的，只要结构体实现了该接口的方法，那它们就能赋值给该接口，从而可以根据不同的赋值变量，而去执行不同的方法。
```

#### go的结构体能不能比较？

```
结构体中含有不能比较的类型时，不能比较；
声明两个比较值的结构体的名字不同，即使字段名、类型、顺序相同，也不能比较（强转类型可以比较），说白了，必须用同一个结构体类型声明的值，才能比较；
```

#### 进程 线程 协程 的理解

```
进程：一个正在执行程序。比如你启动main函数，，跑起来之后，系统分配了各种资源，以及独立的内存空间，就是一个进程。进程因为具有独立的内存空间，稳定安全。但是进程间切换开销太大。

线程：轻量级的进程，是cpu调度的最小单位，一个进程至少包含一个主线程，或者多个线程。多线程之间共享进程的资源。共享内存空间，因此线程间切换开销小（进程内的线程切换仅仅只涉及内核态，进程间的线程切换涉及内核态与用户态的转换）。但是线程是共享内存，在读取数据的时候需要采用互斥或者顺序的手段，保证数据的一致性。

协程：是一种用户态的轻量级线程。协程的调度完全由用户态调度，协程是不被操作系统所管理的，而是被用户管理。协程的切换可以防止用户态向内核态切换。
```

#### Go的调度方式？

```
主动调度：自己主动放弃执行，通过runtime.Gosched()实现。取消G与M之间的绑定关系，将G放入到全局运行对列中去。
被动调度：发生锁、sleep、channel等阻塞的情况。取消P与M之间的绑定关系，P再去找一个空闲的M。
抢占调度：① 抢占执行时间过长的G，发生在函数调用的时候。若该G执行时间超过10ms，发生抢占；② 抢占系统调用。当G在系统调用中超过10ms时，发生抢占；
对于抢占调度，1.14之后，当cpu密集型任务执行时，可能没有发生函数调用，也没有系统调用，这样就没法进行抢占。 1.14是单独起一个线程对运行的协程进行监控，超过10ms的则进行抢占。
```

#### 调度器有哪些设计策略？

```
复用线程：避免重复的创建、销毁线程，而是对线程的复用（work stealing机制和hand off机制）
抢占机制：一个协程占用cpu的时长是有时间限制的，当该协程运行超时之后，会被其他协程抢占，防止其他协程被饿死，
```

#### 两个 nil 可能不相等吗？

```go
可能。
接口(interface) 是对非接口值(例如指针，struct等)的封装，内部实现包含 2 个字段，类型 T 和 值 V。一个接口等于 nil，当且仅当 T 和 V 处于 unset 状态（T=nil，V is unset）。
两个接口值比较时，会先比较 T，再比较 V。
接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较。
func main() {
	var p *int = nil
	var i interface{} = p
	fmt.Println(i == p) // true
	fmt.Println(p == nil) // true
	fmt.Println(i == nil) // false
}
上面这个例子中，将一个 nil 非接口值 p 赋值给接口 i，此时，i 的内部字段为(T=*int, V=nil)，i 与 p 作比较时，将 p 转换为接口后再比较，因此 i == p，p 与 nil 比较，直接比较值，所以 p == nil。

但是当 i 与 nil 比较时，会将 nil 转换为接口 (T=nil, V=nil)，与i (T=*int, V=nil) 不相等，因此 i != nil。因此 V 为 nil ，但 T 不为 nil 的接口不等于 nil。


```

#### 函数返回局部变量的指针是否安全？

```
这在 Go 中是安全的，Go 编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上。
```

#### 服务器之间如何共享 session

```
1.通过数据库mysql共享session

  a.采用一台专门的mysql服务器来存储所有的session信息。

  	用户访问随机的web服务器时，会去这个专门的数据库服务器check一下session的情况，以达到session同步的目的。 

  	缺点就是：依懒性太强，mysql服务器无法工作，影响整个系统；

  b.将存放session的数据表与业务的数据表放在同一个库。如果mysql做了主从，需要每一个库都需要存在这个表，并且需要数据实时同步。

    缺点：用数据库来同步session，会加大数据库的负担，数据库本来就是容易产生瓶颈的地方，如果把session还放到数据库里面，无疑是雪上加霜。上面的二种方法,第一点方法较好，把放session的表独立开来，减轻了真正数据库的负担 。但是session一般的查询频率较高，放在数据库中查询性能也不是很好，不推荐使用这种方式。

2.通过cookie共享session

   把用户访问页面产生的session放到cookie里面，就是以cookie为中转站。

   当访问服务器A时，登录成功之后将产生的session信息存放在cookie中；当访问请求分配到服务器B时，服务器B先判断服务器有没有这个session，如果没有，在去看看客户端的cookie里面有没有这个session，如果cookie里面有，就把cookie里面的sessoin同步到web服务器B，这样就可以实现session的同步了。 

   缺点：cookie的安全性不高，容易伪造、客户端禁止使用cookie等都可能造成无法共享session。

3.通过服务器之间的数据同步session

　　使用一台作为用户的登录服务器，当用户登录成功之后，会将session写到当前服务器上，我们通过脚本或者守护进程将session同步到其他服务器上，这时当用户跳转到其他服务器，session一致，也就不用再次登录。

　　缺陷：速度慢，同步session有延迟性，可能导致跳转服务器之后，session未同步。而且单向同步时，登录服务器宕机，整个系统都不能正常运行。
　　
4.通过NFS共享Session

　　选择一台公共的NFS服务器（Network File Server）做共享服务器，所有的Web服务器登陆的时候把session数据写到这台服务器上，那么所有的session数据其实都是保存在这台NFS服务器上的，不论用户访问那太Web服务器，都要来这台服务器获取session数据，那么就能够实现共享session数据了。

　　缺点：依赖性太强，如果NFS服务器down掉了，那么大家都无法工作了，当然，可以考虑多台NFS服务器同步的形式。
　　
5.通过memcache同步session

　　memcache可以做分布式，如果没有这功能，他也不能用来做session同步。他可以把web服务器中的内存组合起来，成为一个"内存池"，不管是哪个服务器产生的sessoin都可以放到这个"内存池"中，其他的都可以使用。 

　　优点：以这种方式来同步session，不会加大数据库的负担，并且安全性比用cookie大大的提高，把session放到内存里面，比从文件中读取要快很多。 

　　缺点：memcache把内存分成很多种规格的存储块，有块就有大小，这种方式也就决定了，memcache不能完全利用内存，会产生内存碎片，如果存储块不足，还会产生内存溢出。 
　　
6.通过redis共享session

　　redis与memcache一样，都是将数据放在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。
```

#### PSR 规范

```
psr-1是基本代码规范
psr-2是代码风格规范
psr-3是日志接口规范
psr-4是为了解决自动加载
psr-6是缓存接口规范
psr-7是HTTP消息接口规范
```



## Mysql

### 基础

```
1. 如何区分FLOAT和DOUBLE？
	1）浮点数以 8 位精度存储在 FLOAT 中，并且有四个字节。
	2）浮点数存储在 DOUBLE 中，精度为 18 位，有八个字节。

2. 区分 CHAR_LENGTH 和 LENGTH？
	CHAR_LENGTH 是字符数，而 LENGTH 是字节数。Latin 字符的这两个数据是相同的，但是对于 Unicode 和其他编码，它们是不同的。

3. CHAR 和 VARCHAR 的区别？
	CHAR 和 VARCHAR 类型在存储和检索方面有所不同
	CHAR 列长度固定为创建表时声明的长度，长度值范围是 1 到 255
	当 CHAR 值被存储时，它们被用空格填充到特定长度，检索 CHAR 值时需删除尾随空格。
```



#### 什么是聚集索引和非聚集索引

```
简单来说，聚集索引就是基于主键创建的索引，除了主键索引以外的其他索引，称为非聚集索引，也叫做二级索引。

由于在InnoDB引擎里面，一张表的数据对应的物理文件本身就是按照B+树来组织的一种索引结构，而聚集索引就是按照每张表的主键来构建一颗B+树，然后叶子节点里面存储了这个表的每一行数据记录。

所以基于InnoDB这样的特性，聚集索引并不仅仅是一种索引类型，还代表着一种数据的存储方式。

同时也意味着每个表里面必须要有一个主键，如果没有主键，InnoDB会默认选择或者添加一个隐藏列作为主键索引来存储这个表的数据行。一般情况是建议使用自增id作为主键，这样的话id本身具有连续性使得对应的数据也会按照顺序存储在磁盘上，写入性能和检索性能都很高。否则，如果使用uuid这种随机id，那么在频繁插入数据的时候，就会导致随机磁盘IO，从而导致性能较低。

需要注意的是，InnoDB里面只能存在一个聚集索引，原因很简单，如果存在多个聚集索引，那么意味着这个表里面的数据存在多个副本，造成磁盘空间的浪费，以及数据维护的困难。
```

#### 何谓事务？

```
事务是逻辑上的一组操作，要么都执行，要么都不执行。
事务都有 ACID 特性：
原子性（Atomicity） ： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
一致性（Consistency）： 执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
隔离性（Isolation）： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
持久性（Durability）： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。
```

####  不可重复读和幻读有什么区别

```
不可重复读的重点是内容修改或者记录减少比如多次读取一条记录发现其中某些记录的值被修改；
幻读的重点在于记录新增比如多次执行同一条查询语句（DQL）时，发现查到的记录增加了。
                   
幻读其实可以看作是不可重复读的一种特殊情况，单独把区分幻读的原因主要是解决幻读和不可重复读的方案不一样。
举个例子：执行 delete 和 update 操作的时候，可以直接对记录加锁，保证事务安全。而执行 insert 操作的时候，由于记录锁（Record Lock）只能锁住已经存在的记录，为了避免插入新记录，需要依赖间隙锁（Gap Lock）。也就是说执行 insert 操作的时候需要依赖 Next-Key Lock（Record Lock+Gap Lock） 进行加锁来保证不出现幻读。
```

#### 并发事务的控制方式有哪些？

```
MySQL 中并发事务的控制方式无非就两种：锁 和 MVCC。锁可以看作是悲观控制的模式，多版本并发控制（MVCC，Multiversion concurrency control）可以看作是乐观控制的模式。

锁 控制方式下会通过锁来显示控制共享资源而不是通过调度手段，MySQL 中主要是通过 读写锁 来实现并发控制。
共享锁（S 锁） ：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
排他锁（X 锁） ：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁（锁不兼容）。

读写锁可以做到读读并行，但是无法做到写读、写写并行。另外，根据根据锁粒度的不同，又被分为 表级锁(table-level locking) 和 行级锁(row-level locking) 。InnoDB 不光支持表级锁，还支持行级锁，默认为行级锁。行级锁的粒度更小，仅对相关的记录上锁即可（对一行或者多行记录加锁），所以对于并发写入操作来说， InnoDB 的性能更高。不论是表级锁还是行级锁，都存在共享锁（Share Lock，S 锁）和排他锁（Exclusive Lock，X 锁）这两类。

MVCC 是多版本并发控制方法，即对一份数据会存储多个版本，通过事务的可见性来保证事务能看到自己应该看到的版本。通常会有一个全局的版本分配器来为每一行数据设置版本号，版本号是唯一的。
MVCC 在 MySQL 中实现所依赖的手段主要是: 隐藏字段、read view、undo log。
undo log : undo log 用于记录某行数据的多个版本的数据。
read view 和 隐藏字段 : 用来判断当前版本数据的可见性。
```

#### 请你简单说一下Mysql的事务隔离级别

```
1. 假设有两个事务T1/T2同时在执行，T1事务有可能会读取到T2事务未提交的数据，但是未提交的事务T2可能会回滚，也就导致了T1事务读取到最终不一定存在的数据产生脏读的现象。

2. 假设有两个事务T1/T2同时执行，事务T1在不同的时刻读取同一行数据的时候结果可能不一样，从而导致不可重复读的问题。

3. 假设有两个事务T1/T2同时执行，事务T1执行范围查询或者范围修改的过程中，事务T2插入了一条属于事务T1范围内的数据并且提交了，这时候在事务T1查询发现多出来了一条数据，或者在T1事务发现这条数据没有被修改，看起来像是产生了幻觉，这种现象称为幻读。

而这三种现象在实际应用中，可能有些场景不能接受某些现象的存在，所以在SQL标准中定义了四种隔离级别，分别是：

读未提交(READ-UNCOMMITTED)，在这种隔离级别下，可能会产生脏读、不可重复读、幻读。
读已提交(READ-COMMITTED)（RC)，在这种隔离级别下，可能会产生不可重复读和幻读。
可重复读(REPEATABLE-READ)（RR），在这种隔离级别下，可能会产生幻读
串行化(SERIALIZABLE)，在这种隔离级别下，多个并行事务串行化执行，不会产生安全性问题。
这四种隔离级别里面，只有串行化解决了全部的问题，但也意味着这种隔离级别的性能是最低的。
```

#### MySQL 的隔离级别是基于锁实现的吗？

```
MySQL 的隔离级别基于锁和 MVCC 机制共同实现的。
SERIALIZABLE 隔离级别是通过锁来实现的，READ-COMMITTED 和 REPEATABLE-READ 隔离级别是基于 MVCC 实现的。不过， SERIALIZABLE 之外的其他隔离级别可能也需要用到锁机制，就比如 REPEATABLE-READ 在当前读情况下需要使用加锁读来保证不会出现幻读。
```

###  MySQL 锁

#### 表级锁和行级锁了解吗？有什么区别？

```
MyISAM 仅仅支持表级锁(table-level locking)，一锁就锁整张表，这在并发写的情况下性非常差。InnoDB 不光支持表级锁(table-level locking)，还支持行级锁(row-level locking)，默认为行级锁。
行级锁的粒度更小，仅对相关的记录上锁即可（对一行或者多行记录加锁），所以对于并发写入操作来说， InnoDB 的性能更高。

表级锁和行级锁对比 ：
表级锁： MySQL 中锁定粒度最大的一种锁（全局锁除外），是针对非索引字段加的锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。不过，触发锁冲突的概率最高，高并发下效率极低。表级锁和存储引擎无关，MyISAM 和 InnoDB 引擎都支持表级锁。
行级锁： MySQL 中锁定粒度最小的一种锁，是 针对索引字段加的锁 ，只针对当前操作的行记录进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。行级锁和存储引擎有关，是在存储引擎层面实现的。
```

####  行级锁的使用有什么注意事项？

```
InnoDB 的行锁是针对索引字段加的锁，表级锁是针对非索引字段加的锁。当我们执行 UPDATE、DELETE 语句时，如果 WHERE条件中字段没有命中唯一索引或者索引失效的话，就会导致扫描全表对表中的所有行记录进行加锁。
```

#### InnoDB 有哪几类行锁？

```
InnoDB 行锁是通过对索引数据页上的记录加锁实现的，MySQL InnoDB 支持三种行锁定方式：
记录锁（Record Lock） ：也被称为记录锁，属于单个行记录上的锁。
间隙锁（Gap Lock） ：锁定一个范围，不包括记录本身。
临键锁（Next-Key Lock） ：Record Lock+Gap Lock，锁定一个范围，包含记录本身，主要目的是为了解决幻读问题（MySQL 事务部分提到过）。记录锁只能锁住已经存在的记录，为了避免插入新记录，需要依赖间隙锁。

在 InnoDB 默认的隔离级别 REPEATABLE-READ 下，行锁默认使用的是 Next-Key Lock。但是，如果操作的索引是唯一索引或主键，InnoDB 会对 Next-Key Lock 进行优化，将其降级为 Record Lock，即仅锁住索引本身，而不是范围。
```

#### 什么是 next-key lock

```
next-key 锁是索引记录上的记录锁和索引记录之前的间隙上的间隙锁的组合。
```

#### next-key lock 加锁范围是什么？

```
加锁时，会先给表添加意向锁，IX 或 IS；
加锁是如果是多个范围，是分开加了多个锁，每个范围都有锁；
主键等值查询，数据存在时，会对该主键索引的值加行锁 X,REC_NOT_GAP；
主键等值查询，数据不存在时，会对查询条件主键值所在的间隙添加间隙锁 X,GAP；

说明：
LOCK_MODE = X 是前开后闭区间；
X,GAP 是前开后开区间（间隙锁）；
X,REC_NOT_GAP 行锁。
```

#### 共享锁和排他锁呢？

```
不论是表级锁还是行级锁，都存在共享锁（Share Lock，S 锁）和排他锁（Exclusive Lock，X 锁）这两类：

共享锁（S 锁） ：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
排他锁（X 锁） ：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁（锁不兼容）。
```

#### 意向锁有什么作用？

```
如果需要用到表锁的话，如何判断表中的记录没有行锁呢，一行一行遍历肯定是不行，性能太差。我们需要用到一个叫做意向锁的东东来快速判断是否可以对某个表使用表锁。
意向锁是表级锁，共有两种：
意向共享锁（Intention Shared Lock，IS 锁）：事务有意向对表中的某些记录加共享锁（S 锁），加共享锁前必须先取得该表的 IS 锁。
意向排他锁（Intention Exclusive Lock，IX 锁）：事务有意向对表中的某些记录加排他锁（X 锁），加排他锁之前必须先取得该表的 IX 锁。

意向锁和共享锁和排它锁互斥（这里指的是表级别的共享锁和排他锁，意向锁不会与行级的共享锁和排他锁互斥）。

意向锁是有数据引擎自己维护的，用户无法手动操作意向锁，在为数据行加共享/排他锁之前，InooDB 会先获取该数据行所在在数据表的对应意向锁。
```



### MVCC的理解

```
对于MVCC的理解，我觉得可以先从数据库的三种并发场景说起：
第一种：读读
就是线程A与线程B同时在进行读操作，这种情况下不会出现任何并发问题。

第二种：读写  
就是线程A与线程B在同一时刻分别进行读和写操作。
这种情况下，可能会对数据库中的数据造成以下问题：
事物隔离性问题，
出现脏读，幻读，不可重复读的问题

第三种：写写
就是线程A与线程B同时进行写操作
这种情况下可能会存在数据更新丢失的问题。

而MVCC就是为了解决事务操作中并发安全性问题的无锁并发控制技术全称为Multi-Version Concurrency Control ，也就是多版本并发控制。它是通过数据库记录中的隐式字段，undo日志 ，Read View 来实现的。

MVCC主要解决了三个问题
第一个是：通过MVCC 可以解决读写并发阻塞问题从而提升数据并发处理能力
第二个是：MVCC 采用了乐观锁的方式实现，降低了死锁的概率
第三个是：解决了一致性读的问题也就是事务启动时根据某个条件读取到的数据，直到事务结束时，再次执行相同条件，还是读到同一份数据，不会发生变化。

而我们在使用MVCC时一般会根据业务场景来选择组合搭配乐观锁或悲观锁。
这两个组合中，MVCC用来解决读写冲突，乐观锁或者悲观锁解决写写冲突从而最大程度的提高数据库并发性能。
```

#### 当前读和快照读有什么区别？

```
1. 快照读（一致性非锁定读）就是单纯的 SELECT 语句，但不包括下面这两类 SELECT 语句：
SELECT ... FOR UPDATE
SELECT ... LOCK IN SHARE MODE

快照即记录的历史版本，每行记录可能存在多个历史版本（多版本技术）。
快照读的情况下，如果读取的记录正在执行 UPDATE/DELETE 操作，读取操作不会因此去等待记录上 X 锁的释放，而是会去读取行的一个快照。
只有在事务隔离级别 RC(读取已提交) 和 RR（可重读）下，InnoDB 才会使用一致性非锁定读：
在 RC 级别下，对于快照数据，一致性非锁定读总是读取被锁定行的最新一份快照数据。
在 RR 级别下，对于快照数据，一致性非锁定读总是读取本事务开始时的行数据版本。

快照读比较适合对于数据一致性要求不是特别高且追求极致性能的业务场景。


2. 当前读 （一致性锁定读）就是给行记录加 X 锁或 S 锁。

当前读的一些常见 SQL 语句类型如下：
# 对读的记录加一个X锁
SELECT...FOR UPDATE
# 对读的记录加一个S锁
SELECT...LOCK IN SHARE MODE
# 对修改的记录加一个X锁
INSERT...
UPDATE...
DELETE...
```

#### 日常工作中是怎么优化SQL

```
1.加索引，增加索引是一种简单高效的手段，但是需要选择合适的列，同时避免导致索引失效的操作，比如like、函数等。
2.避免返回不必要的数据列，减少返回的数据列可以增加查询的效率。 
3.根据查询分析器适当优化SQL的结构，比如是否走全表扫描、避免子查询等
4.分库分表，在单表数据量较大或者并发连接数过高的情况下，通过这种方式可以有效提升查询效率
5.读写分离，针对读多写少的场景，这样可以保证写操作的数据库承受更小的压力，也可以缓解独占锁和共享锁的竞争。
```

#### Mysql为什么使用B+Tree作为索引结构

```
首先，常规的数据库存储引擎，一般都是采用B树或者B+树来实现索引的存储。
因为B树是一种多路平衡树，用这种存储结构来存储大量数据，它的整个高度会相比二叉树来说，会矮很多。
而对于数据库来说，所有的数据必然都是存储在磁盘上的，而磁盘IO的效率实际上是很低的，特别是在随机磁盘IO的情况下效率更低。
所以树的高度能够决定磁盘IO的次数，磁盘IO次数越少，对于性能的提升就越大，这也是为什么采用B树作为索引存储结构的原因。

但是在Mysql的InnoDB存储引擎里面，它用了一种增强的B树结构，也就是B+树来作为索引和数据的存储结构。
相比较于B树结构，B+树做了几个方面的优化。
1. B+树的所有数据都存储在叶子节点，非叶子节点只存储索引。
2. 叶子节点中的数据使用双向链表的方式进行关联。

使用B+树来实现索引的原因，我认为有几个方面。
1. B+树非叶子节点不存储数据，所以每一层能够存储的索引数量会增加，意味着B+树在层高相同的情况下存储的数据量要比B树要多，使得磁盘IO次数更少。
2. 在Mysql里面，范围查询是一个比较常用的操作，而B+树的所有存储在叶子节点的数据使用了双向链表来关联，所以在查询的时候只需查两个节点进行遍历就行，而B树需要获取所有节点，所以B+树在范围查询上效率更高。
3. 在数据检索方面，由于所有的数据都存储在叶子节点，所以B+树的IO次数会更加稳定一些。
4. 因为叶子节点存储所有数据，所以B+树的全局扫描能力更强一些，因为它只需要扫描叶子节点。但是B树需要遍历整个树
5. 另外，基于B+树这样一种结构，如果采用自增的整型数据作为主键，还能更好的避免增加数据的时候，带来叶子节点分裂导致的大量运算的问题。

总结：
技术方案的选型，更多的是去解决当前场景下的特定问题，并不一定是说B+树就是最好的选择，就像MongoDB里面采用B树结构，本质上来说，其实是关系型数据库和非关系型数据库的差异。
```

#### Mysql索引的优点和缺点？

```
索引，是一种能够帮助Mysql高效从磁盘上检索数据的一种数据结构。

在Mysql中的InnoDB引擎中，采用了B+树的结构来实现索引和数据的存储

Mysql里面的索引的优点有很多
1. 通过B+树的结构来存储数据，可以大大减少数据检索时的磁盘IO次数，从而提升数据查询的性能
2. B+树索引在进行范围查找的时候，只需要找到起始节点，然后基于叶子节点的链表结构往下读取即可，查询效率较高。
3. 通过唯一索引约束，可以保证数据表中每一行数据的唯一性

当然，索引的不合理使用，也会有带来很多的缺点。
1. 数据的增加、修改、删除，需要涉及到索引的维护，当数据量较大的情况下，索引的维护会带来较大的性能开销。
2. 一个表中允许存在一个聚簇索引和多个非聚簇索引，但是索引数不能创建太多，否则造成的索引维护成本过高。
3. 创建索引的时候，需要考虑到索引字段值的分散性，如果字段的重复数据过多，创建索引反而会带来性能降低。

```

#### 索引什么时候失效？

```
1. 在索引列上做运算，比如使用函数，Mysql在生成执行计划的时候，它是根据统计信息来判断是否要使用索引的。
        而在索引列上加函数运算，导致Mysql无法识别索引列，也就不会再走索引了。
        不过从Mysql8开始，增加了函数索引可以解决这个问题。
2. 在一个由多列构成的组合索引中，需要按照最左匹配法则，也就是从索引的最左列开始顺序检索，否则不会走索引。
在组合索引中，索引的存储结构是按照索引列的顺序来存储的，因此在sql中也需要按照这个顺序才能进行逐一匹配。
否则InnoDB无法识别索引导致索引失效。
3. 当索引列存在隐式转化的时候， 比如索引列是字符串类型，但是在sql查询中没有使用引号。
那么Mysql会自动进行类型转化，从而导致索引失效
4. 在索引列使用不等于号、not查询的时候，由于索引数据的检索效率非常低，因此Mysql引擎会判断不走索引。
5. 使用like通配符匹配后缀%xxx的时候，由于这种方式不符合索引的最左匹配原则，所以也不会走索引。
但是反过来，如果通配符匹配的是前缀xxx%，符合最左匹配，也会走索引。
6.使用or连接查询的时候，or语句前后没有同时使用索引，那么索引会失效。只有or左右查询字段都是索引列的时候，才会生效。

除了这些场景以外，对于多表连接查询的场景中，连接顺序也会影响索引的使用。
不过最终是否走索引，我们可以使用explain命令来查看sql的执行计划，然后针对性的进行调优即可。

```

#### InnoDB 与MyISAM 有什么区别

```
1. 事务支持不同，InnoDB 支持事务处理，而 MyISAM 不支持。
2. 并发处理不同：InnoDB 支持行级锁，而 MyISAM 支持表级锁
3. 外键支持不同：InnoDB 支持外键约束，而 MyISAM 不支持
4. 性能上存在差异：MyISAM 的读取速度比 InnoDB 快，但是在高并发环境下，InnoDB 的性能更好。这是因为 InnoDB 支持行级锁和事务处理，而 MyISAM 不支持。
所以，如果是读多写少的情况下，使用MyISAM引擎会更合适
5.数据安全不同：InnoDB 支持崩溃恢复和数据恢复，而 MyISAM 不支持。如果 MySQL 崩溃了或者发生意外故障，InnoDB 可以通过恢复日志来恢复数据。
6. MyISAM 不支持 MVVC，而 InnoDB 支持。
```

#### 为什么 SQL 语句不要过多的 join？

```
1. 性能问题：每个 join 操作都需要对两个或多个表进行连接操作，这个操作需要消耗大量的计算资源和时间，如果 join 操作过多，会导致 SQL 的执行效率降低，从而影响整个系统的性能。
2. 可读性和维护性问题：join 操作会使 SQL 语句变得复杂，难以理解和维护，特别是当 join 操作涉及到多个表的时候，SQL 语句的复杂度会呈现指数级增长，给代码的可读性和可维护性带来挑战。

```

### mysql读写分离和分库分表常见问题

#### 什么是读写分离？

```
读写分离主要是为了将对数据库的读写操作分散到不同的数据库节点上。
这样的话，就能够小幅提升写性能，大幅提升读性能。
```

#### 如何实现读写分离？

```
不论是使用哪一种读写分离具体的实现方案，想要实现读写分离一般包含如下几步：
1. 部署多台数据库，选择其中的一台作为主数据库，其他的一台或者多台作为从数据库。
2. 保证主数据库和从数据库之间的数据是实时同步的，这个过程也就是我们常说的主从复制。
3. 系统将写请求交给主数据库处理，读请求交给从数据库处理。

落实到项目本身的话，常用的方式有两种：
1. 代理方式
我们可以在应用和数据中间加了一个代理层。应用程序所有的数据请求都交给代理层处理，代理层负责分离读写请求，将它们路由到对应的数据库中。

2. 组件方式
在这种方式中，我们可以通过引入第三方组件来帮助我们读写请求。
```

#### 主从复制原理是什么？

```
MySQL binlog(binary log 即二进制日志文件) 主要记录了 MySQL 数据库中数据的所有变化(数据库执行的所有 DDL 和 DML 语句)。因此，我们根据主库的 MySQL binlog 日志就能够将主库的数据同步到从库中。
1. 主库将数据库中数据的变化写入到 binlog
2. 从库连接主库从库会创建一个 I/O 线程向主库请求更新的 binlog
3. 主库会创建一个 binlog dump 线程来发送 binlog ，从库中的 I/O 线程负责接收
4. 从库的 I/O 线程将接收的 binlog 写入到 relay log 中。
5. 从库的 SQL 线程读取 relay log 同步数据本地（也就是再执行一遍 SQL ）。

```

#### 如何避免主从延迟？

```
读写分离对于提升数据库的并发非常有效，但是，同时也会引来一个问题：
主库和从库的数据存在延迟，比如你写完主库之后，主库的数据同步到从库是需要时间的，这个时间差就导致了主库和从库的数据不一致性问题。这也就是我们经常说的 主从同步延迟 

1. 强制将读请求路由到主库处理
既然你从库的数据过期了，那我就直接从主库读取嘛！这种方案虽然会增加主库的压力，但是，实现起来比较简单，也是我了解到的使用最多的一种方式。
对于这种方案，你可以将那些必须获取最新数据的读请求都交给主库处理。

2. 延迟读取
还有一些朋友肯定会想既然主从同步存在延迟，那我就在延迟之后读取啊，比如主从同步延迟 0.5s,那我就 1s 之后再读取数据。这样多方便啊！方便是方便，但是也很扯淡。
不过，如果你是这样设计业务流程就会好很多：对于一些对数据比较敏感的场景，你可以在完成写请求之后，避免立即进行请求操作。比如你支付成功之后，跳转到一个支付成功的页面，当你点击返回之后才返回自己的账户。
```

#### 什么情况下会出现主从延迟？如何尽量减少延迟？

```
MySQL 主从同步延时是指从库的数据落后于主库的数据，这种情况可能由以下两个原因造成：
	1. 从库 I/O 线程接收 binlog 的速度跟不上主库写入 binlog 的速度，导致从库 relay log 的数据滞后于主库 binlog 的数据；
	2. 从库 SQL 线程执行 relay log 的速度跟不上从库 I/O 线程接收 binlog 的速度，导致从库的数据滞后于从库 relay log 的数据。

与主从同步有关的时间点主要有 3 个：
	1. 主库执行完一个事务，写入 binlog，将这个时刻记为 T1；
	2. 从库 I/O 线程接收到 binlog 并写入 relay log 的时刻记为 T2；
	3. 从库 SQL 线程读取 relay log 同步数据本地的时刻记为 T3。
结合我们上面讲到的主从复制原理，可以得出：
	1. T2 和 T1 的差值反映了从库 I/O 线程的性能和网络传输的效率，这个差值越小说明从库 I/O 线程的性能和网络传输效率越高。
	2. T3 和 T2 的差值反映了从库 SQL 线程执行的速度，这个差值越小，说明从库 SQL 线程执行速度越快。
	
那什么情况下会出现出从延迟呢？这里列举几种常见的情况：
	1. 从库机器性能比主库差：从库接收 binlog 并写入 relay log 以及执行 SQL 语句的速度会比较慢（也就是 T2-T1 和 T3-T2 的值会较大），进而导致延迟。解决方法是选择与主库一样规格或更高规格的机器作为从库，或者对从库进行性能优化，比如调整参数、增加缓存、使用 SSD 等。
	2. 从库处理的读请求过多：从库需要执行主库的所有写操作，同时还要响应读请求，如果读请求过多，会占用从库的 CPU、内存、网络等资源，影响从库的复制效率（也就是 T2-T1 和 T3-T2 的值会较大，和前一种情况类似）。解决方法是引入缓存（推荐）、使用一主多从的架构，将读请求分散到不同的从库，或者使用其他系统来提供查询的能力，比如将 binlog 接入到 Hadoop、Elasticsearch 等系统中。
	3. 大事务：运行时间比较长，长时间未提交的事务就可以称为大事务。由于大事务执行时间长，并且从库上的大事务会比主库上的大事务花费更多的时间和资源，因此非常容易造成主从延迟。解决办法是避免大批量修改数据，尽量分批进行。类似的情况还有执行时间较长的慢 SQL ，实际项目遇到慢 SQL 应该进行优化。
	4. 从库太多：主库需要将 binlog 同步到所有的从库，如果从库数量太多，会增加同步的时间和开销（也就是 T2-T1 的值会比较大，但这里是因为主库同步压力大导致的）。解决方案是减少从库的数量，或者将从库分为不同的层级，让上层的从库再同步给下层的从库，减少主库的压力。
	5. 网络延迟：如果主从之间的网络传输速度慢，或者出现丢包、抖动等问题，那么就会影响 binlog 的传输效率，导致从库延迟。解决方法是优化网络环境，比如提升带宽、降低延迟、增加稳定性等。
	6. 单线程复制：MySQL5.5 及之前，只支持单线程复制。为了优化复制性能，MySQL 5.6 引入了 多线程复制，MySQL 5.7 还进一步完善了多线程复制。
	7. 复制模式：MySQL 默认的复制是异步的，必然会存在延迟问题。全同步复制不存在延迟问题，但性能太差了。半同步复制是一种折中方案，相对于异步复制，半同步复制提高了数据的安全性，减少了主从延迟（还是有一定程度的延迟）

```

### 什么是分库？

```
分库 就是将数据库中的数据分散到不同的数据库上，可以垂直分库，也可以水平分库。

垂直分库 就是把单一数据库按照业务进行划分，不同的业务使用不同的数据库，进而将一个数据库的压力分担到多个数据库。
举个例子：说你将数据库中的用户表、订单表和商品表分别单独拆分为用户数据库、订单数据库和商品数据库。

水平分库 是把同一个表按一定规则拆分到不同的数据库中，每个库可以位于不同的服务器上，这样就实现了水平扩展，解决了单表的存储和性能瓶颈的问题。
举个例子：订单表数据量太大，你对订单表进行了水平切分（水平分表），然后将切分后的 2 张订单表分别放在两个不同的数据库。
```

#### 什么是分表？

```
分表 就是对单表的数据进行拆分，可以是垂直拆分，也可以是水平拆分。

垂直分表 是对数据表列的拆分，把一张列比较多的表拆分为多张表。
举个例子：我们可以将用户信息表中的一些列单独抽出来作为一个表。

水平分表 是对数据表行的拆分，把一张行比较多的表拆分为多张表，可以解决单一表数据量过大的问题。
举个例子：我们可以将用户信息表拆分成多个用户信息表，这样就可以避免单一表数据量过大对性能造成影响。

水平拆分只能解决单表数据量大的问题，为了提升性能，我们通常会选择将拆分后的多张表放在不同的数据库中。也就是说，水平分表通常和水平分库同时出现。
```

#### 什么情况下需要分库分表？

```
1. 单表的数据达到千万级别以上，数据库读写速度比较缓慢。
2. 数据库中的数据占用的空间越来越大，备份时间越来越长。
3. 应用的并发量太大。
```

#### 常见的分片算法有哪些？

```
分片算法主要解决了数据被水平分片之后，数据究竟该存放在哪个表的问题。                                        哈希分片：求指定 key（比如 id） 的哈希，然后根据哈希值确定数据应被放置在哪个表中。
1. 哈希分片: 比较适合随机读写的场景，不太适合经常需要范围查询的场景。
2. 范围分片：按照特性的范围区间（比如时间区间、ID 区间）来分配数据，比如 将 id 为 1~299999 的记录分到第一个库， 300000~599999 的分到第二个库。
3. 范围分片: 适合需要经常进行范围查找的场景，不太适合随机读写的场景（数据未被分散，容易出现热点数据的问题）。
4. 地理位置分片：很多 NewSQL 数据库都支持地理位置分片算法，也就是根据地理位置（如城市、地域）来分配数据。
5. 融合算法：灵活组合多种分片算法，比如将哈希分片和范围分片组合。                                                         
```

#### 分库分表会带来什么问题呢？

```
引入分库分表之后，会给系统带来什么挑战呢？
1. join 操作：同一个数据库中的表分布在了不同的数据库中，导致无法使用 join 操作。这样就导致我们需要手动进行数据的封装，比如你在一个数据库中查询到一个数据之后，再根据这个数据去另外一个数据库中找对应的数据。不过，很多大厂的资深 DBA 都是建议尽量不要使用 join 操作。因为 join 的效率低，并且会对分库分表造成影响。对于需要用到 join 操作的地方，可以采用多次查询业务层进行数据组装的方法。不过，这种方法需要考虑业务上多次查询的事务性的容忍度。

2. 事务问题：同一个数据库中的表分布在了不同的数据库中，如果单个操作涉及到多个数据库，那么数据库自带的事务就无法满足我们的要求了。这个时候，我们就需要引入分布式事务了。

3. 分布式 ID：分库之后， 数据遍布在不同服务器上的数据库，数据库的自增主键已经没办法满足生成的主键唯一了。我们如何为不同的数据节点生成全局唯一主键呢？这个时候，我们就需要为我们的系统引入分布式 ID 了。

4. 跨库聚合查询问题：分库分表会导致常规聚合查询操作，如 group by，order by 等变得异常复杂。这是因为这些操作需要在多个分片上进行数据汇总和排序，而不是在单个数据库上进行。为了实现这些操作，需要编写复杂的业务代码，或者使用中间件来协调分片间的通信和数据传输。这样会增加开发和维护的成本，以及影响查询的性能和可扩展性。

```

#### 分库分表后，数据怎么迁移呢？

```
分库分表之后，我们如何将老库（单库单表）的数据迁移到新库（分库分表后的数据库系统）呢？
比较简单同时也是非常常用的方案就是停机迁移，写个脚本老库的数据写到新库中。比如你在凌晨 2 点，系统使用的人数非常少的时候，挂一个公告说系统要维护升级预计 1 小时。然后，你写一个脚本将老库的数据都同步到新库中。

如果你不想停机迁移数据的话，也可以考虑双写方案。双写方案是针对那种不能停机迁移的场景，实现起来要稍微麻烦一些。具体原理是这样的：
1. 我们对老库的更新操作（增删改），同时也要写入新库（双写）。如果操作的数据不存在于新库的话，需要插入到新库中。 这样就能保证，咱们新库里的数据是最新的。
2. 在迁移过程，双写只会让被更新操作过的老库中的数据同步到新库，我们还需要自己写脚本将老库中的数据和新库的数据做比对。如果新库中没有，那咱们就把数据插入到新库。如果新库有，旧库没有，就把新库对应的数据删除（冗余数据清理）。
3. 重复上一步的操作，直到老库和新库的数据一致为止。
```



#### binlog和redolog有什么区别？

```
binlog和redolog都是Mysql里面用来记录数据库数据变更操作的日志。
binlog和redolog的区别有很多，我可以简单总结三个点

1. 使用场景不同，binlog主要用来做数据备份、数据恢复、以及主从集群的数据同步； Redo Log主要用来实现Mysql数据库的事务恢复，保证事务的ACID特性。当数据库出现崩溃的时候，Redo Log可以把未提交的事务回滚，把已提交的事务进行持久化，从而保证数据的一致性和持久性。
2. 记录的信息不同，binlog是记录数据库的逻辑变化，它提供了三种日志格式分别是statement，row以及mixed；
redo log记录的是物理变化，也就是数据页的变化结果。

3. 记录的时机不同， binlog是在执行SQL语句的时候，在主线程中生成逻辑变化写入到磁盘中，所以它是语句级别的记录方式； RedoLog是在InnoDB存储引擎层面的操作，它是在Mysql后台线程中生成并写入到磁盘中的，所以它是事务级别的记录方式，一个事务操作完成以后才会被写入到redo log中。

```



#### MySQL 存储引擎架构了解吗？

```
MySQL 存储引擎采用的是 插件式架构 ，支持多种存储引擎，我们甚至可以为不同的数据库表设置不同的存储引擎以适应不同场景的需要。存储引擎是基于表的，而不是数据库。

并且，你还可以根据 MySQL 定义的存储引擎实现标准接口来编写一个属于自己的存储引擎。这些非官方提供的存储引擎可以称为第三方存储引擎，区别于官方存储引擎。
```

#### mysql 分区有哪几种？

```
RANGE分区：基于属于一个给定连续区间的列值，把多行分配给分区。

LIST分区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。

HASH分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL 中有效的、产生非负整数值的任何表达式。

KEY分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。

```


#### 1. go 原子操作

go 中 sync.atomic的常见操作

Atomic 提供的原子操作能够确保任一时刻只有一个goroutine对变量进行操作,善用atomic能够避免程序中出现大量的锁操作.

atomic常见操作有:

增减

载入

比较并交换

交换

存储

##### 1. 增减操作

atomic包中提供了如下以Add为前缀的增减操作:

```go
- func AddInt32(addr *int32, delta int32) (new int32)

- func AddInt64(addr *int64, delta int64) (new int64)

- func AddUint32(addr *uint32, delta uint32) (new uint32)

- func AddUint64(addr *uint64, delta uint64) (new uint64)

- func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)
```

需要注意的是，第一个参数必须是指针类型的值，通过指针变量可以获取被操作数在内存中的地址，从而施加特殊的CPU指令，确保同一时间只有一个goroutine能够进行操作。

使用举例：

```go
package 
import (
	"fmt"
  "sync/atomic"
  "time"
)

func main() {
  var opts int64 = 0
  
  for i := 0; i < 50; i++ {
    // 注意第一个参数必须是地址
    atomic.AddInt64(&opts, 3) // 加操作
    // atomic.AddInt64(&opts, -1) // 减操作
    time.Sleep(time.Millisecond)
  }
  time.Sleep(time.Second)
  fmt.Println("opts: ", atomic.LoadInt64(&opts))
}
```



##### 2. 载入操作

atomic 包中提供了如下以Load为前缀的增减操作:

````go
- func LoadInt32(addr *int32) (val int32)

- func LoadInt64(addr *int64) (val int64)

- func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

- func LoadUint32(addr *uint32) (val uint32)

- func LoadUint64(addr *uint64) (val uint64)

- func LoadUintptr(addr *uintptr) (val uintptr)

````

载入操作能够保证原子的读变量的值，当读取的时候，任何其它的CPU操作都无法对该变量进行读写，其实现机制受到底层硬件的支持，见上述例子中的atomic.LoadInt64(&opts)

##### 3. 比较并交换

该操作简称CAS（Compare And Swap)。这类操作的前缀为CompareAndSwap:

```go
- func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)

- func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)

- func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)

- func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)

- func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)

- func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)

```

该操作在进行交换前首先确保变量的值未被改变，即仍然保持参数old所记录的值，满足此前提下才进行交换操作。CAS的做法类型操作数据库时常见的乐观锁机制。

需要注意的是，当有大量的goroutine对变量进行读写操作时，可能导致CAS操作无法成功，这时可以利用for循环多次尝试。

使用实例：

```go
var value int64

func atomicAddOp(tmp int64) {
  for {
    oldValue := value
    if atomic.CompareAndSwapInt64(&value, oldValue, oldValue+tmp) {
      return
    }
  }
}
```



##### 4. 交换

此类操作的前缀为Swap:

```go
- func SwapInt32(addr *int32, new int32) (old int32)

- func SwapInt64(addr *int64, new int64) (old int64)

- func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

- func SwapUint32(addr *uint32, new uint32) (old uint32)

- func SwapUint64(addr *uint64, new uint64) (old uint64)

- func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
```

相对于CAS，明显此类操作更为暴力直接，并不管变量的旧值是否被改变，直接赋予新值然后返回被替换的值。



##### 5. 存储

此类操作的前缀为Store: 

```go
- func StoreInt32(addr *int32, val int32)

- func StoreInt64(addr *int64, val int64)

- func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)

- func StoreUint32(addr *uint32, val uint32)

- func StoreUint64(addr *uint64, val uint64)

- func StoreUintptr(addr *uintptr, val uintptr)
```

此类操作确保了写变量的原子性，避免其他操作读到了修改变量过程中的脏数据。
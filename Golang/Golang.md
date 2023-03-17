[toc]

## 基础语法

### 1. make 和 new 的区别

共同点：

1. new 和 make 都用于分配内存
2. new 和 make 都是在堆上分配内存

不同点：

1. new 对指针类型分配内存，并且内存对应的值为类型零值，返回值是分配类型的指针
2. make 用于 slice、map和 channel 的初始化，返回值为引用类型本身，而不是指针
3. new 分配的空间被清零，make 分配空间后，会进行初始化



### 2. 常量和变量的区别

1. 变量是“可读、可写”，而常量是“只读”的

2. 变量在运行期分配存储内存（非优化状态）

   常量通常会被编译器在预处理阶段直接展开，作为指令数据使用

   数字常量不会分配存储空间，无须像变量那样通过内存寻址来取值，因此无法获取地址



### 3. Go函数参数传递方式

https://blog.csdn.net/m0_71777195/article/details/125502836

**Go里面函数传参只有值传递一种方式**

> 值传递：指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
>
> 参数传递还有引用传递，所谓引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

Go语言中所有的传参都是值传递（传值），都是一个副本，一个拷贝。因为拷贝的内容有时候是非引用类型（int、string、struct等这些），这样就在函数中就无法修改原内容数据；有的是引用类型（指针、map、slice、channel等这些），这样就可以修改原内容数据。

> 所谓值类型：变量和变量的值存在同一个位置。
>
> 所谓引用类型：变量和变量的值是不同的位置，变量的值存储的是对值的引用。

### 4. for range 的时候它的地址会发生变化么？

for range的时候，地址并没有发生变化。在循环时，会创建一个变量，之后每次循环时遍历到的数据都是以值覆盖的方式赋给这个变量。内存地址始终不变。

解决办法：在每次循环时，创建一个临时变量。

### 5. defer

https://blog.csdn.net/Cassie_zkq/article/details/108567205

#### 执行顺序

- 多个 defer 语句，遵从后进先出的原则，最后声明的 defer 语句，最先得到执行。
- defer 在 return 语句之后执行，但在函数退出之前，defer 可以修改返回值。

#### defer特性

1. 多个defer语句，按先进后出的方式执行

   所有的defer语句会放入栈中，在入栈的时候会进行相关的值拷贝（也就是下面的“对应的参数会实时解析”）。

2. defer声明时，对应的参数会实时解析

   辨析：defer后面跟无参函数、有参函数和方法

   ```go
   package main
   
   import "fmt"
   
   func test(a int) {//无返回值函数
   	defer fmt.Println("1、a =", a) //方法
   	defer func(v int) {
           fmt.Println("2、a =", v)
       }(a) //有参函数
   	defer func() {
           fmt.Println("3、a =", a)
       }() //无参函数
   	a++
   }
   func main() {
   	test(1)
   }
   
   /**
   3、a = 2
   2、a = 1
   1、a = 1
   */
   ```

3. 关键字 defer 用于注册延迟调用。

4. 这些调用直到 return 前才被执。因此，可以用来做资源清理。

   defer、return、返回值三者的执行逻辑应该是：
   return最先执行，return负责将结果写入返回值中；
   接着defer开始执行一些收尾工作；
   最后函数携带**当前返回值**（可能和最初的返回值不相同）退出。

   **有函数返回值的则return将结果写入返回值，defer进行收尾，可以看做 return最先执行，然后return将结果存入返回值，最后defer执行**

5. defer可以修改函数最终返回值，修改时机：有名返回值或者函数返回指针

####  defer用途

1. 关闭文件句柄

   ```go
   unc ReadFile(filename string) ([]byte, error) {
       f, err := os.Open(filename)
       if err != nil {
           return nil, err
       }
       defer f.close()
       return ReadAll()
   }
   ```

2. 锁资源释放

   ```go
   var mu sync.Mutex
   var m = make(map[string]int)
    
   func lookup(key string) int {
       mu.Lock()
       defer mu.Unlock()
       return m[key]
   }
   ```

3. 数据库连接释放

4. 配合recover()来恢复发生 panic

   ```go
   defer func() {
     if e := recover(); e != nil {
       if se, ok := e.(error); ok {
         err = se.Error()
       }
     }
   }()
   ```



### 6. 字符串

sring不可变，[]byte可变

#### byte 和 rune

byte等同于**uint8**，表示一个字节，常用来处理ascii字符

rune等同于**int32**，主要用于表示一个字符类型大于一个字节小于等于4个字节的情况下，常用来处理unicode或utf-8字符，特别是**中文字符。**

#### 字符串拼接

使用 + 拼接性能最差，strings.Builder，bytes.Buffer 相近，strings.Buffer 更快

原因：

- 字符串在 Go 语言中是不可变类型，占用内存大小是固定的
- 使用 +。每次都会重新分配内存
- strings.Builder，bytes.Buffer 底层都是[]byte 数组
- 内存扩容策略，不需要每次拼接重新分配内存



### 7. tag 和反射

反射可以在==运行期间==，操作任意类型的对象。

可以通过`TypeOf`方法获得对象类型。通过`ValueOf`获得对象值。

Go 中解析的 tag 是通过反射实现的。

tag 可以理解为 struct 字段的注解，可以用来定义字段的一个或多个属性。框架/工具可以通过反射获取到某个字段定义的属性，采取相应的处理方式。tag 丰富了代码的语义，增强了灵活性。



### 8. 空 struct{} 的用途

使用空结构体 struct{} 可以节省内存，一般作为占位符使用，表明这里并不需要一个值。

1. 使用 map 表示集合 set 时，只关注 key，value 可以使用 struct{} 作为占位符。如果使用其他类型作为占位符，例如 int，bool，不仅浪费了内存，而且容易引起歧义。
2. 使用信道(channel)控制并发时，我们只是需要一个信号，但并不需要传递值，这个时候，也可以使用 struct{} 代替。
3. 声明只包含方法的结构体。



### 10. interface 比较

interface 的内部实现包含了 2 个字段，类型 T 和 值 V

- 两个接口值比较时，会先比较 T，再比较 V。
- 接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较。

2 个 interface 相等有以下2种情况：

1. 两个 interface 均等于 nil（此时 V 和 T 都处于 unset 状态）
2. 类型 T 相同，且对应的值 V 相等。

```go
func main() {
    var p *int = nil
    var i interface{} = p
    fmt.Println(i == p) // true
    fmt.Println(p == nil) // true
    fmt.Println(i == nil) // false
}
```

上面这个例子中，将一个 nil 非接口值 p 赋值给接口 i，此时，i 的内部字段为`(T=*int, V=nil)`，i 与 p 作比较时，将 p 转换为接口后再比较，因此 `i == p`，p 与 nil 比较，直接比较值，所以 `p == nil`。

但是当 i 与 nil 比较时，会将 nil 转换为接口 `(T=nil, V=nil)`，与i `(T=*int, V=nil)` 不相等，因此 `i != nil`。因此 V 为 nil ，但 T 不为 nil 的接口不等于 nil。



### 10. goroutine 内存泄漏

#### 原因

- Goroutine 内进行channel/mutex 等读写操作被一直阻塞。 

- Goroutine 内的业务逻辑进入死循环，资源一直无法释放。 

- Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待

#### 场景

- channel 如果忘记初始化，那么无论你是读，还是写操作，都会造成阻塞。 
- channel 发送数量 超过 channel接收数量，就会造成阻塞
- channel 接收数量 超过 channel发送数量，也会造成阻塞
- http request body未关闭，goroutine不会退出
- 互斥锁忘记解锁
- sync.WaitGroup使用不当

#### 排查

单个函数：调用 `runtime.NumGoroutine` 方法来打印 执行代码前后Goroutine 的运行数量，进行前后比较，就能知道有没有泄露了。

生产/测试环境：使用`PProf`实时监测Goroutine的数量。



### 11. init() 函数是什么时候执行的？

import –> const –> var –> init() –> main()

import : 由 runtime 初始化每个导入的包，初始化顺序不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化。



## 底层数据结构

### 1. Slice

https://zhuanlan.zhihu.com/p/449851884

切片的本质就是对底层数组的封装，它包含了三个信息：底层数组的指针、切片的长度（len）和切片的容量（cap）。

slice底层结构并没有使用加锁的方式，不支持并发读写，不是线程安全的。

现在有一个数组 a := [8] int {0, 1, 2, 3, 4, 5, 6, 7}，切片 s1 := a [:5]，相应示意图如下。

![slice](https://cdn.learnku.com/uploads/images/202302/07/24886/54qhsTWejC.png!large)

> 如果没有发生扩容，修改在原来的内存中
>
> 如果发生了扩容，修改会在新的内存中
>
> **在复制 slice 的时候，slice 中数组的指针也被复制了，在触发扩容逻辑之前，两个 slice 指向的是相同的数组，触发扩容逻辑之后指向的就是不同的数组了**

#### 扩容机制

GO1.17版本及之前

1. 当新切片需要容量 > 两倍的原容量时，直接使用期望容量作为新切片的容量。

2. 如果原容量 < 1024，那么新切片的容量变成原来的 2 倍。（避免频繁扩容，从而减少内存分配的次数和数据拷贝的代价）

3. 如果原容量 >= 1024，进入一个循环，每次容量变成原来的1.25倍（实际上是不超过25%），直到大于期望容量。（主要避免空间浪费）

GO1.18之后

1. 当新切片需要容量 > 两倍的原容量时，直接使用期望容量作为新切片的容量。
2. 当原容量 < threshold，新 slice 容量变成原来的 2 倍。（threshold=256）
3. 当原容量 > threshold，进入一个循环，每次容量增加 (旧容量+3*threshold)/4，直到大于期望容量。

#### 非线程安全

slice不支持并发读写，所以并不是线程安全的，使用多个 goroutine 对类型为 slice 的变量进行操作，每次输出的值大概率都不会一样，与预期值不一致。 slice在并发执行中不会报错，但是数据会丢失。

如果想实现slice线程安全，有两种方式：

方式一：通过加锁实现slice线程安全，适合对性能要求不高的场景。

方式二：通过channel实现slice线程安全，适合对性能要求高的场景。

```go
buffer := make(chan int)
a := make([]int, 0)
// 消费者
go func() {
    for v := range buffer {
        a = append(a, v)
    }
}()
// 生产者
var wg sync.WaitGroup
for i := 0; i < 10000; i++ {
    wg.Add(1)
    go func(i int) {
        defer wg.Done()
        buffer <- i
    }(i)
}
```

#### 数组（Array）和切片（Slice）的区别

1. **数组长度不同**

   数组初始化必须指定长度，并且长度就是固定的

   切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大

2. **函数传参不同**

   数组是值类型，将一个数组赋值给另一个数组时，传递的是一份深拷贝，函数传参操作都会复制整个数组数据，会占用额外的内存，函数内对数组元素值的**修改**，不会修改原数组内容。

   切片是引用类型，将一个切片赋值给另一个切片时，传递的是一份浅拷贝，函数传参操作不会拷贝整个切片，只会复制len和cap，底层共用同一个数组，不会占用额外的内存，函数内对数组元素值的**修改**，会修改原数组内容。

   > 深拷贝：拷贝的是数据本身，创造一个样的新对象，新创建的对象与原对象不共享内存，新创建的对象在内存中开辟一个新的内存地址，新对象值修改时不会影响原对象值。
   >
   > 浅拷贝：拷贝的是数据地址，只复制指向的对象的指针，此时新对象和老对象指向的内存地址是一样的，新对象值修改时老对象也会变化。
   >
   > 值类型的数据，默认赋值操作都是深拷贝。
   >
   > 引用类型的数据，默认全部都是浅拷贝。

#### 内存泄漏

由于slice的底层是数组，很可能数组很大，但slice所取的元素数量却很小，这就导致数组占用的绝大多数空间是被浪费的

### 2. Map

https://zhuanlan.zhihu.com/p/460958342

Golang 中 map 的底层实现是一个哈希表。map类型变量实际上是一个指针，指向hmap结构体，hmap包含多个bmap数组（桶）

```go
type hmap struct {
    count     int                  // 元素个数
    flags     uint8
    B         uint8                // B是buckets数组的长度的对数 2^B表示桶数量
    noverflow uint16               // 溢出的bucket个数
    hash0     uint32               // hash seed

    buckets    unsafe.Pointer      // 桶
    oldbuckets unsafe.Pointer      // 旧桶
    nevacuate  uintptr             // 即将迁移的旧桶编号

    extra *mapextra                // 用于扩容的指针
}
```

<img src="https://gitee.com/Transmigration_zhou/pic/raw/master/image-20230213002458315.png" alt="image-20230213002458315" style="zoom:50%;" />

![图片](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/640.png)

#### 扩容机制

> 双倍扩容：分配新桶是旧桶的两倍，然后旧buckets数据搬迁到新的buckets。
>
> 等量扩容：不扩大容量，buckets数量维持不变，重新做一遍类似双倍扩容的搬迁动作，把松散的键值对重新排列一次，使得同一个 bucket 中的 key 排列地更紧密，节省空间，提高 bucket 利用率，减少溢出桶的使用，进而保证更快的存取。
>
> 负载因子越大，填入的元素越多，空间利用率就越高，但发生哈希冲突的几率就变大。
> 负载因子越小，填入的元素越少，冲突发生的几率减小，但空间浪费也会变得更多，而且还会提高扩容操作的次数。

1. 负载因子 = 哈希表存储的元素个数 / 桶个数（count/2^B^）> 6.5，触发双倍扩容

2. 溢出桶太多，触发等量扩容

   - 当桶数量 <= 2^15^ 时（B<=15），如果溢出桶总数(noverflow) >= 桶数量

   - 当桶数量 > 2^15^ 时，如果溢出桶总数 >= 2^15^

#### 查找过程

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220117201006909.png)

1. 写保护机制

   先查hmap的标志位flags，如果flags写标志位此时是1，说明其他协程正在写操作，直接panic

2. 计算hash值

   key经过哈希函数计算后，得到64bit哈希值
   10010111 | 101011101010110101010101101010101010 | 10010

4. 找到hash对应的桶

   上面64位后5(hmap的B值)位定位所存放的桶
   如果当前正在扩容中，并且定位到旧桶数据还未完成迁移，则使用旧的桶

4. 遍历桶查找

   上面64位前8位用来在tophash数组查找快速判断key是否在当前的桶中，如果不在需要去溢出桶查找  

5. 返回key对应的指针

#### 冲突解决方式

采用链地址法解决冲突，具体就是插入key到map中时，当key定位的桶填满8个元素后，将会创建一个溢出桶，并且将溢出桶插入当前桶的所在链表尾部。

#### 非线程安全

不支持并发写的（panic），但是可以并发读，是一种非线程安全的结构。

如果想实现map线程安全，有两种方式：

方式一：使用读写锁 `map` + `sync.RWMutex`

方式二：使用Go提供的 `sync.Map`

##### sync.Map

https://blog.csdn.net/Darrenchiu/article/details/107337482

```go
type Map struct {
    mu Mutex
    read atomic.Value
    dirty map[any]*entry
    misses int
}
```

和原始map+RWLock的实现并发的方式相比，减少了加锁对性能的影响。

sync.Map里有两个map一个是专门用于读的read map，另一个是提供读写的dirty map；优先读read map，若不存在则加锁穿透读dirty map，同时记录一个未从read map读到的计数，当计数到达一定值，就将read map用dirty map进行覆盖。

优点：通过读写分离，降低锁时间来提高效率。

缺点：不适用于大量写的场景，这样会导致read map读不到数据而进一步加锁读取，同时dirty map也会一直晋升为read map，整体性能较差。

适用场景：大量读，少量写。存在大量写的场景可以考虑map+metux。



### 3. Channel

channel 是一个队列，遵循先进先出的原则，负责协程之间的通信



channel变量是一个存储在函数栈帧上的指针，指向堆上的hchan结构体

```go
type hchan struct {
    closed   uint32   // channel是否关闭的标志
    elemtype *_type   // channel中的元素类型

    buf      unsafe.Pointer // "环形"缓存区（数组）的位置
    qcount   uint           // 已经存储元素个数
    dataqsiz uint           // 最多存储元素个数
    elemsize uint16         // 每个元素占多大空间
    sendx    uint           // 写下标的位置
    recvx    uint           // 读下标的位置
    // 尝试读取channel或向channel写入数据而被阻塞的goroutine
    recvq    waitq  // 读等待队列
    sendq    waitq  // 写等待队列
    lock mutex //互斥锁，保证读写channel时不存在并发竞争问题
}
```

| 操作 \ 状态 | 未初始化         | 关闭                               | 正常             | 没值     | 满       |
| ----------- | ---------------- | ---------------------------------- | ---------------- | -------- | -------- |
| 关闭        | panic            | panic                              | 关闭成功         | 关闭成功 | 关闭成功 |
| 发送        | 永远阻塞导致死锁 | panic                              | 阻塞或者发送成功 | 发送成功 | 阻塞     |
| 接收        | 永远阻塞导致死锁 | 缓冲区为空则为零值，否则可以继续读 | 阻塞或者接收成功 | 阻塞     | 接收成功 |

如果多个 goroutine 都监听同一个 channel，那么 channel 上的数据都可能随机被某一个 goroutine 取走进行消费

如果多个 goroutine 都监听同一个 channel，如果这个 channel 被关闭，则所有 goroutine 都能收到退出信号



**向channel写数据的流程：**

1. 如果等待接收队列recvq不为空，说明缓冲区中没有数据或者没有缓冲区，此时直接从recvq取出G,并把数据写入，最后把该G唤醒，结束发送过程；
2. 如果缓冲区中有空余位置，将数据写入缓冲区，结束发送过程；
3. 如果缓冲区中没有空余位置，将待发送数据写入G，将当前G加入sendq，进入睡眠，等待被读goroutine唤醒；

**向channel读数据的流程：**

1. 如果等待发送队列sendq不为空且没有缓冲区，直接从sendq中取出G，把G中数据读出，最后把G唤醒，结束读取过程；
2. 如果等待发送队列sendq不为空，此时说明缓冲区已满，从缓冲区中首部读出数据，把G中数据写入缓冲区尾部，把G唤醒，结束读取过程；
3. 如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程；
4. 如果缓冲区中没有数据，将当前goroutine加入recvq，进入睡眠，等待被写goroutine唤醒；



#### 无缓冲channel与容量为1的channel的区别

https://www.cnblogs.com/jo3yzhu/p/13559761.html

##### 无缓冲的channel

```go
c1 := make(chan int) // 无缓冲

c1 <- 1   // A
<- c1     // B
```

**这里的A或B，无论谁先执行，谁都会阻塞以等待另一个goroutine执行**，也就是说往里写得等来读的，从里读得等来写的。最重要的是，**A和B对c1的读写是同步的**，直观的理解是A和B对c1的读写是同时发生的，当A对c1写完了，则B从c1中就读完了。这样的特性可以用于做并发单位之间的同步操作，如果在A和B中对同一个无缓冲通道进行了读写，那么A和B一定会在读写的地方进行同步，谁先到谁阻塞等待另外一个。
综上，==如果在一个协程里写这样的代码，一定会死锁。==无缓冲的channel的读写者必须**同时完成发送和接收**，而不能串行，显然单协程无法满足。所以这里造成了循环等待，会死锁。

##### 缓冲为1的channel

```go
c2 := make(chan int, 1) // 有缓冲

c2 <- 1   // A
<- c2     // B
```

有缓冲的通道并不强制channel的读写者必须同时完成发送和接收，读者只会在没有数据时阻塞，写者只会在没有可用容量时阻塞，这就有点像阻塞队列了。

#### 使用场景

- 停止信号监听

  关闭channel或向channel发送一个元素，使接收方通过channel获得信息后做相应的操作

- 定时任务（select语句）

  - 超时控制
  - 定期执行某个任务

- 解耦生产方和消费方

  服务启动时，启动n个worker，作为工作协程池，这些协程工作在一个for无限循环里, 从某个channel消费工作任务并执行

- 控制并发数

  `var limit = make(chan int, 3)`

#### 线程安全

channel的底层实现中，hchan结构体中采用Mutex锁来保证数据读写安全。在对循环数组buf中的数据进行入队和出队操作时，必须先获取互斥锁，才能操作channel数据。

#### channel如何控制goroutine并发执行顺序

使用channel进行通信通知，用channel去传递信息，从而控制并发执行顺序

从第x个协程中拿数据，通知第x+1个协程

#### channel共享内存有什么优劣势

go设计思想：**不要通过共享内存来通信，我们应该使用通信来共享内存**

go引入了channel和goroutine实现CSP模型，将生产者和消费者进行了解耦，channel和消息队列很相似。

**优点：**使用 channel 可以帮助我们解耦生产者和消费者，可以降低并发当中的耦合

**缺点：**容易出现死锁

#### channel发送和接收什么情况下会死锁

1. 非缓存channel读写不能同时

2. 缓存channel缓存满时写/缓存空时读

3. 多个channel相互等待



### 4. Mutex

Go sync包提供了两种锁类型：互斥锁sync.Mutex 和 读写互斥锁sync.RWMutex，都属于悲观锁。

```GO
type Mutex struct {  
     state int32  
     sema  uint32
 }
```

state表示锁的状态，有锁定、被唤醒、饥饿模式等，并且是用state的二进制位来标识的，不同模式下会有不同的处理方式。

![mutex_state](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/mutex_state.png)

sema表示信号量，mutex阻塞队列的定位是通过这个变量来实现的，从而实现goroutine的阻塞和唤醒。

- 重复 Unlock() 会导致 panic 
- 使用 Lock() 加锁后，再次 Lock() 会导致死锁
- 锁定状态与 goroutine 没有关联，一个 goroutine 可以 Lock，另一个 goroutine 可以 UnlockWaitGroup

#### 加解锁过程

```go
func (m *Mutex) Lock() {
    // Fast path
    if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
    	...
        return
    }
    // Slow path
    m.lockSlow()
}
```

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/mutex_lock.png)

```go
func (m *Mutex) Unlock() {  
   ...
   // Fast path()
   new := atomic.AddInt32(&m.state, -mutexLocked)  
   if new != 0 {  
       // Slow path
       m.unlockSlow(new)
   }  
}
```

![mutex_unlock](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/mutex_unlock.png)

#### 自旋

自旋：加锁时，如果当前Locked位为1，说明该锁当前由其他协程持有，尝试加锁的**协程并不是马上转入阻塞，而是会持续的探测Locked位是否变为0**，这个过程即为自旋过程。（自旋等待，即不停地尝试获取锁，直到获取到为止）

自旋时间很短，但如果在自旋过程中发现锁已被释放，那么协程可以立即获取锁。此时即便有协程被唤醒也无法获取锁，只能再次阻塞。

自旋的好处是，当加锁失败时不必立即转入阻塞，有一定机会获取到锁，这样可以避免协程的切换。

自旋必须满足以下所有条件：

1. 锁已被占用，并且锁不处于饥饿模式。
2. 积累的自旋次数小于最大自旋次数（active_spin=4）。
3. cpu 核数大于 1。
4. 有空闲的 P。
5. 当前 goroutine 所挂载的 P 下，本地待运行队列为空。

#### Mutex模式

- 正常模式(非公平锁)

  该模式下，协程如果加锁不成功不会立即转入阻塞排队，而是判断是否满足自旋的条件，如果满足则会启动自旋过程，尝试抢锁。唤醒的goroutine 不会直接拥有锁，而是会和新请求锁的 goroutine 竞争锁。新请求锁的 goroutine 具有优势：它正在 CPU 上执行，而且可能有好几个，所以刚刚唤醒的 goroutine 有很大可能在锁竞争中失败，长时间获取不到锁，就会切换到饥饿模式。

- 饥饿模式(公平锁)

  当一个 goroutine 等待锁时间超过 1 毫秒时，它可能会遇到饥饿问题。

  处于饥饿模式下，不会启动自旋过程，也即一旦有协程释放了锁，那么一定会唤醒协程，被唤醒的协程将会成功获取锁，同时也会把等待计数减1。

恢复正常模式的条件：

1. G的执行时间小于1ms
2. 等待队列已经全部清空了



### 5. goroutine

goroutine是由Go运行时（runtime）管理的协程（用户态的轻量级线程），而不是操作系统管理。

相比较于操作系统线程，goroutine的资源占用和使用代价都要小得多，可以创建几十个、几百个甚至成千上万个goroutine也不会造成系统资源的枯竭，Go的运行时负责对goroutine进行管理。

goroutine 本身只是一个数据结构，真正让 goroutine 运行起来的是**调度器**。

Go 实现了一个用户态的调度器（GMP模型）

#### GMP模型

https://learnku.com/articles/41728

- G：goroutine协程，使用go关键字可以创建一个golang协程。

- M：thread线程，协程必须放在线程上执行。

- P：processor处理器，包含运行Go代码的必要资源，也有调度goroutine的能力，最多有 GOMAXPROCS 个。

  ==选择待执行G，井调度到线程M上执行。==

  ==M必须拥有P，才能执行G中的代码，P负责G的调度。==

引入P的原因是：GM模型M获取G时都要加锁，会因为频繁加锁解锁而发生等待，影响程序并发性能。

<img src="https://www.liwenzhou.com/images/Go/concurrence/gpm.png" alt="gpm" style="zoom: 67%;" />

> 全局队列：存放等待运行的 G。
>
> P 的本地队列：存放等待运行的G，数量不超过256个。新建 G 时，G 优先加入到 P 的本地队列，
>
> 新建一个goroutine的时候，优先放到P的本地队列中，如果本地队列满了会批量移动部分 G 到全局队列。
> 当P的本地队列为空时，M也会尝试从全局队列拿一些G放到P的本地队列 。
>
> M：线程想运行任务就得获取 P，从 P 的本地队列获取 G，P 队列为空时，M 也会尝试从全局队列拿一批 G 放到 P 的本地队列，或从其他 P 的本地队列偷一半放到自己 P 的本地队列。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。
>
> 当 M 调用结束时候，这个 G 会尝试获取一个空闲的 P ，并放入到这个 P 的本地队列。如果获取不到 P，那么这个线程 M 变成休眠状态， 加入到空闲线程中，然后这个 G 会被放入全局队列中。
>
> 一个P最多包含257个G：本地队列runq（256个）+优先级最高runnext（1个）
>
> runnext：一个P如果还有剩余时间片，那么P中runnext字段中的goroutine会继承这部分剩余时间并执行。



 **如果一个P上的G超过了257个咋办？**

如果一个P上的G超过257个，就会将超过部分放到全局队列上 ，全局队列是一个链表，无限长。



**如果一个P上的所有G执行完了咋办？**

`runtime.findrunnable() `

1. 如果P的runnext存在有就用这个G【==不加锁==】
2. 从P的本地队列中顺序拿一下G【 ==不加锁==】
3. 从全局队列中拿一个G【==加锁==】，顺便从全局队列拿出128（不足128就给全部）个G给这个P
4. 从netpoll网络轮询器中拿一个G，剩下的通过 injectglist 放到全局队列中【==加锁==】
5. 随机从其他的P的本地队列中偷一半给当前P
6. 将P设置为空闲状态并且将M从P中拿下来，但是不会kill M



##### goroutine调度流程

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/ddyl519.jpg)

#### 调度器的生命周期

![17-pic-go调度器生命周期.png](https://gitee.com/Transmigration_zhou/pic/raw/master/j37FX8nek9.png!large)

> M0 : 启动后编号为0的主线程，这个M对应的实例会在全局变量runtime.m0中，不需要在heap上分配，==M0负责执行初始化操作和启动第一个协程G，之后M0便和普通M一样。==
>
> G0：每次启动一个M都会第一个创建的goroutine，==G0仅负责调度G，G0不会被调度程序抢占，每个M都有一个自己的G0。==在调度或系统调用时会使用G0的栈空间，全局变量的G0是M0的G0。



#### goroutine发生重新调度的场景：

- 阻塞 I/O
- select操作
- 阻塞在channel
- 等待锁
- 主动调用 runtime.Gosched()



#### goroutine 的抢占式调度

https://jingwei.link/2019/05/26/golang-routine-scheduler.html

goroutine通过**抢占机制**来打断长时间占用 CPU 资源的 goroutine ，发起重新调度。

**抢占机制**：Golang 运行时（runtime）中的系统监控线程 sysmon 可以找出“长时间占用”的 goroutine，从而“提醒”相应的 goroutine 该中断了。

[toc]

## 基础语法

### 1. make 和 new 的区别

共同点：

1. new 和 make 都用于分配内存
2. new 和 make 都是在堆上分配内存

不同点：

1. new 创建一个指定类型的指针，并且内存对应的值为类型零值，返回值是分配类型的指针
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
>
> 严格来说，Go 语言没有引用类型，但是可以把 map、chan 称为引用类型，这样便于理解。除了 map、chan 之外，Go 语言中的函数、接口、slice 切片都可以称为引用类型。
>
> > 指针类型也可以理解为是一种引用类型。

#### 什么情况下传入函数的切片更改不会影响到外部？

如果指向底层数组的指针被覆盖或者修改（copy、重分配、append触发扩容），此时函数内部对数据的修改将不再影响到外部的切片，代表长度的len和容量cap也均不会被修改。

### 4. 深拷贝与浅拷贝

深拷贝：拷贝的是数据本身，创造一个样的新对象，新创建的对象与原对象不共享内存，新创建的对象在内存中开辟一个新的内存地址，新对象值修改时不会影响原对象值。

浅拷贝：拷贝的是数据地址，只复制指向的对象的指针，此时新对象和老对象指向的内存地址是一样的，新对象值修改时老对象也会变化。

值类型的数据，默认赋值操作都是深拷贝。

引用类型的数据，默认全部都是浅拷贝。

### 5. for range 的时候它的地址会发生变化么？

for range的时候，地址并没有发生变化。在循环时，会创建一个变量，之后每次循环时遍历到的数据都是以值覆盖的方式赋给这个变量。内存地址始终不变。

解决办法：在每次循环时，创建一个临时变量。

原理：https://juejin.cn/post/6844903701576957960

ps: 1.22的时候已经修复

### 6. defer

https://blog.csdn.net/Cassie_zkq/article/details/108567205

#### 执行顺序

- 多个 defer 语句，遵从后进先出的原则，最后声明的 defer 语句，最先得到执行。
- defer 在 return 语句之后执行，但在函数退出之前，defer 可以修改返回值。

#### defer特性

1. 多个defer语句，按后进先出的方式执行

   所有的defer语句会放入栈中，在入栈的时候会进行相关的值拷贝（也就是下面的“对应的参数会实时解析”）。

2. defer声明时，对应的参数会实时解析

   辨析：defer后面跟无参函数、有参函数和方法

   ```go
   package main
   
   import "fmt"
   
   func test(a int) {//无返回值函数
   	defer fmt.Println("1、a =", a) //方法（值拷贝）
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

   > https://juejin.cn/post/7095631673865273352
   >
   > return返回值的运行机制：return并非原子操作，共分为**赋值**、**返回值**两步操作。
   >
   > defer、return、返回值三者的执行逻辑应该是：
   > return最先执行，return负责将结果写入返回值中（即赋值）；
   > 接着defer开始执行一些收尾工作；
   > 最后函数携带**当前返回值**（可能和最初的返回值不相同）退出。
   >
   > 无名返回值会执行一个类似创建一个临时变量作为保存return值的动作
   >
   > 有名返回值的函数，由于返回值在函数定义的时候已经将该变量进行定义，在执行return的时候会先执行返回值保存操作，而后续的defer函数会改变这个返回值(虽然defer是在return之后执行的，但是由于使用的函数定义的变量，所以执行defer操作后对该变量的修改会影响到return的值。

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



### 7. 字符串

sring不可变，[]byte可变

#### byte 和 rune

byte等同于**uint8**，表示一个字节，常用来处理ascii字符

rune等同于**int32**，主要用于表示一个字符类型大于一个字节小于等于4个字节的情况下，常用来处理unicode或utf-8字符，特别是**中文字符。**

#### 字符串拼接

使用 + 拼接性能最差，strings.Builder，bytes.Buffer 相近，strings.Buffer 更快

![image-20230124203352486 ](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb32e9e83a3748b58687dca0546e9a44~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

![image-20230124203506740 ](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9349abba07504127bffa0ede6dffc314~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

原因：

- 字符串在 Go 语言中是不可变类型，占用内存大小是固定的
- 使用 + 每次都会重新分配内存
- strings.Builder，bytes.Buffer 底层都是 []byte
- 内存扩容策略，不需要每次拼接重新分配内存



### 8. tag 和反射

反射可以在==运行期间==，操作任意类型的对象。

可以通过`TypeOf`方法获得对象类型。通过`ValueOf`获得对象值。

Go 中解析的 tag 是通过反射实现的。

tag 可以理解为 struct 字段的注解，可以用来定义字段的一个或多个属性。框架/工具可以通过反射获取到某个字段定义的属性，采取相应的处理方式。tag 丰富了代码的语义，增强了灵活性。



### 9. 空 struct{} 的用途

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



### 11. goroutine 内存泄漏（协程泄露）

> 一般我们常说的内存泄漏是指**堆内存的泄漏**。
>
> 堆内存是指程序从堆中分配的，大小任意的(内存块的大小可以在程序运行期决定)内存块，使用完后必须释放的内存。

#### 原因

- Goroutine 内进行 channel/mutex 等读写操作被一直阻塞。 

- Goroutine 内的业务逻辑进入死循环，资源一直无法释放。 

- Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待。

#### 场景

- channel 如果忘记初始化，那么无论你是读，还是写操作，都会造成阻塞。 
- channel 发送数量 超过 channel 接收数量，就会造成阻塞
- channel 接收数量 超过 channel 发送数量，也会造成阻塞
- http request body未关闭，goroutine不会退出
- 互斥锁忘记解锁
- sync.WaitGroup 使用不当，wg.Add 与 wg.Done 数量不匹配

#### 排查

单个函数：调用 `runtime.NumGoroutine` 方法来打印 执行代码前后 Goroutine 的运行数量，进行前后比较，就能知道有没有泄露了。

生产/测试环境：使用`PProf`实时监测Goroutine的数量。

> 记录一次实战例子
>
> - 使用 prometheus（普罗米修斯）+ grafana 平台对内存和CPU的监控
> - 在代码中引入pprof，然后开启一个端口来监听http请求，设置handle为nil
> - 使用go命令调用pprof工具来查看内存或cpu状态`go tool pprof http://localhost:6060/debug/pprof/heap`

#### 解决协程泄露的常见方式

如协程会无限地向通道`ch`发送数据，这个协程永远不会退出，导致了协程泄露。

- 使用带超时的操作，比如`select`语句配合`time.After`。
- 使用context包来传递取消信号。



### 12. init() 函数是什么时候执行的？

import –> const –> var –> init() –> main()

import : 由 runtime 初始化每个导入的包，初始化顺序不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化。



### 13. 哪些类型不可以作为字典的键类型？

切片Slice、映射Map、函数类型Function以及包含不可比较类型的结构体。



### 14. struct 能不能比较？

https://juejin.cn/post/6881912621616857102

- 同一个 struct 的两个实例可比较也不可比较。当结构包含==Slice，Map，Function==不可直接比较成员变量时不可直接比较（==），但是可以借助 `reflect.DeepEqual` 函数来对两个变量进行比较。
- 两个不同的 struct 的实例可比较也不可比较。可以通过强制转换来比较。如果成员变量中含有不可比较成员变量，即使可以强制转换，也不可以比较。



### 15. select

1. `select` 语句只能用于通道操作，用于在多个通道之间进行选择，以监听通道的就绪状态，而不是用于其他类型的条件判断。
2. `select` 语句可以包含多个 `case` 子句，每个 `case` 子句对应一个通道操作。当其中任意一个通道就绪时，相应的 `case` 子句会被执行。
3. 如果多个通道都已经就绪，`select` 语句会随机选择一个通道来执行。这样确保了多个通道之间的公平竞争。
4. `select` 语句的执行可能是阻塞的，也可能是非阻塞的。如果没有任何一个通道就绪且没有默认的 `default` 子句，`select` 语句会阻塞，直到有一个通道就绪。如果有 `default` 子句，且没有任何通道就绪，那么 `select` 语句会执行 `default` 子句，从而避免阻塞。

用途：多路复用、非阻塞通信、超时处理` case <-time.After(3 * time.Second)`



### 16. go语言的锁有几种类型？

1. 互斥锁（sync.Mutex）
2. 读写锁（sync.RWMutex）
3. 等待组（sync.Waitgroup）
4. 一次性锁（sync.Once）
5. 条件变量（sync.Cond）
6. 原子操作（sync/atomic）



### 17. CAS 机制

CAS（Compare And Swap）是一种常见的并发控制机制，作为原子操作提供的。CAS是一种无锁的技术，当多个线程尝试使用共享数据时，CAS能够检测到其他线程是否已经改变了这个数据，这是一种解决并发问题的策略。

CAS操作包含三个参数：内存位置（V）、期望值 A 和新值 B。这个操作的流程如下：

1. 比较内存地址 V 中存储的值与期望值 A 是否相等。
2. 如果相等，则将内存地址 V 中存储的值更新为新值 B。这个比较和替换是在一个不可中断的操作中完成的。
3. 如果不相等，则说明其他线程已经修改了内存地址 V 中存储的值，此时 CAS 操作失败，需要重新尝试。

CAS 在 Golang 中是以==共享内存==的方式来实现的一种同步机制`atomic.CompareAndSwapInt32(&addr, oldValue, oldValue+delta)`

原子操作一般是由==硬件底层==支持的，而锁则是由操作系统层面来实现的。比起使用锁，使用 CAS原子操作这个过程是不会形成临界区和创建临界区的，大大减少了同步对程序性能的影响，所以性能要高效一些。

但原子也有一定的弊端，在被操作值频繁变更的情况下，很可能失败，需要一直重试直到成功为止。



### 18. 面向对象

面向对象是一种思想，可以将复杂问题简单化，让我们从执行者变成指挥者面向对象的三大特性：==封装/继承/多态==。

- 封装：把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。

  - Golang中没有 public 和 private 的关键字，如果类名大写，则其他包也能够访问；如果标识符以小写字母开头，则它是未导出的，只能在同一个包内访问。导出的标识符（大写字母开头）不仅仅限于结构体名字，还包括结构体中的字段名、函数名等。这种命名规范有助于代码的可维护性和封装性。
	- setName 方法正确地修改实例，需要将其改为**指针接收者**，而不是**值接收者**。
  - 在 Golang 开发中并没有特别强调封装，这点并不像Java。
- 继承：从一个已知的类中派生出一个新的类，新类可以拥有已知类的行为和属性，并且可以通过重写（override）来增强已知类的能力。
  - 在 Golang 中，如果一个 struct 嵌套了另一个匿名结构体，那么这个结构体可以直接访问匿名结构体的字段和方法，从而实现了继承特性。
  - 继承多个构造体的情况，在没有重写的情况下会报错。
- 多态：多态的本质就是一个程序中存在多个同名的不同方法，主要通过==子类对父类的覆盖==、==类里对方法的重载（overloading）==、将子类作为父类对象来使用、实现接口等方法实现的。
  - 在 Go语言，多态是通过接口实现的。
  - 在 Golang 中没有方法或函数的重载。

$\textcolor{Cyan}{继承是子类获得父类的成员。} $

$\textcolor{Green}{重写是继承后重新实现父类的方法。} $

$\textcolor{red}{重载是在一个类里一系列参数不同名字相同的方法。} $

$\textcolor{Purple}{多态则是为了避免在父类里大量重载引起代码臃肿且难于维护。} $

#### 面向对象和面向过程的区别

**面向过程**：**面向过程性能比面向对象高**。因为类的调用需要实例化，开销较大，比较消耗资源，所以当性能是最重要的考量因素的时候，比如单片机、嵌入式开发、Linux等时都采用面向过程开发。

**面向对象**：**面向对象易维护，易复用、易拓展**。因为面向对象有封装、继承、多态等特性，所以可以设计出低耦合的系统，可以使系统更加灵活，也更加易于维护



## Goroutine

goroutine 是由 Go 运行时（runtime）管理的协程（用户态的轻量级线程），而不是操作系统管理。

相比较于操作系统线程，goroutine 的资源占用和使用代价都要小得多，可以创建几十个、几百个甚至成千上万个goroutine 也不会造成系统资源的枯竭，Go 的运行时负责对 goroutine 进行管理。

goroutine 本身只是一个数据结构，真正让 goroutine 运行起来的是**调度器**。

Go 实现了一个用户态的调度器（GMP模型）

### GMP模型

属于是多对多模型

https://learnku.com/articles/41728

- G：goroutine协程，使用go关键字可以创建一个golang协程。

- M：thread线程，协程必须放在线程上执行。

- P：processor处理器，包含运行Go代码的必要资源，也有调度goroutine的能力，最多有 GOMAXPROCS 个。

  ==选择待执行G，井调度到线程M上执行。==

  ==M必须拥有P，才能执行G中的代码，P负责G的调度。==

引入 P 的原因是：GM 模型 M 获取 G 时都要加锁，会因为频繁加锁解锁而发生等待，影响程序并发性能。

==M 与 P 是一对一的关系，P 与 G 则是一对多的关系。==

<img src="https://www.liwenzhou.com/images/Go/concurrence/gpm.png" alt="gpm" style="zoom: 67%;" />

> ##### goroutine 调度流程
>
> 全局队列：存放等待运行的 G。
>
> P 的本地队列：存放等待运行的G，数量不超过256个。新建 G 时，G 优先加入到 P 的本地队列，
>
> 新建一个 goroutine 的时候，优先放到 P 的本地队列中，如果本地队列满了会批量移动部分 G 到全局队列。
> 当 P 的本地队列为空时，M 也会尝试从全局队列拿一些 G 放到 P 的本地队列 。
>
> M：线程想运行任务就得获取 P，从 P 的本地队列获取 G，P 队列为空时，M 也会尝试从全局队列拿一批 G 放到 P 的本地队列，或从其他 P 的本地队列偷一半放到自己 P 的本地队列。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。（==work stealing 机制==）
>
> 当 线程 M 执行某一个 G 时候如果发生了阻塞操作，释放绑定的 P，把 P 转移给其他空闲的 M 执行（没有空闲的 M 就会创建一个新的线程）。（==hand off 机制==）
>
> 当 M 调用结束时候，这个 G 会尝试获取一个空闲的 P ，并放入到这个 P 的本地队列。如果获取不到 P，那么这个线程 M 变成休眠状态， 加入到空闲线程中，然后这个 G 会被放入全局队列中。
>
> 
>
> 一个 P 最多包含 257 个 G ：本地队列 runq（256个）+优先级最高 runnext（1个）
>
> runnext：一个 P 如果还有剩余时间片，那么 P 中runnext字段中的goroutine会继承这部分剩余时间并执行。

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/ddyl519.jpg)

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



#### GMP 的 G 有哪些状态

Goroutine 在 Go 运行时（runtime）系统中可以有以下 9 种状态：

1. Gidle：Goroutine 处于空闲状态，即没有被创建或者被回收
2. ==Grunnable==：Goroutine 已经准备就绪，可以被调度器调度执行，但是还未被选中执行
3. ==Grunning==：Goroutine 正在执行中，被赋予了M和P的资源
4. ==Gsyscall==：Goroutine 发起了系统调用，进入系统调用阻塞状态
5. ==Gwaiting==：Goroutine 被阻塞等待某个事件的发生，比如等待 I/O、等待锁、等待 channel 等
6. Gscan：GC正在扫描栈空间
7. Gdead：没有正在执行的用户代码
8. Gcopystack：栈正在被拷贝，没有正在执行的代码
9. ==Gpreempted==：Goroutine 被**抢占**，即在运行过程中被调度器中断，等待重新唤醒

对应操作系统中三大进程状态

- 就绪态：Grunnable
- 运行态：Grunning
- 等待态：Gwaiting、Gsyscall、Gpreempted

![img](http://www.uml.org.cn/j2ee/images/20220429466.jpg)



#### 调度时机（什么时候进行切换/执行）

https://juejin.cn/post/7330052230472663055

- 主动调度（协作式调度）：业务程序主动调用 runtime.Gosched 函数让出 CPU 而产生的调度

  > 1. 将 G 状态从 Grunning（运行态） 切换为 Grunnable（就绪态）；
  > 2. 释放 M 当前运行的 G；
  > 3. 将 G 放入全局运行队列 sched.runq；
  > 4. 调用 schedule 函数获取 G 进行调度

- 被动调度：G 执行业务代码时，因条件不满足需要等待，而发生的调度

  - 等待接收 channel 数据，但 channel 又没有数据的时候，就会发生 G 阻塞

- 抢占式调度：由于 sysmon 检测 G 运行时间太长(比如sleep，死循环)或长时间处于系统调用之中，被调度器剥夺运行权，从而发生的调度。

  - **基于协作的抢占式调度**
    - 针对 G 运行时间太长（一般是 10ms）的情况，retake 会设置抢占标志
    - 当发生==函数调用==时，会检查抢占标记，如果有抢占标记就会触发抢占让出cpu
    - 只在有函数调用的地方才能插入“抢占”代码，死循环并没有给编译器插入抢占代码的机会
  - **基于信号的抢占式调度**（异步抢占）
    - 针对 G 运行时间太长（一般是 10ms）的情况，retake 会在支持异步抢占的系统内，直接发送信号给 M，M 收到信号后实施异步抢占
    - 解决由密集循环导致的无法抢占的问题

  

#### 调度遇到阻塞的情况

Goroutine 阻塞一般有：系统调用（syscall），网络IO，协程挂起，执行计算四种。

- **系统调用（syscall）**：G会阻塞内核线程M，此时M运行着G先跟P分离，P寻找其他空闲的M进行绑定；等G的系统调用完成后，G将重新分配到全局队列，M也会继续寻找绑定空闲的P。 ==被动调度==

- **网络IO**：网络轮询器（NetPoller）来处理网络请求和 IO 操作的问题，其后台通过 kqueue（MacOS），epoll（Linux）或 iocp（Windows）来实现 IO 多路复用。不会导致M被阻塞，仅阻塞G。 ==被动调度==

- **协程挂起**：当G遇到channel阻塞，sleep等阻塞后，G将挂起，不阻塞M，挂起完成过后才会放到队列里面，等待P的分配。 ==被动调度==

- **执行计算**：当G遇到执行程序比较长时，超过10ms后会让出CPU执行权，回到队列等待，不阻塞M。 ==抢占式调度==



### 调度器的生命周期

![17-pic-go调度器生命周期.png](https://gitee.com/Transmigration_zhou/pic/raw/master/j37FX8nek9.png!large)

> M0 : 启动后编号为0的主线程，这个M对应的实例会在全局变量runtime.m0中，不需要在heap上分配，==M0负责执行初始化操作和启动第一个协程G，之后M0便和普通M一样。==
>
> G0：每次启动一个M都会第一个创建的goroutine，==G0仅负责调度G，G0不会被调度程序抢占，每个M都有一个自己的G0。==在调度或系统调用时会使用G0的栈空间，全局变量的G0是M0的G0。



### goroutine 发生重新调度的场景

- 阻塞 I/O
- select操作
- 阻塞在channel
- 等待锁
- 主动调用 runtime.Gosched()



### goroutine 的抢占式调度

https://jingwei.link/2019/05/26/golang-routine-scheduler.html

goroutine通过**抢占机制**来打断长时间占用 CPU 资源的 goroutine ，发起重新调度。

**抢占机制**：Golang 运行时（runtime）中的系统监控线程 sysmon 可以找出“长时间占用”的 goroutine，从而“提醒”相应的 goroutine 该中断了。





## 底层数据结构

### 1. Slice

#### 数据结构

https://zhuanlan.zhihu.com/p/449851884

切片的本质就是对底层数组的封装，它包含了三个信息：底层数组的指针、切片的长度（len）和切片的容量（cap）。

切片的长度（len）就是它所包含的元素个数。

切片的容量（cap）是从它的第一个元素开始数，到其底层数组元素末尾的个数。

> 我们可以把容量当做成**总长度减去左指针走过的元素值**，比如：
>
> cap(s) = 6
>
> s[:0] ——> cap = 6 - 0 =6
>
> s[2:] ——> cap = 6 - 2 = 4

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

   [深拷贝与浅拷贝](# 4. 深拷贝与浅拷贝)
   
3. 切片不支持比较操作，数组内元素类型能比较就支持比较操作

#### 内存泄漏

由于slice的底层是数组，很可能数组很大，但slice所取的元素数量却很小，这就导致数组占用的绝大多数空间是被浪费的。

### 2. Map

#### 数据结构

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

1. 负载因子 = 哈希表存储的元素个数 / 桶个数（count/2^B^）> 6.5，触发双倍扩容

2. 溢出桶太多，触发等量扩容

   - 当桶数量 <= 2^15^ 时（B<=15），如果溢出桶总数(noverflow) >= 桶数量
- 当桶数量 > 2^15^ 时，如果溢出桶总数 >= 2^15^

> 双倍扩容：分配新桶是旧桶的两倍，然后旧buckets数据搬迁到新的buckets。
>
> 等量扩容：不扩大容量，buckets数量维持不变，重新做一遍类似双倍扩容的搬迁动作，把松散的键值对重新排列一次，使得同一个 bucket 中的 key 排列地更紧密，节省空间，提高 bucket 利用率，减少溢出桶的使用，进而保证更快的存取。
>
> 负载因子越大，填入的元素越多，空间利用率就越高，但发生哈希冲突的几率就变大。
> 负载因子越小，填入的元素越少，冲突发生的几率减小，但空间浪费也会变得更多，而且还会提高扩容操作的次数。

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

#### 未初始化的map

不能对未初始化的map进行赋值，这样将会抛出一个异常。

可以对未初始化的map进行取值，但取出来的东西是空。

通过`map == nil`来判断

#### map 中删除一个 key，它的内存会释放么

如果删除key对应的value是值类型，如int，float，bool，string以及数组和struct，map的内存不会自动释放。

如果删除key对应的value是引用类型，如指针，slice，map，chan等，map的释放的内存是value的内存占用。

将map设置为nil后，内存被回收。

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

channel 是一个队列，遵循先进先出的原则，负责协程之间的通信。

#### 数据结构

channel变量是一个存储在函数栈帧上的指针，指向堆上的hchan结构体。

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
> buf：用来保存goroutine之间传递数据的循环链表 
> sendx和recvx：用来记录此循环链表当前发送或接收数据的下标值。
> sendq 和 recvq：用于保存向该chan发送和从改chan接收数据的goroutine的队列。
> lock：保证channel写入和读取数据时线程安全的锁。
>
> 初始hchan结构体中的buf为空，sendx和recvx均为0。
> 当向ch里发送数据时，首先会对buf加锁，然后**将数据copy到buf中**，然后sendx++，然后释放对buf的锁。
> 当消费ch的时候，会首先对buf加锁，然后将buf中的**数据copy到task变量对应的内存里**，然后recvx++，并释放锁。



#### 操作

发送（写）：发送操作包括了“复制元素值”和“放置副本到通道内部”这两个步骤。即：进入通道的并不是操作符右边的那个元素值，而是它的副本。`ch <- x`

接收（读）：接收操作包含了“复制通道内的元素值”、“放置副本到接收方”、“删掉原值”三个步骤。`<- ch` `v, ok := <-ch`

关闭：关闭 channel 会产生一个广播机制，所有向 channel 读取消息的 goroutine 都会收到消息。`close(ch)`


| 操作 \ 状态 | 未初始化         | 关闭                                                         | 正常             | 没值     | 满       |
| ----------- | ---------------- | ------------------------------------------------------------ | ---------------- | -------- | -------- |
| 关闭        | panic            | panic                                                        | 关闭成功         | 关闭成功 | 关闭成功 |
| 发送（写）  | 永远阻塞导致死锁 | panic                                                        | 阻塞或者发送成功 | 发送成功 | 阻塞     |
| 接收（读）  | 永远阻塞导致死锁 | 能一直读到内容，但是读到的内容根据通道内关闭前是否有元素而不同。<br />缓冲区为空则为零值，并且会返回一个为 false 的 ok-idiom。<br />缓冲区不为空则正常读，ok-idiom 为 true。 | 阻塞或者接收成功 | 阻塞     | 接收成功 |

如果多个 goroutine 都监听同一个 channel，那么 channel 上的数据都可能随机被某一个 goroutine 取走进行消费。

如果多个 goroutine 都监听同一个 channel，如果这个 channel 被关闭，则所有 goroutine 都能收到退出信号。

**向channel写数据的流程：**

1. 如果等待接收队列recvq不为空，说明缓冲区中没有数据或者没有缓冲区，此时直接从recvq取出G，并把数据写入，最后把该G唤醒，结束发送过程；
2. 如果缓冲区中有空余位置，将数据写入缓冲区，结束发送过程；
3. 如果缓冲区中没有空余位置，将待发送数据写入G，将当前G加入sendq，进入睡眠，等待被读goroutine唤醒。

**向channel读数据的流程：**

1. 如果等待发送队列sendq不为空且没有缓冲区，直接从sendq中取出G，把G中数据读出，最后把G唤醒，结束读取过程；
2. 如果等待发送队列sendq不为空，此时说明缓冲区已满，从缓冲区中首部读出数据，把G中数据写入缓冲区尾部，把G唤醒，结束读取过程；
3. 如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程；
4. 如果缓冲区中没有数据，将当前goroutine加入recvq，进入睡眠，等待被写goroutine唤醒。

##### channel发送和接收操作的特点 

1. 一个通道相当于一个先进先出（FIFO）的队列：也就是说，通道中的各个元素值都是严格地按照发送的顺序排列的，先被发送通道的元素值一定会先被接收。
2. 对于同一个通道，发送操作之间和接收操作之间是互斥的：同一时刻，对同一通道发送多个元素，直到这个元素值被完全复制进该通道之后，其他针对该通道的发送操作才可能被执行。接收也是如此。
3. 发送操作和接收操作中，对元素值的处理是不可分割的：前面我们知道发送一个值到通道，是先复制值，再将该副本移动到通道内部，“不可分割”指的是发送操作要么还没复制元素值，要么已经复制完毕，绝不会出现只复制了一部分的情况。接收也是同理，在准备好元素值的副本之后，一定会删除掉通道中的原值，绝不会出现通道中仍有残留的情况。
4. 发送操作和接收操作在完全完成之前会被阻塞：发送操作包括了“复制元素值”和“放置副本到通道内部”这两个步骤。在这两个步骤完全完成之前，发起这个发送操作的那句代码会一直阻塞在那里，在它之后的代码不会有执行的机会，直到阻塞解除。

#### 有缓冲和无缓冲的channel区别

channel 分为无缓冲的 channel 和带缓冲的 channel。当容量为 0 时，该通道为无缓冲通道，当容量大于 0 时，该通道为带缓冲的通道。

```golang
ch := make(chan int)    //无缓冲
ch := make(chan int, 3) //带缓冲
```

非缓冲通道和缓冲通道有着不同的数据传递方式：

- 非缓冲通道：无论是发送操作还是接收操作，一开始执行就会被阻塞，直到配对的操作也开始执行，才会继续传递。即：只有收发双方对接上了，数据才会被传递。数据直接从发送方复制到接收方。非缓冲通道传递数据的方式是同步的。
- 缓冲通道：如果通道已满，对它的所有发送操作都会被阻塞，直到通道中有元素值被接收走。反之，如果通道已空，那么对它的所有接收操作都会被阻塞，直到通道中有新的元素值出现。元素值会先从发送方复制到缓冲通道，之后再由缓冲通道复制给接收方。缓冲通道传递数据的方式是异步的。

#### 双向通道和单向通道

channel 也分为双向通道和单向通道。

- `chan T`表示一个元素类型为`T`的双向通道类型，允许从此类型的值中接收和向此类型的值中发送数据。
- `chan <- T`表示一个元素类型为`T`的单向发送通道类型，不允许从此类型的值中接收数据。
- `<- chan T`表示一个元素类型为`T`的单向接收通道类型，不允许向此类型的值中发送数据。

双向通道`chan T`的值可以被隐式转换为单向通道类型`chan<- T`和`<-chan T`，但反之不行（即使显式也不行）。

#### 使用场景

- 停止信号监听

  关闭 channel 或向 channel 发送一个元素，使接收方通过 channel 获得信息后做相应的操作

- 定时任务（select语句）

  - 超时控制
  - 定期执行某个任务

- 解耦生产方和消费方

  服务启动时，启动 n 个 worker，作为工作协程池，这些协程工作在一个 for 无限循环里, 从某个 channel 消费工作任务并执行

- 控制并发数

  `var limit = make(chan struct{}, 3)`
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

> https://www.jianshu.com/p/f70519a54892
>
> 互斥锁是一种独占锁，当线程A加锁成功后，此时互斥锁已经被线程A独占了，只要线程A没有释放手中的锁，线程B就会失败，就会释放掉CPU给其他线程，线程B加锁的代码就会被阻塞。
>
> 自旋锁通过CPU提供的CAS，在用户态完成加锁和解锁操作，不会主动产生线程上下文切换，所以相比互斥锁来说，会快一些开销小一些。
>
> **对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，"自旋"一词就是因此而得名。**
>
> 自旋锁可能存在的2个问题：
>
> 1. **试图递归地获得自旋锁必然会引起死锁**：递归程序的持有实例在第二个实例循环，以试图获得相同自旋锁时，不会释放此自旋锁。
>    在递归程序中使用自旋锁应遵守下列策略：递归程序决不能在持有自旋锁时调用它自己，也决不能在递归调用时试图获得相同的自旋锁。
> 2. **过多占用cpu资源。**如果不加限制，由于申请者一直在循环等待，因此自旋锁在锁定的时候。如果不成功，不会睡眠，会持续的尝试,单cpu的时候自旋锁会让其它process动不了。因此，一般自旋锁实现会有一个参数限定最多持续尝试次数。超出后，自旋锁放弃当前 time slice，等下一次机会。
>
> 由此可见，**自旋锁比较适用于锁使用者保持锁时间比较短的情况**。正是由于自旋锁使用者一般保持锁时间非常短，因此选择自旋而不是睡眠是非常必要的，自旋锁的效率远高于互斥锁。

Go sync包提供了两种锁类型：互斥锁 sync.Mutex 和读写互斥锁 sync.RWMutex，都属于悲观锁。

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

#### Mutex模式

- 正常模式(非公平锁) 

  - 在正常模式下，一个尝试加锁的 goroutine 先通过自旋的方式尝试通过原子操作获取锁，如果几次自旋之后仍然不能获得锁，则插入到队列的尾部排队等待。
  - 所有的等待者按照先入先出的顺序排队，当锁被释放，被唤醒的 goroutine 并不会直接拥有锁，而是要和处于自旋阶段，尚未排队等待的 goroutine 进行竞争。这些 新来的 goroutine 更有优势，因为它们正在 CPU 中运行，而且它们的数量可以有很多，而被唤醒的 goroutine 只有一个，所以被唤醒的 goroutine 很大概率拿不到锁，这中情况它会被插入到队列的头部。
  - 当一个 goroutine 等待锁时间超过 1 ms 时，这个 Mutex 就进入到了饥饿模式。(==正常锁-->饥饿锁触发条件==)

  优点：高吞吐量

  缺点：尾端延迟（队列尾部的 goroutine 长时间抢不到锁）

- 饥饿模式(公平锁)

  - 在饥饿模式下，Mutex 的拥有者将直接把锁交给队列最前面的等待者。新来的 goroutine 不会自旋，也不会尝试获取锁，它们会直接插入到等待队列的尾部。
  - 恢复正常模式的条件（满足其一）：

    1. 这个等待者的等待时间小于1ms
    2. 这个等待者是最后一个等待者，等待队列已经空了，后面没有饥饿的 goroutine

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



### 5. RWMutex

RWMutex 也称为读写互斥锁，读写互斥锁就是读取/写入互相排斥的锁。它可以由任意数量的读取操作的 goroutine 或单个写入操作的 goroutine 持有。读写互斥锁 `RWMutex` 类型有五个方法，`Lock`，`Unlock`，`Rlock`，`RUnlock` 和 `RLocker`。其中，RLocker 返回一个 Locker 接口，该接口通过调用 `rw.RLock` 和 `rw.RUnlock` 来实现 Lock 和 Unlock 方法。

相比互斥锁，读写互斥锁在高并发读的场景下可以提高并发性能，但在高并发写的场景下仍然存在性能瓶颈。

#### 实现原理

RWMutex 通过 readerCount 的正负来判断当前是处于读锁占有还是写锁占有，负数则代表当前正在进行写锁占有。

当有一个写锁的时候，会将读锁数量设置为负数 1<<30，目的是让新进入的读锁等待写锁之后释放通知读锁。

在处于写锁占有状态后，会将此时的 readerCount 赋值给 readerWait，表示要等前面 readerWait 个读锁释放完才算完整的占有写锁，才能进行后面的独占操作。

读锁释放的时候， 会对 readerWait 对应减一，直到为 0 值，就可以唤起写锁了。

并且在写锁占有后，即时有新的读操作加进来， 也不会影响到 readerWait 值了，只会影响总的读锁数目：readerCount。



### 6. WaitGroup

WaitGroup 解决的就是并发-等待的问题，用于等待一组 goroutine 的完成。

#### 用法

- `Add(delta int)`：向计数器添加值（delta 为负数表示减去相应值）
- `Done()`：减少计数器的值，相当于 Add(-1)
- `Wait()`：阻塞调用它的 goroutine，直到计数器的值变为0

#### 实现原理

- WaitGroup 主要维护了 2 个计数器，一个是请求计数器 counter，一个是等待计数器 waiter count，二者组成一个 64bit 的值（state），请求计数器占高 32bit，等待计数器占低 32bit。
  - counter：当前还未执行结束的 goroutine 计数器
  - waiter count：有多少个协程调用了 Wait 方法在等待所有的 counter 执行完毕，即有多少个等候者
- 每次 Add 修改请求计数器的值（v），当 v 等于 0 时，释放waiter的数量。
- Wait 主要有两个作用：1. 累加 waiter count 2. 阻塞等待信号量



### 7. Cond

和 WaitGroup 相反，要求一组子协程等待主协达到某个状态时才继续运行。

#### 用法

- Wait：会把当前协程放入Cond 的等待队列中并阻塞，直到被 Signal 或者 Broadcast 方法从等待队列中移除并唤醒，用于子协程阻塞。
  - Wait 内部会先调用`c.L.Unlock()`，所以调用 Wait 函数前==需要先加锁==
  - 由于 Wait 函数被唤醒时存在虚假唤醒等情况，导致唤醒后条件依然不成立，因此需要使用 for 语句循环进行等待，知道条件成立为止。
- Signal：主协程唤醒等待队列中的==一个==子协程，先唤醒最先阻塞的子协程，被唤醒的子协程继续执行。
- Broadcast：主协程唤醒等待队列中的==全部==协程，所有子协程继续执行。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var start = false
var done = false

func main() {
	wg := sync.WaitGroup{}
	wg.Add(10)
	m := &sync.Mutex{}
	c := sync.NewCond(m)
	for i := 0; i < 10; i++ {
		go Soldiers(i, c, &wg)
	}
	go Waiter(c)
	Officer(c)
	wg.Wait()
	fmt.Println("所有大兵干完饭")

	CleanUp(c)
	time.Sleep(3 * time.Second)
	fmt.Println("打扫结束")

}

func CleanUp(c *sync.Cond) {
	c.L.Lock()
	done = true
	c.L.Unlock()
	c.Signal()
}

func Officer(c *sync.Cond) {
	fmt.Printf("长官准备中....\n")
	time.Sleep(time.Second * 5)
	c.L.Lock()
	start = true
	c.L.Unlock()
	// 唤醒所有的等待线程
	c.Broadcast()
}

func Soldiers(i int, c *sync.Cond, wg *sync.WaitGroup) {
	defer wg.Done()
	c.L.Lock()
	fmt.Printf("大兵%d号等待干饭....\n", i)
	for !start {
		// wait 解锁lock 同时挂起goroutine
		// 当wait的goroutine被唤醒的时候，会重新将锁加上
		c.Wait()
	}
	fmt.Printf("大兵%d号开始干饭....\n", i)
	c.L.Unlock()

}

func Waiter(c *sync.Cond) {
	c.L.Lock()
	for !done {
		c.Wait()
	}
	fmt.Println("用餐结束，开始打扫....")
	c.L.Unlock()
	return
}
```



### 8. sync.Once

Once 可以用来执行且仅仅执行一次动作，常常用于单例对象的初始化场景，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测试资源。

```go
type Once struct {
	done uint32
	m    Mutex
}
```

done 是标识位，用于判断方法 f 是否被执行完，done 的初始值为 0，当 f 执行结束时，done 被设为 1。

m 做竞态控制，当 f 第一次执行还未结束时，通过 m 加锁的方式阻塞其他 once.Do 执行 f。

````go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
````

Once 只暴露了一个方法 Do，你可以多次调用 Do 方法，但是只有第 一次调用 Do 方法时 f 参数才会执行，这里的 f 是一个无参数无返回值的函数。



### 9. sync.Pool

sync.Pool 是一个临时对象存储池。

它可以缓存对象暂时不用但是之后会用到的对象，并且不需要重新分配内存。这在很大程度上降低了`GC`的压力，并且提高了程序的性能。

sync.Pool 获取对象的顺序是不确定的，并不保证对象的获取顺序与放入池中的顺序一致。

#### 用法

```go
type Person struct {
    Name string
}

var personalPool = sync.Pool{
    New: func() interface{} {
        return &Person{}
    },
}

func main() {
    newPerson := personalPool.Get().(*Person)
    newPerson.Name = "Jack"
    personalPool.Put(newPerson)
}
```

#### 原理 

sync.Pool有两个容器来存储对象，分别是：local 和 victim。

每次垃圾回收的时候，在 victim 中的对象会被清理掉，而在 local中的对象会被移动 victim 当中，并把 local 设置为空。

新对象是放在 local 当中的，调用`pool.Put`也是将对象放在 local 当中的。

调用`pool.Get`时，会先从 victim 中获取，如果没有找到，则就会从 local 中获取，如果 local 中也没有，就会执行初始化时的 New Function，否则就返回 nil。



### 10. Context

context 主要用来在 goroutine 之间传递上下文信息，包括：取消信号、超时时间、截止时间、k-v 等。

context 用来解决 goroutine 之间退出通知、元数据传递的功能。

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- `Deadline`返回绑定当前`context`的任务被取消的截止时间；如果没有设定期限，将返回`ok == false`。
- `Done` 当绑定当前`context`的任务被取消时，将返回一个关闭的`channel`；如果当前`context`不会被取消，将返回`nil`。
- `Err` 如果`Done`返回的`channel`没有关闭，将返回`nil`;如果`Done`返回的`channel`已经关闭，将返回非空的值表示任务结束的原因。如果是`context`被取消，`Err`将返回`Canceled`；如果是`context`超时，`Err`将返回`DeadlineExceeded`。
- `Value` 返回`context`存储的键值对中当前`key`对应的值，如果没有对应的`key`，则返回`nil`。

![img](https://static.cyub.vip/images/202009/context_implement.jpg)

| 类型      | 创建方法                     | 功能                                                         |
| --------- | ---------------------------- | ------------------------------------------------------------ |
| emptyCtx  | Background()/TODO()          | 用做 context 树的根节点。一般情况下，会使用 `Background()` 作为根 ctx，然后在其基础上再派生出子 ctx。要是不确定使用哪个 ctx，就使用 `TODO()`。 |
| cancelCtx | WithCancel()                 | 可取消的 context。当调用返回的 `cancel` 函数或关闭父上下文的 `Done` 通道时，返回的 `ctx` 的 `Done` 通道将关闭。 |
| timerCtx  | WithDeadline()/WithTimeout() | 可取消的 context，过期或超时会自动取消。当截止时间到期、调用返回的取消函数时或当父上下文的 `Done` 通道关闭时，返回的上下文的 `Done` 通道将关闭。 |
| valueCtx  | WithValue()                  | 可存储共享信息的context                                      |

#### 应用场景

- 上下文数据的递归获取

  - 从当前层开始逐层的进行向上递归，直至找到某个匹配的key

    ![img](https://pics4.baidu.com/feed/0e2442a7d933c895c6e2fa74a82470f6830200d9.jpeg@f_auto?token=b9114d289fb79e1d48c94869296bcfa4&s=449C30728386414B4EC16CCA0200C0B2)

- 取消的通知

  ![img](https://pics4.baidu.com/feed/a686c9177f3e6709a80faf0341f09c3bf9dc55f8.jpeg@f_auto?token=88202aba78ac480533fb85a6b4a4aaeb&s=45F0AD738BBB4C0948E5E8D10200C0B1)

- 带有超时context

  ![img](https://pics3.baidu.com/feed/3ac79f3df8dcd1004745847208bc4416b8122f41.jpeg@f_auto?token=c15752ac475d4c5d7b765977f7bfafa4&s=04B478328BF260031ED090D80000D0B1)

## 代码题

### 控制并发数

  ```go
func main() {
    var wg sync.WaitGroup

    sem := make(chan struct{}, 2) // 最多允许2个并发同时执行
    taskNum := 10

    for i := 0; i < taskNum; i++ {
        wg.Add(1)
        sem <- struct{}{} // 获取信号
        go func(id int) {
            defer wg.Done()

            defer func() { <-sem }() // 释放信号

            // do something for task
            time.Sleep(time.Second * 2)
            fmt.Println(id, time.Now())
        }(i)
    }
    wg.Wait()
}
  ```

  ```go
// runDynamicTask 
// 最大同时运行maxTaskNum个任务处理数据
// 自定义令牌池维持maxTaskNum个令牌供竞争
func runDynamicTask(dataChan <-chan int, maxTaskNum int) {
    // 初始化令牌池
    tokenPool := make(chan struct{}, maxTaskNum)
    for i := 0; i < maxTaskNum; i++ {
        tokenPool <- struct{}{}
    }

    var wg sync.WaitGroup

    for data := range dataChan {
        // 先获取令牌，如果被消费完则阻塞等待其它任务返还令牌
        <-tokenPool

        wg.Add(1)
        go func(data int) {
            defer wg.Done()

            // 任务运行完成，返还令牌
            defer func() {
                tokenPool <- struct{}{}
            }()

            // do something
            time.Sleep(3 * time.Second)
            fmt.Println(data, time.Now())
        }(data)
    }

    wg.Wait()
}
  ```

#### 协程池

```go
type Pool struct {
	job chan func()
	sem chan struct{}
}

func NewPool(size int) *Pool {
	return &Pool{
		job: make(chan func(), size),
		sem: make(chan struct{}, size),
	}
}

func (p *Pool) NewJob(f func()) {
	select {
	case p.sem <- struct{}{}:
		go func() {
			p.Work(f)
		}()
	case p.job <- f:
	}
}

func (p *Pool) Work(f func()) {
	defer func() {
		fmt.Println("finish work")
		<-p.sem
	}()

	for {
		f()
		f = <-p.job
	}

	//f()
	//for task := range p.job {
	//	task()
	//}
}
```


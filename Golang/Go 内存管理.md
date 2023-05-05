# Go 内存管理

https://juejin.cn/post/7189525739836801085/

## 内存分配机制

Go的内存管理组件主要有：`mspan`、`mcache`、`mcentral`和`mheap`

- 内存管理单元：mspan
- 线程缓存：mcache
- 中心缓存：mcentral
- 页堆：mheap

### 分块

目标：为对象在heap上分配内存

- 提前将内存分块

  - 调用系统调用mmap()向OS申请一大块内存，如4 MB

  - 先将内存分成大块，例如8 KB，称作mspan

  - 再将大块继续划分成**特定大小**的小块，用于对象分配

  - noscan mspan：分配不包含指针的对象——GC不需要扫描

  - scan mspan：分配包含指针的对象——GC需要扫描

  ![A2D4F8AD-60D3-4ED3-86A9-8CF72B6E5220](https://gitee.com/Transmigration_zhou/pic/raw/master/A2D4F8AD-60D3-4ED3-86A9-8CF72B6E5220.png)

对象分配：根据对象的大小，选择最合适的块返回

### 缓存

![8F3C0309-7580-4ABB-9B5A-DC4E826A8F40](https://gitee.com/Transmigration_zhou/pic/raw/master/8F3C0309-7580-4ABB-9B5A-DC4E826A8F40.png)

> G：goroutine协程，使用go关键字可以创建一个golang协程。
>
> M：thread线程，协程必须放在线程上执行。
>
> P：processor处理器，包含运行Go代码的必要资源，也有调度goroutine的能力。

- 每个P包含一个mcache用于快速分配，用于为绑定于p上的g分配对象
- mcache管理一组mspan
- 当mcache中的mspan分配完毕，向mcentral申请带有未分配块的mspan
- 当mspan中没有分配的对象，mspan会缓存在mcentral中，而不是立刻释放并归还给OS
- 所有mcentral的集合则是存放于mheap中的。



### 分配流程

1. 首先通过计算使用的大小规格
2. 然后使用mcache中对应大小规格的块分配。
3. 如果mcentral中没有可用的块，则向mheap申请，并根据算法找到最合适的mspan。
4. 如果申请到的mspan超出申请大小，将会根据需求进行切分，以返回用户所需的页数。剩余的页构成一个新的 mspan 放回 mheap 的空闲列表。
5. 如果 mheap 中没有可用 span，则向操作系统申请一系列新的页（最小 1MB）。



## 内存逃逸机制

在一段程序中，每一个函数都会有自己的内存区域存放自己的局部变量、返回地址等，这些内存会由编译器在栈中进行分配，每一个函数都会分配一个栈桢，在函数运行结束后进行销毁，但是有些变量我们想在函数运行结束后仍然使用它，那么就需要把这个变量在堆上分配，这种从"栈"上逃逸到"堆"上的现象就成为内存逃逸。

在栈上分配的地址，一般由系统申请和释放，不会有额外性能的开销，比如函数的入参、局部变量、返回值等。在堆上分配的内存，如果要回收掉，需要进行 GC，那么GC 一定会带来额外的性能开销。变量一旦逃逸会导致性能开销变大。

编译器会根据变量是否被外部引用来决定是否逃逸：

1. 如果函数外部没有引用，则优先放到栈中
2. 如果函数外部存在引用，则必定放到堆中
3. 如果栈上放不下，则必定放到堆上

查看逃逸的方法`go build -gcflags "-m -l" main.go `

### 产生逃逸的情况

- 函数返回后变量仍被使用的情况

  1. 返回指针

     由于返回时被外部引用，因此其生命周期大于栈，则溢出。

     ```go
     func escapeValue() *int{
         var a int
         a = 1
         return &a
     }
     ```

  2. 在一个切片上存储指针或带指针的值

     数组可能是在栈上分配的，但其引用的值一定是在堆上。

     ```go
     func escapeString() {
     	s := make([]*string, 10) //does not escape
     	a := "aaa"               //moved to heap: a
     	s[0] = &a
     }
     ```

  3. 发送指针或带有指针的值到 channel 中。

     在编译时，是没有办法知道哪个 goroutine 会在 channel 上接收数据，所以编译器没法知道变量什么时候才会被释放。

     ```go
     func escapeChannel() {
        var ch = make(chan *int, 1)
        var a = 10 //moved to heap: a
        var b = &a
        go func() {
           ch <- b
        }()
     }
     ```

  4. 闭包调用

     导致函数返回后函数内变量仍被外部使用

     ```go
     func fibonacci() func() int {
     	a, b := 0, 1
     	return func() int {
     		a, b = b, a+b
     		return a
     	}
     }
     ```

- 变量过大被分配在堆上

  ```go
  s := make([]int64, 8192, 8192)
  ```

   





## 垃圾回收

### 基本概念

- 动态内存
  - 程序在运行时根据需求动态分配的内存：malloc()

- 自动内存管理（垃圾回收）：由程序语言的运行时系统管理动态内存

  - 避免手动内存管理，专注于实现业务逻辑

  - 保证内存使用的**正确性**和**安全性**

- 三个任务

  - 为新对象分配空间
  - 找到存活对象
  - 回收死亡对象的内存空间

> 一些名词解释：
>
> Mutator: 业务线程，分配新对象，修改对象指向关系
>
> Collector: GC 线程，找到存活对象，回收死亡对象的内存空间
>
> ![8E36D2C3-129A-4124-94A2-3462503707BB](https://gitee.com/Transmigration_zhou/pic/raw/master/8E36D2C3-129A-4124-94A2-3462503707BB.png)
>
> Serial GC: 只有一个 collector
>
> ![B5ED539C-50F9-429F-A8DB-4AD8CC708059](https://gitee.com/Transmigration_zhou/pic/raw/master/B5ED539C-50F9-429F-A8DB-4AD8CC708059.png)
> Parallel GC: 并行 GC，支持多个 collectors 同时回收的 GC 算法
>
> ![A0648E5B-C796-4272-8D9D-984DA2535197](https://gitee.com/Transmigration_zhou/pic/raw/master/A0648E5B-C796-4272-8D9D-984DA2535197.png)
>
> Concurrent GC: 并发 GC，支持 mutator(s) 和 collector(s) **同时执行**的 GC 算法
>
> ![3CE351A7-6F9B-46B8-8828-4B7795FA9BDF](https://gitee.com/Transmigration_zhou/pic/raw/master/3CE351A7-6F9B-46B8-8828-4B7795FA9BDF.png)



- 评价GC算法的四个维度
  - 安全性：不能回收存活的对象 ==基本要求==
  - 吞吐率：$ 1 - \frac{GC时间}{程序执行总时间} $ ==花在GC上的时间==
  - 暂停时间：stop the world（STW） ==业务是否感知==
  - 内存开销 ==GC元数据开销==



### GC算法

常见GC算法有引用计数、分代收集、**标记清除**。

#### 标记清除

被回收的条件：不可达对象

过程：

- 标记根对象 (GC roots)

  - 静态变量、全局变量、常量、线程栈等

- 标记：找到可达对象

  - 求指针指向关系的传递闭包：从根对象出发，找到所有可达对象

- 清理：所有不可达对象 ==根据对象的生命周期，使用不同的标记和清理策略==

  - Copying GC：将存活对象复制到**另外的内存空间**

    ![B70869AA-BE18-44E8-BEBE-23A7CF719FDB](https://gitee.com/Transmigration_zhou/pic/raw/master/B70869AA-BE18-44E8-BEBE-23A7CF719FDB.png)

  - Mark-sweep GC：将死亡对象的內存标记为”可分配” （使用 free list 管理空闲内存）

    ![DA73BFB7-48B1-4FCD-B769-CC733F57C38B](https://gitee.com/Transmigration_zhou/pic/raw/master/DA73BFB7-48B1-4FCD-B769-CC733F57C38B.png)

  - Mark-compact GC：移动并整理存活对象（在原内存空间）

    ![A7E3BA6F-91B3-4AA3-B142-2047CDF2A376](https://gitee.com/Transmigration_zhou/pic/raw/master/A7E3BA6F-91B3-4AA3-B142-2047CDF2A376.png)

#### 分代 GC（Generational GC）

*Java中使用*

分代假说：most objects die young ==> 很多对象在分配出来后很快就不再使用了

每个对象都有年龄：经历过GC的次数

目的：对年轻和老年的对象，制定不同的GC策略，降低整体内存管理的开销

- 年轻代(Young generation)
  - 常规对象分配
  - 由于**存活对象很少**，可以采用copying collection
  - GC吞吐率很高
- 老年代(Old generation)
  - **对象趋向于一直活着，反复复制开销较大**
  - 可以采用mark-sweep collection

#### 引用计数

*python,swift,php中使用*

每个对象都有一个与之关联的引用数目，对象被引用时, 引用计数+1，对象被释放时，引用计数-1。如果引用计数为0，销毁对象。

对象存活的条件：当且仅当引用数大于0

优点

- 内存管理的操作被平摊到程序执行过程中
- 内存管理不需要了解runtime的实现细节：C++智能指针

缺点

- 维护引用计数的开销较大：通过原子操作保证对引用计数操作的原子性和可见性
- 无法回收环形数据结构——weak reference
- 内存开销：每个对象都引入的额外内存空间存储引用数目
- 回收内存时依然可能引发暂停



==运行时GC，编译时ARC==



### Go垃圾回收

Go 语言采用的是标记清除算法。并在此基础上使用了三色标记法和写屏障技术，提高了效率。

#### 标记清除

![image-20221016183237039](https://gitee.com/Transmigration_zhou/pic/raw/master/image-20221016183237039.png)

标记清除收集器是跟踪式垃圾收集器，其执行过程可以分成标记（Mark）和清除（Sweep）两个阶段：

- 标记阶段 — 从根对象出发查找并标记堆中所有存活的对象；
- 清除阶段 — 遍历堆中的全部对象，回收未被标记的垃圾对象并将回收的内存加入空闲链表。

缺点

- STW，stop the world；让程序暂停，程序出现卡顿 **(重要问题)**；
- 标记需要扫描整个heap；
- 清除数据会产生heap碎片。

#### 三色标记

标记清除算法的一大问题是在标记期间，需要暂停程序（Stop the world，STW），标记结束之后，用户程序才可以继续执行。为了能够异步执行，减少 STW 的时间，Go 语言采用了三色标记法。

三色标记算法将程序中的对象分成白色、黑色和灰色三类。

- 白色：不确定对象。
- 灰色：存活对象，子对象待处理。
- 黑色：存活对象。

过程

- 每次新创建的对象，所有对象都是标记为“白色”（这一步需 STW ）
- 每次GC回收开始, 会从根节点开始遍历所有对象，把遍历到的对象从白色集合放入“灰色”集合
- 遍历灰色集合，将灰色对象引用的对象从白色集合放入灰色集合，之后将此灰色对象放入黑色集合
- 重复**第三步**, 直到灰色中无任何对象
- 回收所有的白色标记表的对象，也就是回收垃圾



三色标记法因为多了一个白色的状态来存放不确定对象，所以后续的标记阶段可以并发地执行。当然并发执行的代价是可能会造成一些遗漏，因为那些早先被标记为黑色的对象可能目前已经是不可达的了。所以三色标记法是一个 false negative（假阴性）的算法。

三色标记法并发执行仍存在一个问题，即在 GC 过程中，对象指针发生了改变。比如下面的例子：

```
A (黑) -> B (灰) -> C (白) -> D (白)
```

正常情况下，D 对象最终会被标记为黑色，不应被回收。但在标记和用户程序并发执行过程中，用户程序删除了 C 对 D 的引用，而 A 获得了 D 的引用。标记继续进行，D 就没有机会被标记为黑色了（A 已经处理过，这一轮不会再被处理）。

```
A (黑) -> B (灰) -> C (白) 
  ↓
 D (白)
```

为了解决这个问题，Go 使用了内存屏障技术，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，类似于一个钩子。垃圾收集器使用了写屏障（Write Barrier）技术，当对象新增或更新时，会将其着色为灰色。这样即使与用户程序并发执行，对象的引用发生改变时，垃圾收集器也能正确处理了。

一次完整的 GC 分为四个阶段：

- 1）标记准备(Mark Setup，需 STW)，打开写屏障(Write Barrier)
- 2）使用三色标记法标记（Marking, 并发）
- 3）标记结束(Mark Termination，需 STW)，关闭写屏障。
- 4）清理(Sweeping, 并发)



#### GC触发时机

**主动触发：**

- 调用 `runtime.GC()` 方法，触发 GC

**被动触发：**

- 定时触发，该触发条件由 `runtime.forcegcperiod` 变量控制，默认为 2 分钟。当超过两分钟没有产生任何 GC 时，触发 GC
- 根据内存分配阈值触发，该触发条件由环境变量GOGC控制，默认值为100（100%），当前堆内存占用是上次GC结束后占用内存的2倍时，触发GC

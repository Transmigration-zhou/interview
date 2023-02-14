# Go 内存管理

## 自动内存管理

https://juejin.cn/post/7189525739836801085/

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



常见GC算法有引用计数、分代收集、**标记清除**。



### 标记清除

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

### 分代 GC（Generational GC）

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

### 引用计数（ARC）

*python,swift,php中使用*

每个对象都有一个与之关联的引用数目

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



## Go 内存管理及优化

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

每个p包含一个mcache用于快速分配，用于为绑定于p上的g分配对象

mcache管理一组mspan

当mcache中的mspan分配完毕，向mcentral申请带有未分配块的mspan

当mspan中没有分配的对象，mspan会缓存在mcentral中，而不是立刻释放并归还给OS

### 优化

对象分配是**非常高频**的操作：每秒分配 GB 级别的内存

小对象占比较高，Go内存分配路径长，比较耗时。


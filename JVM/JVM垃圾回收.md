# 1.如何判断对象是否可回收？

https://mp.weixin.qq.com/s?__biz=MzI0NDc3ODE5OQ==&mid=2247490893&idx=1&sn=6371d2f5ce19c3bf000bd7150bd76fd8

两种方法，==引用计数法==和==可达性分析法==。

所谓引用计数法就是在对象中添加一个字段作为引用计数器，每当有一个地方引用这个对象时，计数器的值就加一；当引用失效时，计数器值就减一；计数器为零的对象就是可以被回收的对象。

虽然这个算法简单但是无法解决对象之间的循环引用问题。



目前主流的 JVM 用的都是可达性分析算法。

![图片](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220228233644)

就是通过一系列称为 “GC Roots” 的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，如果从 GC Roots 到某个对象不可达时，则说明此对象是可以被回收的。

### 哪些对象的引用可以作为 GC Roots ？

1. 在虚拟机栈中引用的对象
2. 在本地方法栈中引用的对象
3. 在方法区中引用的对象
4. JVM 内部的引用
5. 所有被同步锁（synchronized 关键字）持有的对象
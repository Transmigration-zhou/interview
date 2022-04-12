![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220330152143.png)

[toc]

[源码分析](http://www.cyc2018.xyz/Java/Java%20%E5%AE%B9%E5%99%A8.html#%E4%B8%89%E3%80%81%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)

## List

### ArrayList

#### 扩容机制

[源码分析视频](https://www.bilibili.com/video/BV1YA411T76k?p=13)

- ArrayList中维护了一个Object类型的数组elementData。
- 当创建ArrayList对象时，如果使用的是无参构造器，则初始elementData容量为0。第1次添加，则扩容elementData为10；如果需要再次扩容，则扩容elementData为1.5倍。
- 如果使用的是指定大小的构造器，则初始elementData容量为指定大小， 如果需要扩容，则直接扩容elementData为1.5倍。

##### ArrayList的扩容因子为什么是1.5?

参考[c++中vector的动态扩容](https://blog.csdn.net/qq_44918090/article/details/120583540)

简单来说就是==空间和时间的权衡==。

选择以倍数方式扩容，是因为==以倍数的方式扩容比以等长个数的扩容方式效率高==

为什么是1.5，而不是1.2，1.25，1.8或者1.75，是因为==1.5 可以充分利用移位操作，减少浮点数或者运算时间和运算次数==。`int newCapacity = oldCapacity + (oldCapacity >> 1);`

为什么是1.5，而不是3，4其他倍数，是==因为每次扩容时，前面释放的空间都不能使用==。

前面释放的空间只有1 + 2 = 3，假设第3次空间已经释放才只有1+2+4=7，而第四次需要8个空间，因此无法使用之前已释放的空间，但是按照小于2倍方式扩容，多次扩容之后就可以复用之前释放的空间了。
如果倍数超过2倍(包含2倍)方式扩容会存在：

- 空间浪费可能会比较高，比如：扩容后申请了64个空间，但只存了33个元素，有接近一半的空间没有使用。
- 无法使用到前面已释放的内存。

#### 遍历方式

```java
//方式1：迭代器遍历
System.out.println("--------迭代器遍历--------");
Iterator<String> it=array.iterator();
while(it.hasNext()) {
    System.out.println(it.next());
}
//方式2：普通for循环
System.out.println("-------普通for循环-----------");
for(int i=0;i<array.size();i++) {
    System.out.println(array.get(i));
}
//方式3：增强for循环
System.out.println("-------增强for循环----------");
for(String s:array) {
    System.out.println(s);
}
```

增强for循环，内部使⽤的是迭代器，所以它们的操作对象是数组和可以使用迭代器的集合。遍历时只能查看，⽆法修改、删除、增加。

如果需要对遍历的对象做增删修改的操作，使⽤普通的for循环来操作。



### ArrayList 和 Vector 的区别

- ArrayList 和 Vector 两者底层都是对象数组`Object[ ]`存储。
- Vector 是线程同步（即线程安全，Vector类的操作方法带有synchronized矢量类的操作方法带有同步），开销就比 ArrayList 要大，执行效率低。
- ArrayList 线程不安全，但执行效率高，适用于频繁的查找工作，但在多线程情况下不建议使用 ArrayList 。
- Vector 无参构造默认初始容量为10，ArrayList 无参构造默认初始容量为0，但第一次扩容为10 。
- Vector 每次扩容请求其大小的 2 倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍。



### LinkedList 

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220331162845.png)



### ArrayList 和 LinkedList 的区别

- ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全。
- ArrayList 底层使用的是 Object 数组，LinkedList 底层使用的是 双向链表。
- ArrayList 采用数组存储，所以插入删除的代价很高，需要移动大量元素。 LinkedList 采用链表存储，但插入删除只需要改变指针。
- LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)。
- ArrayList 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。



## Set

### HashSet

#### 底层机制

1. HashSet底层是HashMap

2. 添加一个元素时，先获取元素的hash值（hashCode方法）

3. 对哈希值进行运算，得出一个索引值即为要存放在哈希表中的位置号

4. 找到存储数据表table，看这个索引位置是否已经存放元素

5. 如果该位置上没有其他元素，直接存放

   如果该位置上已经有其他元素，调用equals比较，如果相同，就放弃添加；如果不相同，则以链表的方式添加。

6. 在Java8中，如果一条链表的元素个数超过TREEIFY_THRESHOLD(默认是8)，并且table的大小>=MIN_TREEIFY_CAPACITY(默认64)，就会进行树化(红黑树)

#### 扩容机制

1. HashSet底层是HashMap，第一次添加时，table数组扩容到16,临界值(threshold)是 16 * 加载因子(loadFactor)是 0.75 = 12

2. 如果table数组使用到了临界值12,就会扩容到16 * 2 = 32,新的临界值就是32 * 0.75 = 24，依次类推
3. 在Java8中，如果一条链表的元素个数到达TREEIFY THRESHOLD(默认8)，并且table的大小>= MIN_TREEIFY_CAPACITY(默认64)，就会进行树化(红黑树)，否则仍然采用数组扩容机制（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）

#### HashSet 去重原理

是通过调用元素内部的hashCode和equals方法实现去重。首先调用hashCode方法，比较两个元素的哈希值，如果哈希值不同，认为是两个对象，停止比较。如果哈希值相同，再去调用equals方法，返回true，认为是一个对象。返回false。



### LinkedHashSet

1. LinkedHashSet插入顺序和遍历顺序一致
2. LinkedHashSet底层是一个LinkedHashMap，底层维护了一个数组+双向链表
3. 添加第一次时，直接将数组table扩容到16，存放的结点类型是`LinkedHashMap$Entry`
4. 数组是 `HashMap$Node[]` 存放的元素/数据是 `LinkedHashMap$Entry`类型



### 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的。
- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。`HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）。`LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。
- 底层数据结构不同又导致这三者的应用场景不同。`HashSet` 用于不需要保证元素插入和取出顺序的场景，`LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景，`TreeSet` 用于支持对元素自定义排序规则的场景。



## Map

### HashMap

[源码分析](http://www.cyc2018.xyz/Java/Java%20%E5%AE%B9%E5%99%A8.html#hashmap)

[HashMap源码&底层数据结构分析](https://javaguide.cn/java/collection/hashmap-source-code.html#hashmap-%E7%AE%80%E4%BB%8B)

#### 底层机制

- JDK 1.7 中 HashMap 的底层数据结构是==数组 + 链表==，使用 `Entry` 类存储 Key 和 Value
- JDK 1.8 中 HashMap 的底层数据结构是==数组 + 链表+红黑树==，使用 `Node` 类存储 Key 和 Value

#### 扩容机制

1. HashMap底层维护了Node类型的数组table，默认为null
2. 当创建对象时，将加载因子(loadfactor)初始化为0.75
3. 当添加key-val时，通过key的哈希值得到在table的索引。然后判断该索引处是否有元素，如果没有元素直接添加。如果该索引处有元素，继续判断该元素的key和准备加入的key是否相等。如果相等，则直接替换val；如果不相等需要判断是树结构还是链表结构，做出相应处理。如果添加时发现容量不够，则需要扩容
4. 第1次添加，则需要扩容table容量为16，临界值(threshold)为12(16*0.75)
5. 以后再扩容,则需要扩容table容量为原来的2倍(32)，临界值为原来的2倍，即24，以此类推
6. 在Java8中，如果一条链表的元素个数到达TREEIFY THRESHOLD(默认8)，并且table的大小>= MIN_TREEIFY_CAPACITY(默认64)，就会进行树化(红黑树)

#### put 方法

![ ](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/put方法.png)

#### 遍历方式

[HashMap 的 7 种遍历方式与性能分析！](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw)



#### HashMap 的长度为什么是 2 的幂次方

首先，一般来说，我们常用的 Hash 函数是这样的：`index = HashCode(key) % Length`，但是因为位运算的效率比较高嘛，所以 HashMap 就相应的改成了这样：`index = HashCode(key) & (Length - 1)`。

那么**为了保证根据上述公式计算出来的 index 值是分布均匀的，我们就必须保证 Length 是 2 的次幂**。

> 解释一下：2 的次幂，也就是 2 的 n 次方，它的二进制表示就是 1 后面跟着 n 个 0，那么 2 的 n 次方 - 1 的二进制表示就是 n 个 1。而对于 & 操作来说，任何数与 1 做 & 操作的结果都是这个数本身。也就是说，index 的结果等同于 HashCode(key) 后 n 位的值，只要 HashCode 本身是分布均匀的，那么我们这个 Hash 算法的结果就是均匀的。



#### 以 HashMap 为例，解释一下为什么重写 equals 方法的时候还需要重写 hashCode 方法呢

https://blog.csdn.net/u011583316/article/details/107129546

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220408141909)

在HashMap中的比较key是这样的，先求出key的hashCode(),比较其值是否相等，若相等再比较equals(),若相等则认为他们是相等的。若equals()不相等则认为他们不相等。

- 如果只重写hashCode()不重写equals()方法，当比较equals()时，其实调用的是Object中的方法，只是看他们是否为同一对象（即进行内存地址的比较）。
- 如果只重写equals()不重写hashCode()方法，在一个判断的时候就会被拦下HashMap认为是不同的Key。



#### 如何保证 HashMap 线程安全？

1. 使用 java.util.Collections 类的 `synchronizedMap` 方法包装一下 HashMap，得到线程安全的 HashMap，其原理就是对所有的修改操作都加上 synchronized。方法如下：

```java
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) 
```

2. 使用线程安全的 `HashTable` 类代替，该类在对数据操作的时候都会上锁，也就是加上 synchronized

3. 使用线程安全的 `ConcurrentHashMap` 类代替，该类在 JDK 1.7 和 JDK 1.8 的底层原理有所不同，JDK 1.7 采用数组 + 链表存储数据，使用分段锁 Segment 保证线程安全；JDK 1.8 采用数组 + 链表/红黑树存储数据，使用 CAS + synchronized 保证线程安全。



### ConcurrentHashMap







### HashMap 和 Hashtable的区别

HashMap 是 Hashtable 的轻量级实现

|                     HashMap                     |                      Hashtable                       |
| :---------------------------------------------: | :--------------------------------------------------: |
|                   线程不安全                    |                       线程安全                       |
|               允许有null的键和值                |                 不允许有null的键和值                 |
|                   效率高一点                    |                       效率稍低                       |
|    方法不是Synchronize(同步)的，要提供外同步    |                 方法是Synchronize的                  |
|        有containsvalue和containsKey方法         |                    有contains方法                    |
| 迭代器采用的是Iterator，是快速失败（Fail-Fast） | 迭代器**还**采用Enumeration，是安全失败（Fail-Safe） |

> **快速失败 Fail-Fast**:一旦发生异常，立即停止当前的操作，并上报给上层的系统来处理这些故障。
>
> **安全失败 Fail-Safe**:在故障发生之后会维持系统继续运行。

**初始化容量不同**：HashMap 的初始容量为 16，Hashtable 初始容量为 11。两者的负载因子默认都是 0.75；

**扩容机制不同**：当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1；



###  HashMap 和 TreeMap 区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

实现 `NavigableMap` 接口让 `TreeMap` 有了对==集合内元素的搜索==的能力。

实现`SortedMap`接口让 `TreeMap` 有了对集合中的元素==根据键排序的能力==。默认是按 key 的升序排序，不过我们也可以指定排序的比较器。




[toc]

## 排序

### 怎么对100亿数据进行排序（磁盘上有8G 大的文件，里面都是 int，内存只有2G，我想把文件排序，你打算怎么排？）

1.把这个大文件用==哈希分成1000个小文件==，把100亿个数字对1000取模，模出来的结果在0到999之间，每个结果对应一个文件，所以我这里取的哈希函数是 h = x % 1000，哈希函数取得”好”，能使冲突减小，结果分布均匀。

2.拆分完了之后，得到一些几十MB的小文件，那么就可以放进内存里排序了，可以用快速排序，归并排序，堆排序等等。

3.1000个小文件内部排好序之后，就要把这些内部有序的小文件，合并成一个大的文件，可以==用**二叉堆**来做1000路合并==的操作，每个小文件是一路，合并后的大文件仍然有序。(每次取1000个小文件中最小的数据)

https://blog.csdn.net/ghoota/article/details/52766299

### 对一个省上百万考生的考试成绩排序 要求o(n)

计数排序(类似桶排序)

![6607d8538edc043249caad5eedd32b93.gif](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220124035035.gif)

https://blog.csdn.net/weixin_39573535/article/details/109959710



## 复杂度

### 时间复杂度和空间复杂度定义,时间换空间和空间换时间的例子有哪些?

**时间复杂度**是指执行**这个算法所需要的计算工作量**

**空间复杂度**是指执行这个算法**所需要的内存空间**

例子：两个值互换的算法

用空间换时间：桶式排序算法、Hash算法、线段树、树状数组

用时间换空间：操作系统中为了节省内存或者缓存，使用的交换算法属于这个范畴，例如LRU算法；多线程、Session和redis缓存、数据库索引技术



## 数据结构

### 数组和链表的区别

| 数组                                                         | 链表                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 在内存中连续存放                                             | 在内存中可以存在任何地方，不要求连续                         |
| 静态分配内存                                                 | 动态分配内存                                                 |
| 数组元素在栈区                                               | 链表元素在堆区                                               |
| 数组利用下标定位，时间复杂度为O(1)                           | 链表定位元素时间复杂度O(n)                                   |
| 数组插入或删除元素的时间复杂度O(n)                           | 链表插入或删除元素的时间复杂度O(1)                           |
| 不利于扩展，数组定义的空间不够时要重新定义数组               | 不指定大小，扩展方便。链表大小不用定义，数据随意增删         |
| 在内存预读方面，内存管理会将连续的存储空间提前读入缓存（局部性原理），所以数组往往会被都读入到缓存中，这样进一步提高了访问的效率 | 链表由于在内存中分布是分散的，往往不会都读入到缓存中，这样本来访问效率就低，这样效率反而更低了 |



### 数据结构中栈、堆有什么区别？

堆是一种经过排序的树形数据结构，每个节点都有一个值，通常我们所说的堆的数据结构是指二叉树。

性质：
（1）堆中某个节点的值总是不大于或不小于其父节点的值；
（2）堆总是一棵完全二叉树。

堆常用来实现优先队列，堆的存取是随意的。

栈是限定仅在表尾进行插入和删除操作的线性表。而且栈是一种具有后进先出的数据结构，又称为后进先出的线性表。



### 栈和队列的区别

栈的插入和删除操作都是在一端进行的，而队列的操作却是在两端进行的。

栈是先进后出，队列是先进先出。

栈只允许在表尾一端进行插入和删除，队列只允许在表尾一端进行插入，在表头一端进行删除。



### 什么时候会产生栈溢出，为什么一直递归就会栈溢出

1.==函数调用层次太深。==函数递归调用时，系统要在栈中不断保存函数调用时的现场和产生的变量，如果递归调用太深，就会造成栈溢出，这时递归无法返回。再有，当函数调用层次过深时也可能导致栈无法容纳这些调用的返回地址而造成栈溢出。
2.==动态申请空间使用之后没有释放。==由于C语言中没有垃圾资源自动回收机制，因此，需要程序主动释放已经不再使用的动态地址空间。申请的动态空间使用的是堆空间，动态空间使用不会造成堆溢出。
3.==数组访问越界。==C语言没有提供数组下标越界检查，如果在程序中出现数组下标访问超出数组范围，在运行过程中可能会内存访问错误。
4.==指针非法访问。==指针保存了一个非法的地址，通过这样的指针访问所指向的地址时会产生内存访问错误。

对于递归，该过程需要在堆栈上至少有一个返回地址。这需要堆栈空间，因此最终会导致StackOverFlow。



## 排序

### 稳定排序

利用关键词排序后，关键词相同的元素之间的相互顺序不变的排序算法

### 常见排序算法，时间复杂度

|   排序方法   | 时间复杂度(平均) | 时间复杂度(最好) | 时间复杂度(最坏) |    空间复杂度    | 稳定性 |
| :----------: | :--------------: | :--------------: | :--------------: | :--------------: | :----: |
| 直接插入排序 |     O($n^2$)     |      O($n$)      |     O($n^2$)     |      O($1$)      |  稳定  |
| 二分插入排序 |     O($n^2$)     |    O($nlogn$)    |     O($n^2$)     |      O($1$)      |  稳定  |
|   希尔排序   |   O($n^{1.3}$)   |      O($n$)      |     O($n^2$)     |      O($1$)      | 不稳定 |
| 直接选择排序 |     O($n^2$)     |      O($n$)      |     O($n^2$)     |      O($1$)      | 不稳定 |
|    堆排序    |    O($nlogn$)    |    O($nlogn$)    |    O($nlogn$)    |      O($1$)      | 不稳定 |
|   冒泡排序   |     O($n^2$)     |      O($n$)      |     O($n^2$)     |      O($1$)      |  稳定  |
|   快速排序   |    O($nlogn$)    |    O($nlogn$)    |     O($n^2$)     | O($logn$)~O($n$) | 不稳定 |
|   归并排序   |    O($nlogn$)    |    O($nlogn$)    |    O($nlogn$)    |      O($n$)      |  稳定  |
|   基数排序   |   O(d($n+r$))    |   O(d($n+r$))    |   O(d($n+r$))    |    O($n+rd$)     |  稳定  |

### 希尔排序（缩小增量排序）

1. **先**–对已有的数列进行分组，明确增量；
2. **中**–每个分组使用直接插入排序进行位置交换；
3. **后**–将每个分组执行直接插入排序后的结果在进行直接插入排序；

![fig](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220125030458.png)

### 基数排序

![img](https://img-blog.csdnimg.cn/f254d3bf02844dc1a91e682437f9d5ca.webp)



### 简单介绍一下快排的原理。什么情况下是性能是最差的

原理：

1. 先从数列中取出一个数作为基准数；
2. 分区过程，将比这个数大的数全放到它的右边，小于或等于它的数全放到它的左边；
3. 再对左右区间重复第二步，直到各区间只有一个数。

快速排序最优的情况就是每一次取到的元素都刚好平分整个数组

最差的情况就是每一次取到的元素就是数组中最小/最大的，这种情况其实就是冒泡排序了(每一次都排好一个元素的顺序)，即当你的序列全部是有序排列的时候



### 快速排序和归并排序的异同

归并排序：是先递归在排序

快速排序：是先找到分割的标志来将数组进行大致的切割（标志的左边都是小于标志的值，右边都是大于标志的值），然后再进行递归

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220125033447.png)

### 堆排序的基本思路

1. 将无需序列构建成一个二叉树，根据升序降序需求选择大顶堆或小顶堆**（一般升序采用大顶堆，降序采用小顶堆)**
2. 遍历二叉树的非叶子节点**自下往上**的构造大顶堆，针对每个非叶子节点，都跟它的左右子节点比较，把最大的值换到这个子树的父节点。
3. 将堆顶元素与末尾元素交换，将最大元素"沉"到数组末端
4. 重新调整结构，使其满足堆定义，然后继续交换堆顶元素与当前末尾元素，反复执行调整+交换步骤，直到整个序列有序



### 代码

#### 堆排序

```go
package main

import (
	"fmt"
)

type Heap struct {
	arr  []int
	size int
}

func (h Heap) heap(u int) {
	l, r := 2*u+1, 2*u+2
	max := u
	if l < h.size && h.arr[max] < h.arr[l] {
		max = l
	}
	if r < h.size && h.arr[max] < h.arr[r] {
		max = r
	}
	if max != u {
		h.arr[max], h.arr[u] = h.arr[u], h.arr[max]
		h.heap(max)
	}
}

func main() {
	var n int
	fmt.Scanln(&n)
	a := make([]int, n)
	for i := 0; i < n; i++ {
		fmt.Scanf("%d", &a[i])
	}
	a = heapSort(a)
	for _, v := range a {
		fmt.Printf("%d ", v)
	}
}

func heapSort(a []int) []int {
	h := Heap{
		arr:  a,
		size: len(a),
	}
	for i := h.size / 2 - 1; i >= 0; i-- { // 从非叶子节点开始倒序遍历
		h.heap(i)
	}
	for h.size > 0 {
		h.arr[0], h.arr[h.size-1] = h.arr[h.size-1], h.arr[0]
		h.size--
		h.heap(0)
	}
	return h.arr
}
```

#### 快速排序

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
	var n int
	fmt.Scanln(&n)
	a := make([]int, n)
	for i := 0; i < n; i++ {
		fmt.Scanf("%d", &a[i])
	}
	quickSort(a, 0, n-1)
	for _, v := range a {
		fmt.Printf("%d ", v)
	}
}

func quickSort(a []int, l int, r int) {
	if l >= r {
		return
	}
	mid := partition(a, l, r)
	quickSort(a, l, mid-1)
	quickSort(a, mid+1, r)
}

func partition(a []int, l int, r int) int {
    pos := rand.Intn(r-l) +  l
	a[l], a[pos] = a[pos], a[l]
	tmp := a[l]
	for l < r {
		for l < r && a[r] >= tmp {
			r--
		}
		a[l] = a[r]
		for l < r && a[l] <= tmp {
			l++
		}
		a[r] = a[l]
	}
	a[l] = tmp
	return l
}
```

#### 归并排序

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	var n int
	fmt.Scanln(&n)
	a := make([]int, n)
	for i := 0; i < n; i++ {
		fmt.Scanf("%d", &a[i])
	}
	a = mergeSort(a)
	for _, v := range a {
		fmt.Printf("%d ", v)
	}
}

func mergeSort(a []int) []int {
	if len(a) <= 1 {
		return a
	}
	mid := len(a) / 2
	l, r := mergeSort(a[:mid]), mergeSort(a[mid:])
	return merge(l, r)
}

func merge(a []int, b []int) []int {
	res := make([]int, len(a)+len(b))
	i, j, k := 0, 0, 0
	for i < len(a) && j < len(b) {
		if a[i] <= b[j] {
			res[k] = a[i]
			i++
		} else {
			res[k] = b[j]
			j++
		}
		k++
	}
	for i < len(a) {
		res[k] = a[i]
		i++
		k++
	}
	for j < len(b) {
		res[k] = b[j]
		j++
		k++
	}
	return res
}
```





## 树

### 二叉搜索树

**所有的结点存储一个关键字**

**最多拥有两个叉**

任何节点的键值一定大于其左子树中的每一个节点的键值，并小于其右子树中的每一个节点的键值。

![image-20230207180242181](https://gitee.com/Transmigration_zhou/pic/raw/master/image-20230207180242181.png)

### 平衡二叉树（AVL树）

![image-20230207180837376](https://gitee.com/Transmigration_zhou/pic/raw/master/image-20230207180837376.png)

它是一棵空树或者它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一颗平衡二叉树。

平衡二叉树很好的解决了二叉查找树退化成链表的问题，把插入，查找，删除的时间复杂度最好情况和最坏情况都维持在$O(logN)$。但是频繁的旋转会使插入和删除牺牲掉$O(logN)$左右的时间。

### 红黑树

![image-20230207180850225](https://gitee.com/Transmigration_zhou/pic/raw/master/image-20230207180850225.png)

https://juejin.cn/post/6844904020549730318

红黑树是平衡二叉树。红黑树能够以$O(logN)$的时间复杂度进行搜索，插入，删除操作。

#### 红黑树的特性

- 每个节点要么是红色，要么是黑色。
- 根节点永远是黑色的。
- 所有的叶子节点都是空节点（即null），并且是黑色的。
- 每个红色节点的两个子节点都是黑色。（从每个叶子到根的路径上不会有两个连续的红色节点。）
- 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点。

### B-Tree

![img](https://pic2.zhimg.com/v2-2c2264cc1c6c603dfeca4f84a2575901_r.jpg)

多路搜索树，并不是二叉的。

### B+Tree

B+树是对B树的一种变形树

![图片](https://pic2.zhimg.com/80/v2-ab406e8e0137fe653629dddae08386a9_1440w.webp)

B+ 树非叶子节点只存储键值信息，数据记录都存放在叶子节点中。

B+ 树中各个页之间是通过双向链表连接的，叶子节点中的数据是通过单向链表连接的。


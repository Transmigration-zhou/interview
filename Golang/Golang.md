[toc]

## 基础部分

### 1. make 和 new 的区别

共同点：

1. new 和 make 都用于分配内存
2. new 和 make 都是在堆上分配内存

不同点：

1. new 对指针类型分配内存，返回值是分配类型的指针
2. make 用于 slice、map和 channel 的初始化，返回值为类型本身，而不是指针
3. new 分配的空间被清零，make 分配空间后，会进行初始化

### 2. 数组和切片的区别

1. 数组是定长，访问和复制不能超过数组定义的长度，否则就会下标越界，切片长度和容量可以自动扩容
2. 数组是值类型，切片是引用类型，每个切片都引用了一个底层数组，切片本身不能存储任何数据，都是这底层数组存储数据，所以修改切片的时候修改的是底层数组中的数据。切片一旦扩容，指向一个新的底层数组，内存地址也就随之改变

### 3. 切片作为函数参数是传值还是传引用？

https://blog.csdn.net/m0_71777195/article/details/125502836

**Go里面函数传参只有值传递一种方式**

Go语言中所有的传参都是值传递（传值），都是一个副本，一个拷贝。因为拷贝的内容有时候是非引用类型（int、string、struct等这些），这样就在函数中就无法修改原内容数据；有的是引用类型（指针、map、slice、chan等这些），这样就可以修改原内容数据。

### 4. for range 的时候它的地址会发生变化么？

for range的时候，地址并没有发生变化。在循环时，会创建一个变量，之后每次循环时遍历到的数据都是以值覆盖的方式赋给这个变量。内存地址始终不变。

解决办法：在每次循环时，创建一个临时变量。

### 5. defer

https://blog.csdn.net/Cassie_zkq/article/details/108567205

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

### 6. byte 和 rune

byte等同于**uint8**，表示一个字节，常用来处理ascii字符

rune等同于**int32**，主要用于表示一个字符类型大于一个字节小于等于4个字节的情况下，常用来处理unicode或utf-8字符，特别是**中文字符。**

### 7. 反射

反射可以在==运行期间==，操作任意类型的对象。

可以通过`TypeOf`方法获得对象类型。通过`ValueOf`获得对象值。

Go 中解析的 tag 是通过反射实现的。


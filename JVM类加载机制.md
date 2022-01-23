[toc]



## 类的生命周期

![Load Class](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220123132517.png)

### 加载

> 加载阶段是类加载过程的第一个阶段。在这个阶段，JVM 的主要目的是将字节码从各个位置（网络、磁盘等）转化为二进制字节流加载到内存中，接着会为这个类在 JVM 的方法区创建一个对应的 Class 对象，这个 Class 对象就是这个类各种数据的访问入口。

​		 其实加载阶段用一句话来说就是：把代码数据加载到内存中。

### 验证

​		 当 JVM 加载完 Class 字节码文件并在方法区创建对应的 Class 对象之后，JVM 便会启动对该字节码流的校验，只有符合 JVM 字节码规范的文件才能被 JVM 正确执行。这个校验过程大致可以分为下面几个类型：

- **JVM规范校验。**JVM 会对字节流进行文件格式校验，判断其是否符合 JVM 规范，是否能被当前版本的虚拟机处理。例如：文件是否是以 `0x cafe babe`开头，主次版本号是否在当前虚拟机处理范围之内等。
- **代码逻辑校验。**JVM 会对代码组成的数据流和控制流进行校验，确保 JVM 运行该字节码文件后不会出现致命错误。例如一个方法要求传入 int 类型的参数，但是使用它的时候却传入了一个 String 类型的参数。一个方法要求返回 String 类型的结果，但是最后却没有返回结果。代码中引用了一个名为 Apple 的类，但是你实际上却没有定义 Apple 类。

​		 当代码数据被加载到内存中后，虚拟机就会对代码数据进行校验，看看这份代码是不是真的按照JVM规范去写的。

### 准备（重点）

​		 当完成字节码文件的校验之后，JVM 便会开始为类变量分配内存并初始化。这里需要注意两个关键点，即内存分配的对象以及初始化的类型。

- **内存分配的对象。**Java 中的变量有「类变量」和「类成员变量」两种类型，「类变量」指的是被 static 修饰的变量，而其他所有类型的变量都属于「类成员变量」。在准备阶段，JVM 只会为「类变量」分配内存，而不会为「类成员变量」分配内存。「类成员变量」的内存分配需要等到初始化阶段才开始。

​		 例如下面的代码在准备阶段，==只会为 factor 属性分配内存，而不会为 website 属性分配内存==。

```
public static int factor = 3;
public String website = "www.cnblogs.com";
```

- **初始化的类型。**在准备阶段，JVM 会为类变量分配内存，并为其初始化。==但是这里的初始化指的是为变量赋予 Java 语言中该数据类型的零值，而不是用户代码里初始化的值。==

​		 例如下面的代码在准备阶段之后，sector 的值将是 0，而不是 3。

```
public static int sector = 3;
```

​		 但如果一个变量是常量（被 static final 修饰）的话，那么在准备阶段，属性便会被赋予用户希望的值。 ==static final 会直接被赋值，而 static 变量会被赋予零值。==

​		 例如下面的代码在准备阶段之后，number 的值将是 3，而不是 0。

```
public static final int number = 3;
```

### 解析

​		 当通过准备阶段之后，JVM 针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符 7 类引用进行解析。这个阶段的主要任务是将其在常量池中的符号引用替换成直接其在内存中的直接引用。

### 初始化（重点）

​		 到了初始化阶段，用户定义的 Java 程序代码才真正开始执行。在这个阶段，JVM 会根据语句执行顺序对类对象进行初始化，一般来说当 JVM 遇到下面 5 种情况的时候会触发初始化：

- 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。

- 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
- 当使用 JDK1.7 动态语言支持时，如果一个 java.lang.invoke.MethodHandle实例最后的解析结果 REF_getstatic,REF_putstatic,REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先出触发其初始化。


​		 这 5 种场景中的行为称为对一个类进行**主动引用**，除此之外，其它所有引用类的方式都不会触发初始化，称为**被动引用**。

#### 主动引用演示

##### Demo1

```java
class SuperClass {
    public static int value = 123;

    static {
        System.out.println("SuperClass static code init!");
    }

    public SuperClass() {
        System.out.println("SuperClass constructor init! ");
    }

    public static int getValue() {
        return value;
    }

    public static void setValue(int value) {
        SuperClass.value = value;
    }
}

class SubClass extends SuperClass {
    public static int subValue = 456;

    static {
        System.out.println("SubClass static code init!");
    }

    public SubClass() {
        System.out.println("SubClass constructor init! ");
    }

    public static int getSubValue() {
        return subValue;
    }

    public static void setSubValue(int subValue) {
        SubClass.subValue = subValue;
    }
}

public class Main {
    // 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，
    // 如果类没有进行过初始化，则需要先触发其初始化。
    public static void main(String[] args) {
        // 1.new字节码指令
//        SubClass subClass = new SubClass();
        /*
          SuperClass static code init!
          SubClass static code init!
          SuperClass constructor init!
          SubClass constructor init!
         */


        // 2.getstatic字节码指令
        // 被final修饰、已在编译期把结果放入常量池的静态字段除外
//        int subValue = SubClass.subValue;
        /*
          SuperClass static code init!
          SubClass static code init!
         */

        // 3.setstatic字节码指令
        // 被final修饰、已在编译期把结果放入常量池的静态字段除外
//        SubClass.subValue = 789;
        /*
          SuperClass static code init!
          SubClass static code init!
         */

        // 4.invokestatic字节码指令
//        SubClass.getSubValue();
        /*
          SuperClass static code init!
          SubClass static code init!
         */
    }
}
```

##### Demo2

```java
import java.lang.reflect.Constructor;

class SuperClass {
    public static int value = 123;

    static {
        System.out.println("SuperClass static code init!");
    }

    public SuperClass() {
        System.out.println("SuperClass constructor init! ");
    }

    public static int getValue() {
        return value;
    }

    public static void setValue(int value) {
        SuperClass.value = value;
    }
}

class SubClass extends SuperClass {
    public static int subValue = 456;

    static {
        System.out.println("SubClass static code init!");
    }

    public SubClass() {
        System.out.println("SubClass constructor init! ");
    }

    public static int getSubValue() {
        return subValue;
    }

    public static void setSubValue(int subValue) {
        SubClass.subValue = subValue;
    }
}

public class Main {
    // 使用java.lang.reflect包的方法对类进行反射调用的时候，
    // 如果类没有进行过初始化，则需要先触发其初始化。
    public static void main(String[] args) {
        try {
            // 使用Class.forName();也行，不要使用对象.class。
            Class<SubClass> clazz = SubClass.class;
            Constructor<SubClass> constructor = clazz.getConstructor();
            constructor.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

输出结果是：

```
SuperClass static code init!
SubClass static code init!
SuperClass constructor init! 
SubClass constructor init! 
```

##### Demo3

```java
import java.lang.reflect.Constructor;

class SuperClass {
    public static int value = 123;

    static {
        System.out.println("SuperClass static code init!");
    }

    public SuperClass() {
        System.out.println("SuperClass constructor init! ");
    }

    public static int getValue() {
        return value;
    }

    public static void setValue(int value) {
        SuperClass.value = value;
    }
}

class SubClass extends SuperClass {
    public static int subValue = 456;

    static {
        System.out.println("SubClass static code init!");
    }

    public SubClass() {
        System.out.println("SubClass constructor init! ");
    }

    public static int getSubValue() {
        return subValue;
    }

    public static void setSubValue(int subValue) {
        SubClass.subValue = subValue;
    }
}

public class Main {
    // 当初始化一个类的时候，如果发现其父类还没有进行过初始化，
    // 则需要先触发其父类的初始化
    public static void main(String[] args) {
        SubClass subClass = new SubClass();
    }
}
```

输出结果是：

```
SuperClass static code init!
SubClass static code init!
SuperClass constructor init! 
SubClass constructor init! 
```

##### Demo4

```java
import java.lang.reflect.Constructor;

class SuperClass {
    public static int value = 123;

    static {
        System.out.println("SuperClass static code init!");
    }

    public SuperClass() {
        System.out.println("SuperClass constructor init! ");
    }

    public static int getValue() {
        return value;
    }

    public static void setValue(int value) {
        SuperClass.value = value;
    }
}

class SubClass extends SuperClass {
    public static int subValue = 456;

    static {
        System.out.println("SubClass static code init!");
    }

    public SubClass() {
        System.out.println("SubClass constructor init! ");
    }

    public static int getSubValue() {
        return subValue;
    }

    public static void setSubValue(int subValue) {
        SubClass.subValue = subValue;
    }
}

public class Main {
    static {
        System.out.println("Initialization static code init!");
    }

    public Main() {
        System.out.println("Initialization constructor init!");
    }

    public static void main(String[] args) {
        // 当虚拟机启动时，用户需要指定一个要执行的主类(包含main()方法的那个类)，虚拟机会先初始化这个主类。
    }
}
```

输出结果是：

```
Initialization static code init!
```

##### Demo5

```java
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

class MethodHandleClass {
    static {
        System.out.println("MethodHandleClass static code init!");
    }

    public MethodHandleClass() {
        System.out.println("MethodHandleClass constructor init!");
    }

    // REF_invokeStatic
    public static void testREF_invokeStatic(String str) {
        System.out.println(str);
    }
}

public class Main {
    static {
        System.out.println("Initialization static code init!");
    }

    public Main() {
        System.out.println("Initialization constructor init!");
    }

    public static void main(String[] args) {
        MethodHandles.Lookup lookup = MethodHandles.lookup();
        try {
            // REF_invokeStatic
            MethodHandle testREF_invokeStatic = lookup.findStatic(MethodHandleClass.class, "testREF_invokeStatic", MethodType.methodType(void.class, String.class));
            testREF_invokeStatic.invoke("啥也不干，打印一段话");

            // REF_getStatic

            // REF_putStatic
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
    }
}
```

#### 被动引用演示

##### Demo1

```java
/**
 * 被动引用 Demo1:
 * 通过子类引用父类的静态字段，不会导致子类初始化。
 *
 */
class SuperClass {
    static {
        System.out.println("SuperClass init!");
    }

    public static int value = 123;
}

class SubClass extends SuperClass {
    static {
        System.out.println("SubClass init!");
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(SubClass.value);
    }
}
```

输出结果是：

```
SuperClass init!
123
```

​		 对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

##### Demo2

```java
/**
 * 被动引用 Demo2:
 * 通过数组定义来引用类，不会触发此类的初始化。
 *
 */
class SuperClass {
    static {
        System.out.println("SuperClass init!");
    }
    public static int value = 123;
}

class SubClass extends SuperClass {
    static {
        System.out.println("SubClass init!");
    }
}

public class Main {
    public static void main(String[] args) {
        SuperClass[] superClasses = new SuperClass[10];
    }
}
```

​		 这段代码不会触发父类的初始化，但会触发“L+全类名”这个类的初始化，它由虚拟机自动生成，直接继承自 java.lang.Object，创建动作由字节码指令 newarray 触发。

##### Demo3

```java
/**
 * 被动引用 Demo3:
 * 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。
 *
 */
class ConstClass {
    static {
        System.out.println("ConstClass init!");
    }
    public static final String HELLO_BINGO = "Hello Bingo";
}

public class Main {
    public static void main(String[] args) {
        System.out.println(ConstClass.HELLO_BINGO);
    }
}
```

输出结果是：

```
Hello Bingo
```

### 使用

​		 当 JVM 完成初始化阶段之后，JVM 便开始从入口方法开始执行用户的程序代码。

### 卸载

​		 当用户程序代码执行完毕后，JVM 便开始销毁创建的 Class 对象，最后负责运行的 JVM 也退出内存。

## 例子

### Demo1

```java
public class Book {
    public static void main(String[] args) {
        System.out.println("Hello ShuYi.");
    }

    Book() {
        System.out.println("书的构造方法");
        System.out.println("price=" + price + ",amount=" + amount);
    }

    {
        System.out.println("书的普通代码块");
    }

    int price = 110;

    static {
        System.out.println("书的静态代码块");
    }

    static int amount = 112;
}
```

输出结果是：

```
书的静态代码块
Hello ShuYi.
```

​		 下面我们来简单分析一下，首先根据上面说到的触发初始化的5种情况的第4种（当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类），我们会进行类的初始化。

​		 在我们代码中，我们只知道有一个构造方法，但实际上Java代码编译成字节码之后，是没有构造方法的概念的，只有类初始化方法 和 对象初始化方法 。

- 类初始化方法。编译器会按照其出现顺序，收集类变量的赋值语句、静态代码块，最终组成类初始化方法。==**类初始化方法一般在类初始化的时候执行。**==

上面的这个例子，其类初始化方法就是下面这段代码了：

```java
static {
	System.out.println("书的静态代码块");
}
static int amount = 112;
```

- 对象初始化方法。编译器会按照其出现顺序，收集成员变量的赋值语句、普通代码块，最后收集构造函数的代码，最终组成对象初始化方法。==**对象初始化方法一般在实例化类对象的时候执行。**==

上面这个例子，其对象初始化方法就是下面这段代码了：

```java
{
	System.out.println("书的普通代码块");
}
int price = 110;
System.out.println("书的构造方法");
System.out.println("price=" + price +",amount=" + amount);
```

​		 其实上面的这个例子其实没有执行对象初始化方法。

​		 因为我们确实没有进行 Book 类对象的实例化。如果你在 main 方法中增加 new Book() 语句，你会发现对象的初始化方法执行了！

### Demo2

```java
class Grandpa {
    static {
        System.out.println("爷爷在静态代码块");
    }
}

class Father extends Grandpa {
    static {
        System.out.println("爸爸在静态代码块");
    }

    public static int factor = 25;

    public Father() {
        System.out.println("我是爸爸~");
    }
}

class Son extends Father {
    static {
        System.out.println("儿子在静态代码块");
    }

    public Son() {
        System.out.println("我是儿子~");
    }
}

public class InitializationDemo {
    public static void main(String[] args) {
        System.out.println("爸爸的岁数:" + Son.factor);    //入口
    }
}
```

输出结果是：

```
爷爷在静态代码块
爸爸在静态代码块
爸爸的岁数:25
```

​		为什么没有输出「儿子在静态代码块」这个字符串？

​		 **这是因为对于静态字段，只有直接定义这个字段的类才会被初始化（执行静态代码块）。**因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

​		 对面上面的这个例子，我们可以从入口开始分析一路分析下去：

- 首先程序到 main 方法这里，使用标准化输出 Son 类中的 factor 类成员变量，但是 Son 类中并没有定义这个类成员变量。于是往父类去找，我们在 Father 类中找到了对应的类成员变量，于是触发了 Father 的初始化。
- 但根据我们上面说到的初始化的 5 种情况中的第 3 种（当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化）。我们需要先初始化 Father 类的父类，也就是先初始化 Grandpa 类再初始化 Father 类。于是我们先初始化 Grandpa 类输出：「爷爷在静态代码块」，再初始化 Father 类输出：「爸爸在静态代码块」。
- 最后，所有父类都初始化完成之后，Son 类才能调用父类的静态变量，从而输出：「爸爸的岁数：25」。

### Demo3

```java
class Grandpa {
    static {
        System.out.println("爷爷在静态代码块");
    }

    public Grandpa() {
        System.out.println("我是爷爷~");
    }
}

class Father extends Grandpa {
    static {
        System.out.println("爸爸在静态代码块");
    }

    public Father() {
        System.out.println("我是爸爸~");
    }
}

class Son extends Father {
    static {
        System.out.println("儿子在静态代码块");
    }

    public Son() {
        System.out.println("我是儿子~");
    }
}

public class InitializationDemo {
    public static void main(String[] args) {
        new Son();
    }
}
```

输出结果是：

```
爷爷在静态代码块
爸爸在静态代码块
儿子在静态代码块
我是爷爷~
我是爸爸~
我是儿子~
```

分析一下上面代码的执行流程：

- 首先在入口这里我们实例化一个 Son 对象，因此会触发 Son 类的初始化，而 Son 类的初始化又会带动 Father 、Grandpa 类的初始化，从而执行对应类中的静态代码块。因此会输出：「爷爷在静态代码块」、「爸爸在静态代码块」、「儿子在静态代码块」。
- 当 Son 类完成初始化之后，便会调用 Son 类的构造方法，而 Son 类构造方法的调用同样会带动 Father、Grandpa 类构造方法的调用，最后会输出：「我是爷爷」、「我是爸爸」、「我是儿子~」。

### Demo4

```java
public class Book {
    public static void main(String[] args) {
        staticFunction();
    }

    static Book book = new Book();

    static {
        System.out.println("书的静态代码块");
    }

    {
        System.out.println("书的普通代码块");
    }

    Book() {
        System.out.println("书的构造方法");
        System.out.println("price=" + price + ",amount=" + amount);
    }

    public static void staticFunction() {
        System.out.println("书的静态方法");
    }

    int price = 110;
    static int amount = 112;
}
```

输出结果是：

```
书的普通代码块
书的构造方法
price=110,amount=0
书的静态代码块
书的静态方法
```

​		 下面我们一步步来分析一下代码的整个执行流程。

​		 在上面两个例子中，因为 main 方法所在类并没有多余的代码，我们都直接忽略了 main 方法所在类的初始化。

​		 但在这个例子中，main 方法所在类有许多代码，我们就并不能直接忽略了。

- 当 JVM 在准备阶段的时候，便会为类变量分配内存和进行初始化。此时，我们的 book 实例变量被初始化为 null，amount 变量被初始化为 0。
- 当进入初始化阶段后，因为 Book 方法是程序的入口，根据我们上面说到的类初始化的五种情况的第四种（当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类）。所以JVM 会初始化 Book 类，即执行类构造器 。
- JVM 对 Book 类进行初始化首先是执行类构造器（按顺序收集类中所有静态代码块和类变量赋值语句就组成了类构造器 ），后执行对象的构造器（按顺序收集成员变量赋值和普通代码块，最后收集对象构造器，最终组成对象构造器 ）。

​		 对于 Book 类，其类构造方法（）可以简单表示如下：

```java
static Book book = new Book();
static {
    System.out.println("书的静态代码块");
}
static int amount = 112;
```

​		 于是首先执行`static Book book = new Book();`这一条语句，这条语句又触发了类的实例化。于是 JVM 执行对象构造器 ，收集后的对象构造器 代码：

```java
{
    System.out.println("书的普通代码块");
}
int price = 110;
Book() {
    System.out.println("书的构造方法");
    System.out.println("price=" + price +", amount=" + amount);
}
```

​		 于是此时 price 赋予 110 的值，输出：「书的普通代码块」、「书的构造方法」。而此时 price 为 110 的值，而 amount 的赋值语句并未执行，所以只有在准备阶段赋予的零值，所以之后输出「price=110,amount=0」。

​		 当类实例化完成之后，JVM 继续进行类构造器的初始化：

```java
static Book book = new Book();  //完成类实例化
static {
    System.out.println("书的静态代码块");
}
static int amount = 112;
```

​		 即输出：「书的静态代码块」，之后对 amount 赋予 112 的值。

- 到这里，类的初始化已经完成，JVM 执行 main 方法的内容。

```java
public static void main(String[] args) {
    staticFunction();
}
```

即输出：「书的静态方法」。

## 方法论

​		 从上面几个例子可以看出，分析一个类的执行顺序大概可以按照如下步骤：

- **确定类变量的初始值。**在类加载的准备阶段，JVM 会为类变量初始化零值，这时候类变量会有一个初始的零值。如果是被 final 修饰的类变量，则直接会被初始成用户想要的值。
- **初始化入口方法。**当进入类加载的初始化阶段后，JVM 会寻找整个 main 方法入口，从而初始化 main 方法所在的整个类。当需要对一个类进行初始化时，会首先初始化类构造器（），之后初始化对象构造器（）。
- **初始化类构造器。**JVM 会按顺序收集类变量的赋值语句、静态代码块，最终组成类构造器由 JVM 执行。
- **初始化对象构造器。**JVM 会按照收集成员变量的赋值语句、普通代码块，最后收集构造方法，将它们组成对象构造器，最终由 JVM 执行。

​		 如果在初始化 main 方法所在类的时候遇到了其他类的初始化，那么就先加载对应的类，加载完成之后返回。如此反复循环，最终返回 main 方法所在类。



参考：

[JVM基础系列第7讲：JVM 类加载机制 ](https://www.cnblogs.com/chanshuyi/p/jvm_serial_07_jvm_class_loader_mechanism.html)

[JVM 底层原理最全知识总结-类的生命周期 ](https://doocs.github.io/jvm/08-load-class-time.html#%E7%B1%BB%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)

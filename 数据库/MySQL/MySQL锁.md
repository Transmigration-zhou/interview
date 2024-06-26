## 锁机制与 InnoDB 锁算法

### MyISAM 和 InnoDB 存储引擎使用的锁

- MyISAM 采用表级锁(table-level locking)。
- InnoDB 支持行级锁(row-level locking)和表级锁，默认为行级锁



### 表级锁和行级锁

- **表级锁：** MySQL 中锁定 **粒度最大** 的一种锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM 和 InnoDB 引擎都支持表级锁。
- **行级锁：** MySQL 中锁定 **粒度最小** 的一种锁，只针对当前操作的行进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。

不管你是 MySQL 的什么存储引擎，对于表锁的策略都是一样的。

行级锁只在存储引擎层实现。



### 读锁和写锁

对于 InnoDB 引擎来说，读锁和写锁可以加在表上，也可以加在行上。

对于并发读和并发写的问题，可以通过实现一个由两种类型的锁组成的锁系统来解决。这两种类型的锁通常被称为 **共享锁（Shared Lock，S Lock）** 和 **排他锁（Exclusive Lock，X Lock）**，也叫 **读锁（readlock）** 和 **写锁（write lock）**：

- 共享锁 / 读锁：允许事务读（select）数据 `select … lock in share mode;`
- 排他锁 / 写锁：允许事务删除（delete）或更新（`update`）数据 `select … for update;`

读锁是共享的，或者说是相互不阻塞的。多个事务在同一时刻可以同时读取同一个资源，而互不干扰。写锁是排他的，也就是说一个写锁会阻塞其他的读锁和写锁，这样就能确保在给定的时间里，只有一个事务能执行写入，并防止其他用户读取正在写入的同一资源。

|      | S 锁     | X 锁   |
| ---- | -------- | ------ |
| S 锁 | **兼容** | 不兼容 |
| X 锁 | 不兼容   | 不兼容 |



### 意向锁

InnoDB 存储引擎支持 多粒度（granular）锁定，就是说允许事务在行级上的锁和表级上的锁同时存在。

意向锁是一个**表级锁**，其作用就是指明接下来的事务将会用到哪种锁。

- **意向共享锁（IS Lock）**：当事务想要获得一张表中某几行的共享锁（行级锁）时，InnoDB 存储引擎会自动地先获取该表的意向共享锁（表级锁）
- **意向排他锁（IX Lock）**：当事务想要获得一张表中某几行的排他锁（行级锁）时，InnoDB 存储引擎会自动地先获取该表的意向排他锁（表级锁）

意向锁之间是相互兼容的。

|       | IS 锁 | IX 锁 |
| ----- | ----- | ----- |
| IS 锁 | 兼容  | 兼容  |
| IX 锁 | 兼容  | 兼容  |

意向锁与**表级**读写锁之间大部分都是不兼容的，意向锁不会与**行级**的读写锁互斥。

|       | X 锁   | S 锁   |
| ----- | ------ | ------ |
| IS 锁 | 不兼容 | 兼容   |
| IX 锁 | 不兼容 | 不兼容 |



### InnoDB 存储引擎的锁的算法有三种

- Record lock：记录锁，单个行记录上的锁
- Gap lock：间隙锁，锁定一个范围，不包括记录本身
  - 当我们采用范围条件查询数据时，InnoDB 会对这个范围内的数据进行加锁。比如有 id 为：1、3、5、7 的 4 条数据，我们查找 1-7 范围的数据。那么 1-7 都会被加上锁。2、4、6 也在 1-7 的范围中，但是不存在这些数据记录，这些 2、4、6 就被称为间隙。

- Next-key lock：record+gap 临键锁，锁定一个范围，包含记录本身



## 并发控制

实现并发控制的主要手段分为乐观并发控制（乐观锁）和悲观并发控制（悲观锁）两种。

### 悲观锁

悲观锁：在修改数据之前先锁定，再修改的方式。

行锁、表锁、读锁、写锁都是悲观锁

悲观并发控制实际上是“先取锁再访问”的保守策略，为数据处理的安全提供了保证。但是在效率方面，处理加锁的机制会让数据库产生额外的开销，还有增加产生死锁的机会。另外还会降低并行性，一个事务如果锁定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数据。

#### 实现方式

悲观锁的实现，往往依靠数据库提供的锁机制。

在数据库中，悲观锁的流程如下：

1. 在对记录进行修改前，先尝试为该记录加上排他锁。
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
4. 期间如果有其他对该记录做修改或加排他锁的操作，都会等待解锁或直接抛出异常。

### 乐观锁

乐观锁：乐观锁是相对悲观锁而言的，乐观锁假设数据一般情况不会造成冲突。在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果冲突，则返回给用户异常信息，让用户决定如何去做

乐观锁比较适用于读多写少的情况（多读场景），悲观锁比较适用于写多读少的情况（多写场景）。

#### 实现方式

乐观锁不需要借助数据库的锁机制。

主要就是两个步骤：冲突检测和数据更新。

使用版本号实现乐观锁：数据版本机制和时间戳机制

1. 使用版本号实现乐观锁

   - 数据版本(Version)机制
     - 一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。当读取数据时，将 version 的值一同读出，数据每更新一次，对此 version 值加 1。当提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的 version 值进行比对，如果数据库表当前版本号与第一次取出来的 version 值相等，则予以更新，否则认为是过期数据。

   - 时间戳机制
     - 在需要乐观锁控制的 table 中增加一个字段，字段类型使用时间戳(timestamp)，和上面的 version 类似，也是在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则继续，否则就是版本冲突。

2. 使用条件限制实现乐观锁

   - 这个适用于只更新时做数据安全校验，适合库存模型，扣份额和回滚份额，性能更高。

     ![image-20230208200219848](https://gitee.com/Transmigration_zhou/pic/raw/master/image-20230208200219848.png)



## QA

### 1. select……for update会锁表还是锁行？

加的是行锁还是表锁，这就要看是不是用了索引/主键。

没用索引/主键的话就是表锁，否则就是是行锁。

### 2.  insert、lock in share mode是如何加锁的

执行 insert 语句，对要操作的页加 RW-X-LATCH，然后判断是否有和插入意向锁冲突的锁，如果有，加插入意向锁，进入锁等待；如果没有，直接写数据，不加任何锁，结束后释放 RW-X-LATCH。

执行 select ... lock in share mode 语句，对要操作的页加 RW-S-LATCH，如果页面上存在 RW-X-LATCH 会被阻塞，没有的话则判断记录上是否存在活跃的事务，如果存在，则为 insert 事务创建一个排他记录锁，并将自己加入到锁等待队列，最后也会释放 RW-S-LATCH；

### 3. update 加锁

https://www.cnblogs.com/frankyou/p/15271070.html

**InnoDB 存储引擎自己实现了行锁，通过 next-key 锁（记录锁和间隙锁的组合）来锁住记录本身和记录之间的“间隙”，防止其他事务在这个记录之间插入新的记录，从而避免了幻读现象。**

当我们执行 update 语句时，实际上是会对记录加独占锁（X 锁）的，如果其他事务对持有独占锁的记录进行修改时是会被阻塞的。

另外，这个锁并不是执行完 update 语句就会释放的，而是会等事务结束时才会释放。

**在 InnoDB 事务中，对记录加锁带基本单位是 next-key 锁，但是会因为一些条件会降级成间隙锁，或者记录锁。加锁的位置准确的说，锁是加在索引上的而非行上。**、

比如**，在 update 语句的 where 条件使用了唯一索引，那么 next-key 锁会降级成记录锁，也就是只会给一行记录加锁。**

无索引的情况下，如果不走主键，那么update为表锁。

有索引的情况下，走索引或者走主键（走索引等价于先走索引，再走主键），那么update变为行锁。
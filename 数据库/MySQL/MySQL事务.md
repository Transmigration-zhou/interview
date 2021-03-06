## 事务的特性(ACID)

1. **原子性：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. **一致性：** 执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；
3. **隔离性：** 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
4. **持久性：** 一个事务被提交之后，它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。



## 事务的实现（如何保证）

https://draveness.me/mysql-transaction/

https://www.jianshu.com/p/f0a1b00a6002

### 原子性

事务就是一系列的操作，要么全部都执行，要么全部都不执行。

想要保证事务的原子性，就意味着需要在操作发生异常时，对该事务所有之前执行过的操作进行**回滚**。

在MySQL中，这个回滚是通过==回滚日志(Undo Log)==实现的。简单的说，回滚日志就是记录了你所有操作的**逆操作**，在需要回滚时，就把这个事务的回滚日志里的操作全部执行一次。

- 当你delete一条数据的时候，就需要记录这条数据的信息，回滚的时候，insert这条旧数据
- 当你update一条数据的时候，就需要记录之前的旧值，回滚的时候，根据旧值执行update操作
- 当年insert一条数据的时候，就需要这条记录的主键，回滚的时候，根据主键执行delete操作

**undo log**记录了这些回滚需要的信息，当事务发生异常触发ROLLBACK时，就按照日志逻辑地将回滚日志里的操作全部执行，从而达到“撤销”操作的效果。

### 持久性

持久性就是指，事务一旦被提交，那么数据一定会被写入到数据库中并持久储存起来。

另外，当事务被提交后就无法再回滚，如果想要撤销一个已经提交的事务，那就只能执行一个效果与其相反的事务，这也是持久性的一种体现。

与原子性一样，事务的持久性也是通过日志来实现的，MySQL 使用==重做日志（redo log）==实现事务的持久性，重做日志由两部分组成，一是内存中的重做日志缓冲区，因为重做日志缓冲区在内存中，所以它是易失的，另一个就是在磁盘上的重做日志文件，它是持久的。

当我们在事务中尝试对数据进行更改时，首先将数据从磁盘读入内存，更新内存缓存的数据，然后会生成一条重做日志（本次修改的逆操作）缓存，放在重做日志缓冲区中。当事务真正提交时，再将刚才缓冲区中的日志写入重做日志中做持久化保存，最后再把内存中的数据变动同步到磁盘上。

![Redo-Logging](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220320143038.jpg)

在 InnoDB 中，重做日志都是以 512B 的块的形式进行存储的，同时因为块的大小与磁盘扇区大小相同，所以重做日志的写入可以保证原子性，不会由于机器断电导致重做日志仅写入一半并留下脏数据。

**回滚日志**也是需要持久化储存的，因此他们也会创建对应的重做日志，在发生错误后，数据库重启时，会从重做日志中找出未被更新到的数据库磁盘上的日志，重新执行来满足事务的持久性。

#### 采用redo log的好处？

其实好处就是将**redo log**进行刷盘比对数据页刷盘效率高，具体表现如下：

- **redo log**体积小，毕竟只记录了哪一页修改了啥，因此体积小，刷盘快。
- **redo log**是一直往末尾进行追加，属于顺序IO。效率显然比随机IO来的快。



### 隔离性

事务的隔离性会跟并发等相关概念联系的非常密切，因为它主要就是为了保证并行事务处理能够达到“互不干扰”的效果。

隔离的实现说到底其实是**并发控制**，因此不同隔离级别的实现，其实就是采用了不同的并发控制机制。

**1.锁**

锁是一种最为常见的并发控制机制，在一个事务中，我们并不会将整个数据库都加锁，不过在一个事务中，自然是不可能把整个数据库都加锁的，而是只对要访问的数据加锁（具体的粒度有行、表等）。而这些资源锁也是理所当然地分为共享锁（读锁 Shared）和互斥锁（写锁 Exclusive）两种。

读锁可以保证操作并发执行而不受影响，写锁则保证了更新数据库时不会受到其他事务的干扰。

**2.时间戳**

用时间戳实现隔离性，需要为记录配置两个字段

- **读时间戳**：用于保存所有访问该记录的事务中的最大时间戳（最后读取时间）
- **写时间戳**：用于保存将记录改到当前值的事务的时间戳（最后修改时间）

这样的事务在并行执行时，用的是**乐观锁**，先任由事务对数据进行修改，在写回去的时候在判断记录的时间戳有没有修改，如果没有被修改，就写入，否则，就生成一个新的时间戳并再次尝试更新数据。

PostgreSQL就使用了这种思想来控制事务。

**3.多版本和快照隔离**

通过维护多个版本的数据，数据库便可以允许事务并发执行遇到互斥锁时，转而读取旧版本的数据快照。这样就能显著地提升读取的性能，我们简称这一手段为MVCC。



### 一致性

数据库领域其实包含两个一致性，一个是 ACID 中的一致性、另一个是 CAP 定义中的一致性

#### ACID 中的一致性

事务的一致性定义基本可以理解为是事务对数据完整性约束的遵循。这些约束可能包括主键约束、外键约束或是一些用户自定义约束，还有触发器（插入文章的浏览记录时，使用触发器去更新对应文章的浏览量，保证每增加一条浏览记录，对应文章浏览量+1）等等。事务执行的前后都是合法的数据状态，不会违背任何的数据完整性，这就是“一致”的意思。

数据库通过原子性、隔离性、持久性来保证一致性。

一致性也指逻辑上的对于开发者的要求，写出正确的事务逻辑，比如银行的转账不能只加钱不减钱，这是应用层面的一致性要求。

####  CAP 中的一致性

CAP 定理中的数据一致性，其实是说分布式系统中的各个节点中对于同一数据的拷贝有着相同的值。



## 并发一致性问题

1. 修改丢失：一个事务的更新操作被另一个事务的更新操作替换
2. 脏读：脏读指的是一个事务读取到==另一个事务已修改但未提交==的数据
3. 不可重复读：指的是在同一个事务内，对同一数据的多次读取的结果不一样，一般是其它事务进行 update 操作导致的
4. 幻读：幻读也属于不可重复读，但是一般是因为其它事务进行 insert 或 delete 操作导致的



### 回滚日志（undo log）和重做日志（redo log）

在数据库系统中，事务的原子性和一致性是由**事务日志**实现的，在具体的实现上，使用的就是之前提到的回滚日志和重做日志，它们保证了两点：

- 发生错误或者需要回滚的事务能够成功回滚（原子性）
- 事务提交后，数据还没来得及写入磁盘就宕机时，重启后能够成功恢复数据（一致性）

一条事务日志同时包含了修改前后的值，能够非常简单的进行回滚和重做两种操作。

|      | X 锁   | S 锁     |
| ---- | ------ | -------- |
| X 锁 | 不兼容 | 不兼容   |
| S 锁 | 不兼容 | **兼容** |



## 事务的隔离级别

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220303182428.png)

- read uncommitted：读未提交。不能解决任何并发问题

- read committed：读已提交。只能解决脏读问题

- repeatable read：可重复读。只能解决脏读和不可重复读问题

- serializable：串行化 ( 序列化 )。都能解决

MySQL 默认的隔离级别是  repeatable read

这四大隔离级别的隔离效果是逐渐增强的，但是性能是逐渐变差的



## 为什么隔离级别越高，性能越差？

因为四个隔离级别是通过加锁解决的并发问题，MySQL 中的锁可以分为共享锁、排它锁、间隙锁、行锁和表锁

首先读未提交没有加锁，因此性能最好，但是也不能解决任何并发问题

而串行化则在读的时候加的是==共享锁==，写的时候加排它锁阻塞其它事务的写入和读取，因此性能最差，但是可以解决所有的并发问题

而读已提交和可重复读则是在解决一定并发问题时也能具有并发能力，所以使用的锁机制要优化很多，他们的底层使用的是 MVCC 进行实现的



## MVCC 的原理（undo log、ReadView）

https://flying-veal.notion.site/MVCC-undo-log-ReadView-e51f6b51561d4301860189d09c745974



## MySQL如何解决幻读

MySQL在REPEATABLE READ(可重复读)隔离级别实际上就已经解决了幻读问题

https://flying-veal.notion.site/MySQL-3d19104aec054289a9659a8a0afb6565




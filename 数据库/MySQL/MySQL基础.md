[toc]

## 1. 关系型数据库和非关系型数据库区别

关系型数据库：MySQL、Oracle

非关系型数据库：NoSQL、Redis

- 结构：MySQL采用==二维表结构==，NoSQL==以键值对存储==。

- 查询速度：NoSQL数据库将数据==存储于缓存==之中，MySQL将数据==存储在硬盘==中，自然查询速度远不及NoSQL数据库。

- 存储数据的格式：NoSQL的存储格式是key,value形式、文档形式、图片形式等等，所以可以存储基础类型以及对象或者是集合等各种格式，而数据库则只支持基础类型。

- 安全性：MySQL有事务级别，所以安全性高，NoSQL相对低。

- 数据一致性：MySQL的数据有ACID四个特性，保证数据的隔离性，原子性，一致性，持久性，NoSQL不保证一致性，可能查询到中间态的一个数据。

  

***

关系型数据库：指采用了关系模型来组织数据的数据库。
关系模型指的就是二维表格模型，而一个关系型数据库就是由二维表及其之间的联系所组成的一个数据组织。

**关系型数据库的优点：**

- 1.容易理解：二维表结构是非常贴近逻辑世界的一个概念，关系模型相对网状、层次等其他模型来说更容易理解
- 2.使用方便：通用的SQL语言使得操作关系型数据库非常方便
- 3.易于维护：丰富的完整性(实体完整性、参照完整性和用户定义的完整性)大大减低了数据冗余和数据不一致的概率
- 4.支持SQL，可用于复杂的查询。
- 5.支持事务，安全性高

非关系型数据库：指非关系型的，==分布式==的，且一般不保证遵循`ACID`原则的数据存储系统。

为了==处理海量数据==，但==不适用于持久存储==

**非关系型数据库结构：**

非关系型数据库以键值对存储，且结构不固定，每一个元组可以有不一样的字段，每个元组可以根据需要增加一些自己的键值对，不局限于固定的结构，可以减少一些时间和空间的开销。



## 2. 数据库三范式

三大范式（每一范式都在前一范式的基础上）
第一范式：表中字段的数据，不可以再拆分
第二范式：所有非主键属性都完全依赖于主键（消除部分依赖）
第三范式：不存在非主键属性对主键的传递依赖关系（消除传递依赖）

### 冗余字段

冗余字段是指反复出现的，重复的字段。也就是说在数据库中如果表a出现过字段b，表c再出现字段b，那么字段b就可以被看作是冗余字段了。

弊端：不仅会使数据库性能降低还会带来数据不一致等一系列问题

优点：减少 join 语句使用，让数据库执行性能更快



## 3. 视图

视图是一种虚表，与包含数据的表不一样，视图只包含使用时动态检索数据的查询，不包含任何列或数据。

使用视图可以简化复杂的 sql 操作，隐藏具体的细节，保护数据。

视图创建后，可以使用与表相同的方式利用它们。

视图不能被索引，也不能有关联的触发器或默认值，如果视图本身内有order by 则对视图再次order by将被覆盖。

创建视图 :

```sql
create view 视图名 as 查询语句;
```

对于某些视图比如未使用联结子查询分组聚集函数Distinct Union等，是可以对其更新的，对视图的更新将对基表进行更新；但是视图主要用于简化检索，保护数据，并不用于更新，而且大部分视图都不可以更新。

如果开发过程中，遇到的情景满足==简单、安全、数据独立==的情况下就可以使用视图。

**简单**：使用视图的用户不需要关心后面对应表的结构，关联，筛选条件，这是一个过滤好的符合条件的结果集。（重用子查询）

**安全**：使用一个视图的用户只能看到特定的行和列，这种权限限制是表权限管理无法做到的。（项目经理只想我们看到用户名，不想让我们看到密码，就基于用户表的查询，建立一个视图）

**数据独立**：一旦视图的结构确定，可以屏蔽表结构变化对视图的影响，原表增加列对于视图没有影响，但是修改列名，要修改视图，原表增删改数据，视图会自动更新。



## 4. MySQL的三大日志

- undo log：主要用于事务的回滚操作。当事务执行过程中发生异常或需要回滚时，回滚日志记录了事务的操作信息，可以用于撤销事务对数据库的修改，==实现事务的原子性==。例如，对于每个UPDATE语句，对应一条相反的UPDATE的 undo log。
  - 适用场景：事务回滚、MVCC；

- redo log：主要用于==保证事务的持久性==。当数据库发生故障时，通过重做日志可以将未提交的事务重新执行，确保数据的一致性。redo log是循环写，日志空间大小固定，会覆盖。==实现事物的持久性。==
  - 适用场景：崩溃恢复（crash-safe）
- binlog：二进制日志记录了所有对数据库的更改操作，包括数据更新、插入、删除等，以便在主从复制时同步数据或进行数据恢复和备份。与 redo log 不同，binlog 是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。主要用于：
   - 主从复制：在 Master 端开启 binlog ，然后将 binlog 发送到各个 Slave 端， Slave 端重放 binlog 来达到主从数据一致；
   - 数据恢复：通过使用 mysql binlog 工具来恢复数据。
   - 数据备份

### redo log

https://www.cnblogs.com/ZhuChangwu/p/14096575.html

作用：确保事务的持久性。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启 mysql 服务的时候，根据 redo log 进行重做，从而达到事务的持久性这一特性。



mysql中的每一次更新操作都需要写进磁盘，这个过程中需要先找到需要更新的这条数据，然后进行更新操作，整个过程的IO成本很高，为了解决这个问题，提出了WAL技术（write ahead logging），其主要流程为**先写日志，再写磁盘**。

具体来说，当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log里面，并更新内存，这个时候更新就算完成了。

同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做。



InnoDB中重做日志是为了实现事务的==持久性==，也就是ACID中的D。其中主要分为两部分：

- 内存中的重做缓冲日志，临时的
- 重做日志文件，持久的

innodb是支持事务的存储引擎，其在事务提交的时候，必须将将当前事务的日志都写入到重做日志文件中，进行持久化存储，等事务commit操作完成事务才算完成。这里的日志指的是redo log（重做日志）和undo log（回滚日志）。redo log是为了保证事务的持久性，undo log则是为了事务回滚以及MVCC功能。

#### 重做日志的刷盘动作

​	为了保证每次日志都会写入到重做日志文件中，会在每次将重做日志缓冲写入到重做日志文件后，调用一次fsync，将数据刷到磁盘上。

> 这里调用fsync的因为写入到磁盘前，会先写入到内核缓冲区中，再由操作系统决定何时进行刷盘动作，而调用fsync则是手动刷盘

​	fsync操作是比较耗费性能的，因此Innodb存储引擎也允许设置非持久性的模式，即事务提交后，日志不会直接写入到磁盘，而是等待一个时间周期后再执行fsync，这样可以显著提升mysql的执行效率，但是在数据库宕机时容易出现数据丢失的情况。

​	刷盘策略是由参数控制的innodb_flush_log_at_trx_commit

- 0：事务提交不写入重做日志，仅在mysql的master thread中完成，在master thread中每1秒进行一次重做日志的fsync
- 1：每次提交都会执行一次fsync操作（默认）
- 2：事务提交重做日志写入重做日志文件，但是只是写入到了操作系统的文件系统缓存，不执行fsync，此时mysql宕机不会丢失数据，而系统宕机会丢失数据

#### 重做日志格式

InnoDB的redo log是固定大小，从头开始写入，写到末尾后又重新从头开始写。每块是由512字节组成，称为block。

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/qsVOIajZw4.png!large)

​		write pos 是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。

​		checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。

　　write pos 和 checkpoint 之间的是 log 上还空着的部分，可以用来记录新的操作。如果 write pos 追上 checkpoint，表示 log 满了，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 checkpoint 推进一下。

　　有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 crash-safe。

##### write pos和checkpoint的作用

1. 缩短数据库的恢复时间

   - 当数据库发生宕机时，数据库不需要重做所有的日志，因为 checkpoint 之前的页都已经刷新回磁

     盘。数据库只需对 checkpoint 后的重做日志进行恢复，这样就大大缩短了恢复的时间。

2. 缓冲池不够用时，将脏页刷新到磁盘

- 当缓冲池不够用时，根据 LRU 算法会溢出最近最少使用的页，若此页为脏页，那么需要强制执行 checkpoint 将脏页刷回磁盘。

3. 重做日志不可用时，刷新脏页

   - 因为当前事务数据库系统对重做日志的设计都是**循环使用**的，并不是让其无限增大的，重做日志可以被重用的部分是指这些重做日志已经不再需要，当数据库发生宕机时，数据库恢复操作不需要这部分的重做日志，因此这部分就可以被覆盖重用。如果重做日志还需要使用，那么必须强制 checkpoint 将脏页刷回磁盘。

> **当内存数据页跟磁盘数据页内容不一致的时候，我们称这个内存页为“脏页”。**
>
> **内存数据写入到磁盘后，内存和磁盘上的数据页的内容就一致了，称为“干净页”** 。



### binlog

https://www.cnblogs.com/ZhuChangwu/p/14125858.html

binlog 是 MySQL 的二进制日志文件，主要**用于数据备份和主从复制**。

常见的作用：

1. delete没加where条件？不慌！binlog可以帮你恢复数据
2. 搭建一套一主两从的MySQL集群，binlog帮你完成主从的数据同步。
3. 审计，通过分析binlog可以排查是否存在SQL注入攻击。

MySQL 在完成一条更新操作后，Server 层还会生成一条 binlog，等之后事务提交的时候，会将该事物执行过程中产生的所有 binlog 统一写 入 binlog 文件。

binlog 文件是记录了所有数据库表结构变更和表数据修改的日志，不会记录查询类的操作，比如 SELECT 和 SHOW 操作。

#### 刷盘机制

> ![binlog cach](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/how_update/binlogcache.drawio.png)
>
> write 指的就是指把日志写入到 binlog 文件，但是并没有把数据持久化到磁盘，因为数据还缓存在文件系统的 page cache 里，write 的写入速度还是比较快的，因为不涉及磁盘 I/O。
>
> fsync 才是将数据持久化到磁盘的操作，这里就会涉及磁盘 I/O，所以频繁的 fsync 会导致磁盘的 I/O 升高。

- sync_binlog = 0 的时候，表示每次提交事务都只 write，不 fsync，后续交由操作系统决定何时将数据持久化到磁盘。（默认）
- sync_binlog = 1 的时候，表示每次提交事务都会 write，然后马上执行 fsync。
- sync_binlog =N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

在MySQL中系统默认的设置是 sync_binlog = 0，也就是不做任何强制性的磁盘刷新指令，这时候的性能是最好的，但是风险也是最大的。因为一旦主机发生异常重启，还没持久化到磁盘的数据就会丢失。

而当 sync_binlog 设置为 1 的时候，是最安全但是性能损耗最大的设置。因为当设置为 1 的时候，即使主机发生异常重启，最多丢失一个事务的 binlog，而已经持久化到磁盘的数据就不会有影响，不过就是对写入性能影响太大。

如果能容少量事务的 binlog 日志丢失的风险，为了提高写入的性能，一般会 sync_binlog 设置为 100~1000 中的某个数值。

### binlog 和 redo log 的异同

1. 适用对象不同

   - binlog 是 Server 层实现的，并且 binlog 不仅针对于 InnoDB，所有引擎都可以使用

   - redo log 是 InnoDB 引擎特有的

2. 两者存储格式不同
   - binlog 存储的是逻辑日志，记录了所有数据库表结构变更和表数据修改的日志，如：对xxx表中的id=yyy的行做做了什么修改，更改后的值是什么。binlog 不会记录你的 select 、show 这类的操作。
   - redo log 则记录物理日志，记录的是在某个数据页做了什么修改，如：对哪个数据页的那个记录做了什么修改。

3. 写入方式不同
   - binlog 是追加写，写满一个文件，就创建一个新的文件继续写，不会覆盖以前的日志，保存的是全量的日志。
   - redo log 是循环写的，空间固定会用完，全部写满就从头开始，保存未被刷入磁盘的脏页日志。
4. 触发存储的时间点不同
   - binlog 只在事务提交后才写入一次
   - redo log 则是在整个事务处理的过程中在不断的写入
5. 用途不同
   - binlog 用于备份恢复、主从复制
   - redo log 用于掉电等故障恢复

> Q：如果不小心整个数据库的数据被删除了，能使用 redo log 文件恢复数据吗？
>
> 不可以使用 redo log 文件恢复，只能使用 binlog 文件恢复。
>
> 因为 redo log 文件是循环写，是会边写边擦除日志的，只记录未被刷入磁盘的数据的物理日志，已经刷入磁盘的数据都会从 redo log 文件里擦除。
>
> binlog 文件保存的是全量的日志，也就是保存了所有数据变更的情况，理论上只要记录在 binlog 上的数据，都可以恢复，所以如果不小心整个数据库的数据被删除了，得用 binlog 文件恢复数据。

### undo log

undo log 是一种用于撤销回退的日志。在事务没提交之前，MySQL 会先记录更新前的数据到 undo log 日志文件里面，当事务回滚时，可以利用 undo log 来进行回滚。它保证了事务的原子性。

undo 日志用于记录事务开始前的状态，用于事务失败时的回滚操作；redo 日志记录事务执行后的状态，用来恢复未写入 data file 的已成功事务更新的数据。例如某一事务的事务序号为 T1，其对数据 X 进行修改，设 X 的原值是 0，修改后的值为 1，那么 undo 日志为 <T1, X, 0>，redo 日志为 < T1, X, 1>。

- 在**插入**一条记录时，要把这条记录的主键值记下来，这样之后回滚时只需要把这个主键值对应的记录**删掉**就好了；
- 在**删除**一条记录时，要把这条记录中的内容都记下来，这样之后回滚时再把由这些内容组成的记录**插入**到表中就好了；
- 在**更新**一条记录时，要把被更新的列的旧值记下来，这样之后回滚时再把这些列**更新为旧值**就好了。

undo log 两大作用：

- **实现事务回滚，保障事务的原子性**。事务处理过程中，如果出现了错误或者用户执 行了 ROLLBACK 语句，MySQL 可以利用 undo log 中的历史数据将数据恢复到事务开始之前的状态。
- **实现 MVCC（多版本并发控制）关键因素之一**。MVCC 是通过 ReadView + undo log 实现的。undo log 为每条记录保存多份历史数据，MySQL 在执行快照读（普通 select 语句）的时候，会根据事务的 Read View 里的信息，顺着 undo log 的版本链找到满足其可见性的记录。

#### undo log 是如何刷盘（持久化到磁盘）的

undo log 和数据页的刷盘策略是一样的，都需要通过 redo log 保证持久化。

buffer pool 中有 undo 页，对 undo 页的修改也都会记录到 redo log。redo log 会每秒刷盘，提交事务时也会刷盘，数据页和 undo 页都是靠这个机制保证持久化的。



==总结：undo log 让 MySQL 有回滚事物的能力，redo log 让 MySQL 有崩溃恢复的能力，binlog 让MySQL 有搭建集群、数据备份、恢复数据的能力。==

## 5. 一条SQL查询语句的执行流程

https://www.cnblogs.com/wupeixuan/p/11626024.html

MySQL 分为 ==Server 层==和==存储引擎层==两部分。

Server层包括连接器、分析器、优化器、执行器、缓存等，其主要涵盖了mysql的核心功能，以及各种内置的函数，作为存储引擎层的上层，横跨了底层的存储引擎。

存储引擎层则是负责mysql的存储和提取工作，由于其具有插件式的能力，支持在innodb、myisam等多个存储引擎层的选取，mysql在5.5版本后将innodb作为默认的存储引擎。手动指定需要在create table语句中使用engine=myisam。

从图中可以看出，不同的存储引擎共用了同一个server层。

![MySql的逻辑架构图](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220224145046)

1. MySQL 客户端与服务器间建立连接，客户端发送一条查询给服务器；
2. 服务器先检查查询缓存，如果命中了缓存，则立刻返回存储在缓存中的结果；否则进入下一阶段；
3. 服务器端进行 SQL 解析、预处理，生成合法的解析树；
4. 再由优化器生成对应的执行计划；
5. MySQL 根据优化器生成的执行计划，调用相应的存储引擎的 API 来执行，并将执行结果返回给客户端。

### 连接器

想要调用mysql的功能，首先就需要进入连接器阶段，其主要负责与客户端建立连接、获取对应权限并维持连接。

```shell
mysql -h 【xxx】 -P 【xxx】-u 【xxx】-p
```

用户成功建立连接后，如果此时管理员对账号的权限进行了修改，并不会影响已经存在的连接，修改完成后，==需要重新连接才会刷新新的权限==。

客户端自动断连时间为8小时，wait_timeout字段控制

在开发中客户端与mysql服务端建立连接的过程比较复杂，建议使用长连接。

但是随着长连接的数量增加，整体mysql的内存也会持续增长，如果长时间不释放连接，大量的连接请求会将系统OOM。

解决上述问题有两种形式：

1. 在使用数据库的过程中定时的去断开连接，go中一般使用gorm，并且会使用连接池，使用连接池技术后，千万不要使用完db后调用db.Close关闭数据库连接，这样会导致整个数据库连接池关闭，导致连接池没有可用的连接
2. mysql的5.7之后的版本，在执行一次比较大的操作是，通过执行 mysql_reset_connection来重新初始化连接资源

### 查询缓存

连接建立完成后，如果此时执行的是select语句，则首先会进入查询缓存，这里查询的一句主要是是否之前执行过这条语句，数据库将结果以key-value的形式存储起来，如果缓存中查询不到，则继续向后执行，查询到则返回。

> 大部分情况下不建议使用查询缓存
>
> 查询缓存的失效非常频繁,只要有对一个表的更新,这个表上所有的查询缓存都会被清空。
>
> 对于频繁有更新的库,查询缓存命中率非常低。

在mysql8.0之前，可以使用SQL_CACHE显示指定语句进行缓存，

```mysq
select SQL_CACHE * from T where name='zhangsan'；
```

在mysql8.0中，直接将查询缓存整个移除。

### 分析器

分析器是针对当前传入的语句进行SQL解析。

首先进行的是”词法分析“，我们输入的SQL语句是由字符串和空格组成，mysql需要分辨出每个字符串所代表的的含义，如select被识别为关键字，id被识别为主键等。

在进行完词法分析后，进入“语法分析”，语法分析会根据语法规则，判断当前sql语句是否合规。

### 优化器

分析器通过后，当前mysql中已经解析出要运行的语句，此时需要对语句进行内部优化，优化器的主要作用是决定当前语句==走哪个索引==，例如如下的join语句

```Plaintext
mysql> select * from t1 join t2 using(ID) where t1.c=10 and t2.d=20;
```

方案一：在表t1中取出c=10的记录，根据ID关联到表t2中，再判断t2的d是否等于20

方案二：从表t2中取出d=20的记录，根据ID关联到表t1中，再判断t1的c是否等于10

两种形式都可以查询到数据，但是基于当前两个表中的数据量不用执行效率也会不同，优化器的作用就是选择一个最优的执行方案。

### 执行器

mysql的优化器告诉系统要如何选取最优的执行模式，执行器则是正式执行语句。

执行语句前会先校验一下当前用户是否有表的执行权限，如果没有权限会直接报错。当用户有权限操作时，执行器会根据存储引擎去调用对一个接口。

执行器的执行流程：（无索引）

- innodb引擎接口取出这个表的第一行，判断id值是是否符合要求，不符合跳过，符合将结果存储
- 继续调用存储引擎接口取下一条，重复，直至取到最后表最后一行
- 将所有满足条件的数据组成最终的结果集返回。

对于有索引的表，执行的逻辑也差不多。第一次调用的是“取满足条件的第一行”的这个接口，之后循环取“满足条件的下一行”这个接口，这些接口都是引擎中已经定义好的。

数据库的慢查询日志中看到一个rows_examined的字段，表示这个语句执行过程中扫描了多少行。这个值就是在执行器每次调用引擎获取数据行的时候累加的。

在有些场景下，执行器调用一次，在引擎内部则扫描了多行，因此**引擎扫描行数跟rows_examined并不是完全相同的。**



## 6. 一条SQL更新语句的执行流程

- 首先是词法分析和语法分析，确定要更新的表、列、行以及新的值，优化器决定使用哪个索引；
2. 开启事务，修改数据之前记录undo log；
3. 锁定数据：为了确保数据的一致性和并发控制；
4. 执行更新操作，更新数据页中的记录，被修改的数据页为脏页，修改会被记录到redo log中，此时为事务的perpare阶段；
5. 提交事务：如果所有的更新操作都完成，那么事务将提交，更新将被永久保存，记录binlog，中途出错会回滚；
6. 释放锁：一旦事务成功提交或回滚，将释放锁，允许其他事务对这些数据进行访问和修改；
7. 返回更新的行数。

![img](https://ask.qcloudimg.com/http-save/yehe-2119492/7c1c8123623aff868379e72eeb547e05.png)

![2dc563be-e25b-40d4-af49-ee2a0d1c663a](https://gitee.com/Transmigration_zhou/pic/raw/master/2dc563be-e25b-40d4-af49-ee2a0d1c663a.jpeg)

这里和查询语句不同的是，在数据更新时还伴随着两个重要的日志模块也就是：redo log（重做日志），binlog（归档日志），mysql之所以能保证数据的可靠性离不开这两个日志。

### 两阶段提交

​	redo log 的写入拆成了 prepare 和 commit 两个阶段（两阶段提交），之所以要这么做是为了实现两份日志逻辑的一致性（即数据要么在两个日志中都存在要么都不存在）。

​	这里就是**数据库具有将数据恢复到几个月内的任意1秒的能力**。

​	mysql在真正的使用中一般会做一定的备份操作，可以一天维度挥着以周维度，如果现在要将数据恢复到指定时间的某1秒时，可以通过binlog日志进行数据重放，重放当当前需要的时间点即可。

#### 如果不用两阶段提交可能造成的问题

以 `update T set c=c+1 where ID=2;` 为例

1. 先写 redo log 后写 binlog。

   假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启。由于 redo log 写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复后这一行 c 的值是 1。但是由于 binlog 没写完就 crash 了，这时候 binlog 里面就没有记录这个语句。因此，之后备份日志的时候，存起来的 binlog 里面就没有这条语句。然后你会发现，如果需要用这个 binlog 来恢复临时库的话，由于这个语句的 binlog 丢失，这个临时库就会少了这一次更新，恢复出来的这一行 c 的值就是 0，与原库的值不同。

2. 先写 binlog 后写 redo log。

   如果在 binlog 写完之后 crash，由于 redo log 还没写，崩溃恢复以后这个事务无效，所以这一行 c 的值是 0。但是 binlog 里面已经记录了 “把 c 从 0 改成 1” 这个日志。所以，在之后用 binlog 来恢复的时候就多了一个事务出来，恢复出来的这一行 c 的值就是 1，与原库的值不同。

如果不使用 “两阶段提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。



## 7. 主从复制 读写分离

主从复制就是用来建立一个和主数据库完全一样的数据库环境，称为从数据库，主数据库一般是准实时的业务数据库。一台服务器充当主服务器，而另外一台服务器充当从服务器。

此时主服务器会将更新信息写入到 binlog 中，并会维护文件的一个索引用来跟踪日志循环，这个日志可以记录并发送到从服务器的更新中去。

一台从服务器连接到主服务器时，从服务器会通知主服务器从服务器的日志文件中读取最后一次成功更新的位置。然后从服务器会接收从哪个时刻起发生的任何更新，然后锁住并等到主服务器通知新的更新。

读写分离简单俩说就是基于主从复制架构，一个主库（负责写），有多个从库（负责读），写完后主库会自动把数据给同步给从库。

### 主从复制是怎么实现

MySQL 的主从复制依赖于 binlog ，也就是记录 MySQL 上的所有变化并以二进制形式保存在磁盘上。复制的过程就是将 binlog 中的数据从主库传输到从库上。

这个过程一般是**异步**的，也就是主库上执行事务操作的线程不会等待复制 binlog 的线程同步完成。

![MySQL 主从复制过程](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/how_update/%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E8%BF%87%E7%A8%8B.drawio.png?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

MySQL 集群的主从复制过程梳理成 3 个阶段：

- **写入 Binlog**：主库写 binlog 日志，提交事务，并更新本地存储数据。
- **同步 Binlog**：把 binlog 复制到所有从库上，每个从库把 binlog 写到暂存日志中。
- **回放 Binlog**：回放 binlog，并更新存储引擎中的数据。

具体详细过程如下：

- 主库将数据库修改写入它的binlog日志中，从库会启动两个线程，I/O 线程和 SQL线程
- I/O 线程连接主库的 log dump 线程，把主库的 binlog日志读取过来写入自己的中继日志（relay log）

- SQL 线程读取 relay log，解析成 sql 将数据插入到从库中，实现主从的数据一致性。

在完成主从复制之后，你就可以在写数据时只写主库，在读数据时只读从库，这样即使写请求会锁表或者锁记录，也不会影响读请求的执行。（读写分离）

![MySQL 主从架构](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/how_update/%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84.drawio.png?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

### 从库是不是越多越好？

不是的。

因为从库数量增加，从库连接上来的 I/O 线程也比较多，**主库也要创建同样多的 log dump 线程来处理复制的请求，对主库资源消耗比较高，同时还受限于主库的网络带宽**。

所以在实际使用中，一个主库一般跟 2～3 个从库（1 套数据库，1 主 2 从 1 备主），这就是一主多从的 MySQL 集群结构。

### 读写分离存在哪些问题

https://mp.weixin.qq.com/s/PhDyDHPzeFeO3JCfbv0mNQ

主从延迟从而导致数据不一致的情况



## 8. limit 和 offset 在大的偏移量的情况下如何优化呢？

https://www.imooc.com/article/43345

使用limit语句时，当数据量偏移量较小时可以直接使用limit，当数据量偏移量较大时，可以适当的使用子查询来做相关的性能优化。



## 9. MySQL隐藏列

1. DB_ROW_ID：隐藏主键，就算表中未定义主键，也会默认存在这个聚簇索引，仅提供给InnoDB构建树结构存储表数据；
2. DB_Deleted_Bit：删除标识，使用delete语句后不会真的删除数据，仅会把标识改为1，主要是为了防止事务执行失败回滚时会发生两次树的结构调整；
3. DB_TRX_ID：最近更新的事务ID，对于所有的写sql，会为其分配一个顺序自增的事务id。如果是select语句，事务id为0；
4. DB_ROLL_PTR：回滚指针。



## 10. 分库分表

![img](https://pic3.zhimg.com/v2-d57bf722f4372625df9912cd30bda856_r.jpg)

### 垂直分表

垂直分表：**将一个表按照字段分成多个表** ，每个表存储其中一部分字段。

根据数据是否是热点数据划分。热点数据即经常查询、更新频繁的列。

例如一个订单状态信息会频繁进行更新、订单金额在列表会频繁被查询到作为热点数据，而下单地址、手机号码等信息基本不会改变或者改变次数很少作为非热点数据。

#### 好处

1. 更好地提升热门数据的查询效率。
2. 行数据变小，数据库IO效率高。

### 水平分表

水平分表：把同一个表的数据按照一定规则拆分到多个表中（基于数据划分，表结构相同）。一般意义上的分库分表指的就是水平分表。

#### 策略

1. 范围切分：每个表存放1000w数据，满了再建新表。
   - 优点：后续扩容方便，无需进行数据迁移
   - 缺点：读写流量全集中在最新的表上，没有起到流量均摊的效果
   - 适用场景：归档类功能，比如操作日志按日期分表，每个月一张表

  2. 中间表映射（路由表）：每次先查路由表确定完整数据所在的库表，再到该库表查完整数据。
      - 优点：灵活，可以随意设置路由规则
      - 缺点：引入了额外的单点查询，路由表也可能非常大，又存在分库分表的问题
      - 适用场景：对数据量特别大的用户，为其建立路由规则，单独放到一张大表中，其他普通用户还走正常的hash切分逻辑
  3. Hash切分：对分表键进行一定的运算（通常是取模），从而决定路由到哪个库哪个表。
      - 优点：数据分片与读写流量分摊都比较均匀
      - 缺点：可能存在跨节点查询和分页等问题
      - 适用场景：目前大多数互联网服务使用的都是hash切分

### 分库分表存在的问题

1. 分布式事务问题

   在提交订单时，除了创建订单之外，我们还需要扣除相应的库存。而订单表和库存表由于垂直分库，位于不同的库中，这时我们需要通过分布式事务来保证提交订单时的事务完整性。

2. 跨节点 JOIN 查询问题

   用户在查询订单时，我们往往需要通过表连接获取到商品信息，而商品信息表可能在另外一个库中。

   解决该问题的普遍做法是分两次查询实现：在第一次查询的结果集中找出关联数据的id，根据这些id发起第二次请求得到关联数据。

   **字段冗余**：把需要关联的字段放入主表中，避免 join 操作。

3. 跨节点排序、分页、函数计算问题

   由于这类问题都需要基于全部数据集合进行计算。解决方案：与解决跨节点join问题的类似，分别在各个节点上得到结果后在应用程序端进行合并。和 join 不同的是每个结点的查询可以并行执行，因此速度要比单一大表快很多。但如果结果集很大，对应用程序内存的消耗是一个问题。

4. 全局主键 ID 问题

   1. UUID 实现全局 ID

      - 优点：本地生成ID，不需要远程调用，全局唯一不重复。
      - 缺点：占用空间大，连续性差，不适合作为主键索引。

   2. redis 生成 ID

      基于 Redis 分布式锁实现一个递增的主键 ID，这种方式可以保证主键是一个整数且有一定的连续性，但分布式锁存在一定的性能消耗。

   3. 数据库自增 ID

      在分库分表表后使用数据库自增ID，需要一个专门用于生成主键的库，每次服务接收到请求，先向这个库中插入一条没有意义的数据，获取一个数据库自增的ID，利用这个ID去分库分表中写数据。

   4. SnowFlake（雪花算法）

      在分布式系统中产生全局唯一且趋势递增的ID，值类型为64位长整型。

      >  1. 第1位始终为0，可以看作是符号位不可用；
      >  2. 第2位开始的41位是毫秒级时间戳，可用年限为69年；
      >  3. 中间的10位表示机器数，即2^10=1024台机器；
      >  4. 最后12位是自增序列，2^12=4096个数。






## 11. 数据库慢查询问题排查及解决方案

### 问题情况

总的来说，我们应该从==索引、架构、网络、I/O吞吐量、内存、锁、SQL语句==等各个方面去分析。

如果==大多数情况下都正常，偶尔很慢==，则可能是：数据库在刷新脏页，例如 redo log 写满了需要同步到磁盘；或者执行的时候，**遇到锁**，如表锁、行锁。

也有可能是当时网络不好，内存不足，I/O吞吐量小，不过一般不会出现这种情况。

如果这条 SQL 语句==一直执行的很慢==，可能是没有用上索引或则索引失效。



### 定位慢查询 SQL

1. 首先数据库中设置 SQL 慢查询，设置慢查询的超时时间。
2. 然后当出现慢查询时，我们可以去==分析慢查询日志==。我们可以使用 show processlist 命令定位低效率执行SQL，也可以==用 explain 分析 SQL 的执行计划==来查看是否使用索引。



### 如何优化一条慢查询？

   1. 索引角度：在经常用于检索的列上创建索引（where 和 order by）。
   2. SQL 语句角度
      - 只返回必要的列：最好不要使用 select *
      - 分页查询优化：通过 where 和 limit 等进行限制
      - 批量执行插入语句
   3. 使用分库分表（考虑分布式事务）
   4. 缓存重复的数据：借助缓存将经常被重复查询且不容易改变的数据进行存储，避免对数据库的重复查询
   5. 采用读/写分离（主库写，从库读）集群模式



## 12. 给你10个数据库服务器，每个只能接500的qps，现在要实现4000qps，要怎么做？

- 主从数据库：使用 binlog 来同步主数据库的更改到从数据库。
- 使用负载均衡：在数据库前设置负载均衡，使得读请求平均分配到从数据库。

> 负载均衡算法：随机、轮询、权重轮询、一致性哈希



## 13. 查看 Mysql 的主从状态指令

```mysql
show slave status
```

1、Slave_IO_Running：指示复制 I/O 进程是否正在运行。如果值为 "Yes"，表示正常运行。
2、Slave_SQL_Running：指示复制 SQL 进程是否正在运行。如果值为 "Yes"，表示正常运行。
3、Seconds_Behind_Master：表示从服务器与主服务器之间的复制延迟（以秒为单位）。较小的延迟表示较好的主从同步。
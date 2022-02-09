# 一文彻底读懂MySQL事务的四大隔离级别

![img](/Users/mark/typora_workspace/NoteBook/images/171498a008c91b4c~tplv-t2oaga2asx-watermark.image)



## 事务

### 什么是事务？

事务，由一个有限的数据库操作序列构成，这些操作要么全部执行,要么全部不执行，是一个不可分割的工作单位。

> 假如A转账给B 100 元，先从A的账户里扣除 100 元，再在 B 的账户上加上 100 元。如果扣完A的100元后，还没来得及给B加上，银行系统异常了，最后导致A的余额减少了，B的余额却没有增加。所以就需要事务，将A的钱回滚回去，就是这么简单。

### 事务的四大特性



![img](/Users/mark/typora_workspace/NoteBook/images/1712b5213446a402~tplv-t2oaga2asx-watermark.image)



- **原子性：** 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部都执行，要么都不执行。
- **一致性：** 指在事务开始之前和事务结束以后，数据不会被破坏，假如A账户给B账户转10块钱，不管成功与否，A和B的总金额是不变的。
- **隔离性：** 多个事务并发访问时，事务之间是相互隔离的，一个事务不应该被其他事务干扰，多个并发事务之间要相互隔离。。
- **持久性：** 表示事务完成提交后，该事务对数据库所作的操作更改，将持久地保存在数据库之中。

## 事务并发存在的问题

事务并发执行存在什么问题呢，换句话说就是，一个事务是怎么干扰到其他事务的呢？看例子吧~

假设现在有表：

```
CREATE TABLE `account` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `balance` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `un_name_idx` (`name`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
复制代码
```

表中有数据：



![img](/Users/mark/typora_workspace/NoteBook/images/171381960fd0f119~tplv-t2oaga2asx-watermark.image)



### 脏读（dirty read）

假设现在有两个事务A、B：

- 假设现在A的余额是100，事务A正在准备查询Jay的余额
- 这时候，事务B先扣减Jay的余额，扣了10
- 最后A 读到的是扣减后的余额



![img](/Users/mark/typora_workspace/NoteBook/images/1713824c77723cd4~tplv-t2oaga2asx-watermark.image)



由上图可以发现，事务A、B交替执行，事务A被事务B干扰到了，因为事务A读取到事务B未提交的数据,这就是**脏读**。

### 不可重复读（unrepeatable read）

假设现在有两个事务A和B：

- 事务A先查询Jay的余额，查到结果是100
- 这时候事务B 对Jay的账户余额进行扣减，扣去10后，提交事务
- 事务A再去查询Jay的账户余额发现变成了90



![img](/Users/mark/typora_workspace/NoteBook/images/1713829b86401900~tplv-t2oaga2asx-watermark.image)



事务A又被事务B干扰到了！在事务A范围内，两个相同的查询，读取同一条记录，却返回了不同的数据，这就是**不可重复读**。

### 幻读

假设现在有两个事务A、B：

- 事务A先查询id大于2的账户记录，得到记录id=2和id=3的两条记录
- 这时候，事务B开启，插入一条id=4的记录，并且提交了
- 事务A再去执行相同的查询，却得到了id=2,3,4的3条记录了。



![img](/Users/mark/typora_workspace/NoteBook/images/171382b2bdd28029~tplv-t2oaga2asx-watermark.image)



事务A查询一个范围的结果集，另一个并发事务B往这个范围中插入/删除了数据，并静悄悄地提交，然后事务A再次查询相同的范围，两次读取得到的结果集不一样了，这就是**幻读**。

## 事务的四大隔离级别实践

既然并发事务存在**脏读、不可重复、幻读**等问题，InnoDB实现了哪几种事务的隔离级别应对呢？

- 读未提交（Read Uncommitted）
- 读已提交（Read Committed）
- 可重复读（Repeatable Read）
- 串行化（Serializable）

### 读未提交（Read Uncommitted）

想学习一个知识点，最好的方式就是实践之。好了，我们去数据库给它设置**读未提交**隔离级别，实践一下吧~



![img](/Users/mark/typora_workspace/NoteBook/images/17148fd0f161aea8~tplv-t2oaga2asx-watermark.image)



先把事务隔离级别设置为read uncommitted，开启事务A，查询id=1的数据

```
set session transaction isolation level read uncommitted;
begin;
select * from account where id =1;
复制代码
```

结果如下：



![img](/Users/mark/typora_workspace/NoteBook/images/17148f9415ffc166~tplv-t2oaga2asx-watermark.image)

这时候，另开一个窗口打开mysql，也把当前事务隔离级别设置为read uncommitted，开启事务B，执行更新操作



```
set session transaction isolation level read uncommitted;
begin;
update account set balance=balance+20 where id =1;
复制代码
```

接着回事务A的窗口，再查account表id=1的数据，结果如下：



![img](/Users/mark/typora_workspace/NoteBook/images/17148faf1e5d5f48~tplv-t2oaga2asx-watermark.image)



可以发现，在**读未提交（Read Uncommitted）** 隔离级别下，一个事务会读到其他事务未提交的数据的，即存在**脏读**问题。事务B都还没commit到数据库呢，事务A就读到了，感觉都乱套了。。。实际上，读未提交是隔离级别最低的一种。

### 已提交读（READ COMMITTED）

为了避免脏读，数据库有了比**读未提交**更高的隔离级别，即**已提交读**。



![img](/Users/mark/typora_workspace/NoteBook/images/17148c908b12084b~tplv-t2oaga2asx-watermark.image)



把当前事务隔离级别设置为已提交读（READ COMMITTED），开启事务A，查询account中id=1的数据

```
set session transaction isolation level read committed;
begin;
select * from account where id =1;
复制代码
```

另开一个窗口打开mysql，也把事务隔离级别设置为read committed，开启事务B，执行以下操作

```
set session transaction isolation level read committed;
begin;
update account set balance=balance+20 where id =1;
复制代码
```

接着回事务A的窗口，再查account数据，发现数据没变：



![img](/Users/mark/typora_workspace/NoteBook/images/1713d69e118832d2~tplv-t2oaga2asx-watermark.image)



我们再去到事务B的窗口执行commit操作：

```
commit;
复制代码
```

最后回到事务A窗口查询，发现数据变了：



![img](/Users/mark/typora_workspace/NoteBook/images/1713d68ad5a2fd47~tplv-t2oaga2asx-watermark.image)



由此可以得出结论，隔离级别设置为**已提交读（READ COMMITTED）** 时，已经不会出现脏读问题了，当前事务只能读取到其他事务提交的数据。但是，你站在事务A的角度想想，存在其他问题吗？

**提交读的隔离级别会有什么问题呢？**

在同一个事务A里，相同的查询sql，读取同一条记录（id=1），读到的结果是不一样的，即**不可重复读**。所以，隔离级别设置为read committed的时候，还会存在**不可重复读**的并发问题。

### 可重复读（Repeatable Read）

如果你的老板要求，在同个事务中，查询结果必须是一致的，即老板要求你解决不可重复的并发问题，怎么办呢？老板，臣妾办不到？来实践一下**可重复读（Repeatable Read）** 这个隔离级别吧~



![img](/Users/mark/typora_workspace/NoteBook/images/17144b324064255a~tplv-t2oaga2asx-watermark.image)



哈哈，步骤1、2、6的查询结果都是一样的，即**repeatable read解决了不可重复读问题**，是不是心里美滋滋的呢，终于解决老板的难题了~

**RR级别是否解决了幻读问题呢？**

再来看看网上的一个热点问题，有关于RR级别下，是否解决了幻读问题？我们来实践一下：



![img](/Users/mark/typora_workspace/NoteBook/images/1714911d7b42c350~tplv-t2oaga2asx-watermark.image)

由图可得，步骤2和步骤6查询结果集没有变化，看起来RR级别是已经解决幻读问题了~ 但是呢，**RR级别还是存在这种现象**：

![img](/Users/mark/typora_workspace/NoteBook/images/17142571375bd709~tplv-t2oaga2asx-watermark.image)



其实，上图如果事务A中，没有`update account set balance=200 where id=5;`这步操作，`select * from account where id>2`查询到的结果集确实是不变，这种情况没有**幻读**问题。但是，有了update这个骚操作，同一个事务，相同的sql，查出的结果集不同，这个是符合了**幻读**的定义~

这个问题，亲爱的朋友，你觉得它算幻读问题吗？

### 串行化（Serializable）

前面三种数据库隔离级别，都有一定的并发问题，现在放大招吧，实践SERIALIZABLE隔离级别。

把事务隔离级别设置为Serializable，开启事务A，查询account表数据

```
set session transaction isolation level serializable;
select @@tx_isolation;
begin;
select * from account;
复制代码
```

另开一个窗口打开mysql，也把事务隔离级别设置为Serializable，开启事务B，执行插入一条数据：

```
set session transaction isolation level serializable;
select @@tx_isolation;
begin;
insert into account(id,name,balance) value(6,'Li',100);
复制代码
```

执行结果如下：



![img](/Users/mark/typora_workspace/NoteBook/images/1714282f3cb7f7fa~tplv-t2oaga2asx-watermark.image)



由图可得，当数据库隔离级别设置为serializable的时候，事务B对表的写操作，在等事务A的读操作。其实，这是隔离级别中最严格的，读写都不允许并发。它保证了最好的安全性，性能却是个问题~

## MySql隔离级别的实现原理

实现隔离机制的方法主要有两种：

- 读写锁
- 一致性快照读，即 MVCC

MySql使用不同的锁策略(Locking Strategy)/MVCC来实现四种不同的隔离级别。RR、RC的实现原理跟MVCC有关，RU和Serializable跟锁有关。

### 读未提交（Read Uncommitted）

**官方说法：**

> SELECT statements are performed in a nonlocking fashion, but a possible earlier version of a row might be used. Thus, using this isolation level, such reads are not consistent.

读未提交，采取的是读不加锁原理。

- 事务读不加锁，不阻塞其他事务的读和写
- 事务写阻塞其他事务写，但不阻塞其他事务读；

### 串行化（Serializable)

**官方的说法:**

> InnoDB implicitly converts all plain SELECT statements to SELECT ... FOR SHARE if autocommit is disabled. If autocommit is enabled, the SELECT is its own transaction. It therefore is known to be read only and can be serialized if performed as a consistent (nonlocking) read and need not block for other transactions. (To force a plain SELECT to block if other transactions have modified the selected rows, disable autocommit.)

- 所有SELECT语句会隐式转化为`SELECT ... FOR SHARE`，即加共享锁。
- 读加共享锁，写加排他锁，读写互斥。如果有未提交的事务正在修改某些行，所有select这些行的语句都会阻塞。

### MVCC的实现原理

MVCC，中文叫**多版本并发控制**，它是通过读取历史版本的数据，来降低并发事务冲突，从而提高并发性能的一种机制。它的实现依赖于**隐式字段、undo日志、快照读&当前读、Read View**，因此，我们先来了解这几个知识点。

#### 隐式字段

对于InnoDB存储引擎，每一行记录都有两个隐藏列**DB_TRX_ID、DB_ROLL_PTR**，如果表中没有主键和非NULL唯一键时，则还会有第三个隐藏的主键列**DB_ROW_ID**。

- DB_TRX_ID，记录每一行最近一次修改（修改/更新）它的事务ID，大小为6字节；
- DB_ROLL_PTR，这个隐藏列就相当于一个指针，指向回滚段的undo日志，大小为7字节；
- DB_ROW_ID，单调递增的行ID，大小为6字节；



![img](/Users/mark/typora_workspace/NoteBook/images/1714794c14a7e14f~tplv-t2oaga2asx-watermark.image)



#### undo日志

> - 事务未提交的时候，修改数据的镜像（修改前的旧版本），存到undo日志里。以便事务回滚时，恢复旧版本数据，撤销未提交事务数据对数据库的影响。
> - undo日志是逻辑日志。可以这样认为，当delete一条记录时，undo log中会记录一条对应的insert记录，当update一条记录时，它记录一条对应相反的update记录。
> - 存储undo日志的地方，就是**回滚段**。

多个事务并行操作某一行数据时，不同事务对该行数据的修改会产生多个版本，然后通过回滚指针（DB_ROLL_PTR）连一条**Undo日志链**。

我们通过例子来看一下~

```
mysql> select * from account ;
+----+------+---------+
| id | name | balance |
+----+------+---------+
|  1 | Jay  |     100 |
+----+------+---------+
1 row in set (0.00 sec)
复制代码
```

- 假设表accout现在只有一条记录，插入该该记录的事务Id为100
- 如果事务B（事务Id为200），对id=1的该行记录进行更新，把balance值修改为90

事务B修改后，形成的**Undo Log链**如下：



![img](/Users/mark/typora_workspace/NoteBook/images/171478d1a4a873d6~tplv-t2oaga2asx-watermark.image)



#### 快照读&当前读

**快照读：**

读取的是记录数据的可见版本（有旧的版本），不加锁,普通的select语句都是快照读,如：

```
select * from account where id>2;
复制代码
```

**当前读：**

读取的是记录数据的最新版本，显示加锁的都是当前读

```
select * from account where id>2 lock in share mode;
select * from  account where id>2 for update;
复制代码
```

#### Read View

- Read View就是事务执行**快照读**时，产生的读视图。
- 事务执行快照读时，会生成数据库系统当前的一个快照，记录当前系统中还有哪些活跃的读写事务，把它们放到一个列表里。
- Read View主要是用来做可见性判断的，即判断当前事务可见哪个版本的数据~

为了下面方便讨论Read View可见性规则，先定义几个变量

> - m_ids:当前系统中那些活跃的读写事务ID,它数据结构为一个List。
> - min_limit_id:m_ids事务列表中，最小的事务ID
> - max_limit_id:m_ids事务列表中，最大的事务ID

- 如果DB_TRX_ID < min_limit_id，表明生成该版本的事务在生成ReadView前已经提交(因为事务ID是递增的)，所以该版本可以被当前事务访问。
- 如果DB_TRX_ID > m_ids列表中最大的事务id，表明生成该版本的事务在生成ReadView后才生成，所以该版本不可以被当前事务访问。
- 如果 min_limit_id =<DB_TRX_ID<= max_limit_id,需要判断m_ids.contains(DB_TRX_ID)，如果在，则代表Read View生成时刻，这个事务还在活跃，还没有Commit，你修改的数据，当前事务也是看不见的；如果不在，则说明，你这个事务在Read View生成之前就已经Commit了，修改的结果，当前事务是能看见的。

**注意啦！！** RR跟RC隔离级别，最大的区别就是：**RC每次读取数据前都生成一个ReadView，而RR只在第一次读取数据时生成一个ReadView**。

### 已提交读（READ COMMITTED） 存在不可重复读问题的分析历程

我觉得理解一个新的知识点，最好的方法就是**居于目前存在的问题/现象，去分析它**的来龙去脉~ RC的实现也跟MVCC有关，RC是存在重复读并发问题的，所以我们来分析一波RC吧，先看一下执行流程

![img](/Users/mark/typora_workspace/NoteBook/images/17148cab92213e82~tplv-t2oaga2asx-watermark.image)

假设现在系统里有A，B两个事务在执行，事务ID分别为100、200，并且假设存在的老数据，插入事务ID是50哈~



**事务A 先执行查询1的操作**

```
# 事务A，Transaction ID 100
begin ;
查询1：select *  from account WHERE id = 1; 
复制代码
```

**事务 B 执行更新操作，id =1记录的undo日志链如下**

```
begin;
update account set balance =balance+20 where id =1;
复制代码
```



![img](/Users/mark/typora_workspace/NoteBook/images/17149227dbd59f46~tplv-t2oaga2asx-watermark.image)

**回到事务A，执行查询2的操作**



```
begin ;
查询1：select *  from account WHERE id = 1; 
查询2：select *  from account WHERE id = 1; 
复制代码
```

**查询2执行分析：**

- 事务A在执行到SELECT语句时，重新生成一个ReadView，因为事务B（200）在活跃，所以ReadView的m_ids列表内容就是[200]
- 由上图undo日志链可得，最新版本的balance为1000，它的事务ID为200，在活跃事务列表里，所以当前事务（事务A）不可见。
- 我们继续找下一个版本，balance为100这行记录，事务Id为50，小于活跃事务ID列表最小记录200，所以这个版本可见，因此，查询2的结果，就是返回balance=100这个记录~~

**我们回到事务B，执行提交操作，这时候undo日志链不变**

```
begin;
update account set balance =balance+20 where id =1;
commit
复制代码
```



![img](/Users/mark/typora_workspace/NoteBook/images/1714924077c2775d~tplv-t2oaga2asx-watermark.image)



**再次回到事务A，执行查询3的操作**

```
begin ;
查询1：select *  from account WHERE id = 1; 
查询2：select *  from account WHERE id = 1; 
查询3：select *  from account WHERE id = 1; 
复制代码
```

**查询3执行分析：**

- 事务A在执行到SELECT语句时，重新生成一个ReadView，因为事务B（200）已经提交，不载活跃，所以ReadView的m_ids列表内容就是空的了。
- 所以事务A直接读取最新纪录，读取到balance =120这个版本的数据。

所以，这就是RC存在不可重复读问题的过程啦~有不理解的地方可以多读几遍哈~

### 可重复读（Repeatable Read）解决不可重复读问题的一次分析

我们再来分析一波，RR隔离级别是如何解决不可重复读并发问题的吧~

你可能会觉得两个并发事务的例子太简单了，好的！我们现在来点刺激的，开启三个事务~



![img](/Users/mark/typora_workspace/NoteBook/images/171488edb21fe523~tplv-t2oaga2asx-watermark.image)

假设现在系统里有A，B，C两个事务在执行，事务ID分别为100、200，300，存量数据插入的事务ID是50~



```
# 事务A，Transaction ID 100
begin ;
UPDATE account SET balance = 1000  WHERE id = 1;
复制代码
# 事务B，Transaction ID 200
begin ; //开个事务，占坑先
复制代码
```

**这时候，account表中，id =1记录的undo日志链如下：**

![img](/Users/mark/typora_workspace/NoteBook/images/1714881ff86628eb~tplv-t2oaga2asx-watermark.image)



```
# 事务C，Transaction ID 300
begin ;
//查询1：select * from  account WHERE id = 1;
复制代码
```

**查询1执行过程分析：**

- 事务C在执行SELECT语句时，会先生成一个ReadView。因为事务A（100）、B（200）在活跃，所以ReadView的m_ids列表内容就是[100, 200]。
- 由上图undo日志链可得，最新版本的balance为1000，它的事务ID为100，在活跃事务列表里，所以当前事务（事务C）不可见。
- 我们继续找下一个版本，balance为100这行记录，事务Id为50，小于活跃事务ID列表最小记录100，所以这个版本可见，因此，查询1的结果，就是返回balance=100这个记录~~

**接着，我们把事务A提交一下：**

```
# 事务A，Transaction ID 100
begin ;
UPDATE account SET balance = 1000  WHERE id = 1;
commit;
复制代码
```

**在事务B中，执行更新操作，把id=1的记录balance修改为2000，更新完后，undo 日志链如下：**

```
# 事务B，Transaction ID 200
begin ; //开个事务，占坑先
UPDATE account SET balance = 2000  WHERE id = 1;
复制代码
```



![img](/Users/mark/typora_workspace/NoteBook/images/17148a0b1e007c1f~tplv-t2oaga2asx-watermark.image)



**回到事务C，执行查询2**

```
# 事务C，Transaction ID 300
begin ;
//查询1：select * from  account WHERE id = 1;
//查询2：select * from  account WHERE id = 1;
复制代码
```

**查询2:执行分析：**

- 在RR 级别下，执行查询2的时候，因为**前面ReadView已经生成过了，所以直接服用之前的ReadView**，活跃事务列表为[100,200].
- 由上图undo日志链可得，最新版本的balance为2000，它的事务ID为200，在活跃事务列表里，所以当前事务（事务C）不可见。
- 我们继续找下一个版本，balance为1000这行记录，事务Id为100,也在活跃事务列表里，所以当前事务（事务C）不可见。
- 继续找下一个版本，balance为100这行记录，事务Id为50，小于活跃事务ID列表最小记录100，所以这个版本可见，因此，查询2的结果，也是返回balance=100这个记录~~

### 锁相关概念补充（附）：

#### 共享锁与排他锁

InnoDB 实现了标准的行级锁，包括两种：共享锁（简称 s 锁）、排它锁（简称 x 锁）。

- 共享锁（S锁）：允许持锁事务读取一行。
- 排他锁（X锁）：允许持锁事务更新或者删除一行。

如果事务 T1 持有行 r 的 s 锁，那么另一个事务 T2 请求 r 的锁时，会做如下处理：

- T2 请求 s 锁立即被允许，结果 T1 T2 都持有 r 行的 s 锁
- T2 请求 x 锁不能被立即允许

如果 T1 持有 r 的 x 锁，那么 T2 请求 r 的 x、s 锁都不能被立即允许，T2 必须等待T1释放 x 锁才可以，因为X锁与任何的锁都不兼容。

#### 记录锁（Record Locks）

- 记录锁是最简单的行锁，**仅仅锁住一行**。如：`SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE`
- 记录锁**永远都是加在索引上**的，即使一个表没有索引，InnoDB也会隐式的创建一个索引，并使用这个索引实施记录锁。
- 会阻塞其他事务对其插入、更新、删除

记录锁的事务数据（关键词：`lock_mode X locks rec but not gap`），记录如下：

```
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` 
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
复制代码
```

#### 间隙锁（Gap Locks）

- 间隙锁是一种加在两个索引之间的锁，或者加在第一个索引之前，或最后一个索引之后的间隙。
- 使用间隙锁锁住的是一个区间，而不仅仅是这个区间中的每一条数据。
- 间隙锁只阻止其他事务插入到间隙中，他们不阻止其他事务在同一个间隙上获得间隙锁，所以 gap x lock 和 gap s lock 有相同的作用。

#### Next-Key Locks

- Next-key锁是记录锁和间隙锁的组合，它指的是加在某条记录以及这条记录前面间隙上的锁。

### RC级别存在幻读分析

因为RC是存在幻读问题的，所以我们先切到RC隔离级别，分析一波~

假设account表有4条数据。

- 开启事务A，执行当前读，查询id>2的所有记录。
- 再开启事务B，插入id=5的一条数据。
- 事务B插入数据成功后，再修改id=3的记录
- 回到事务A,再次执行id>2的当前读查询



![img](/Users/mark/typora_workspace/NoteBook/images/171496a6382f56e7~tplv-t2oaga2asx-watermark.image)



- 事务B可以插入id=5的数据，却更新不了id=3的数据，陷入阻塞。证明事务A在执行当前读的时候在id =3和id=4这两条记录上加了锁，但是并没有对 id > 2 这个范围加锁~
- 事务B陷入阻塞后，切回事务A执行当前读操作时，死锁出现。因为事务B在 insert 的时候，会在新纪录（id=5）上加锁，所以事务A再次执行当前读，想获取id> 3 的记录，就需要在 id=3,4,5 这3条记录上加锁，但是 id = 5这条记录已经被事务B 锁住了，于是事务A被事务B阻塞，同时事务B还在等待 事务A释放 id = 3上的锁，最终产生了死锁。



![img](/Users/mark/typora_workspace/NoteBook/images/171497ae540bdebf~tplv-t2oaga2asx-watermark.image)



因此，我们可以发现，RC隔离级别下，加锁的select, update, delete等语句，使用的是记录锁，其他事务的插入依然可以执行，因此会存在幻读~

### RR 级别解决幻读分析

因为RR是解决幻读问题的，怎么解决的呢，分析一波吧~

假设account表有4条数据，RR级别。

- 开启事务A，执行当前读，查询id>2的所有记录。

- 再开启事务B，插入id=5的一条数据。

    ![img](/Users/mark/typora_workspace/NoteBook/images/17149563ecc072ba~tplv-t2oaga2asx-watermark.image)

    可以发现，事务B执行插入操作时，阻塞了~因为事务A在执行select ... lock in share mode的时候，不仅在 id = 3,4 这2条记录上加了锁，而且在id > 2 这个范围上也加了间隙锁。

因此，我们可以发现，RR隔离级别下，加锁的select, update, delete等语句，会使用间隙锁+ 临键锁，锁住索引记录之间的范围，避免范围间插入记录，以避免产生幻影行记录。


作者：捡田螺的小男孩
链接：https://juejin.cn/post/6844904115353436174
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
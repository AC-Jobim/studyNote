

# 一、MVCC解析

> 好的博客：[正确的理解MySQL的MVCC及实现原理](https://blog.csdn.net/SnailMann/article/details/94724197?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.baidujs&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.baidujs)
>
> [通俗易懂数据库MVCC讲解，后悔没早点学 ](https://mp.weixin.qq.com/s/oOL4yradD5w73VsrfoyneA)

## 1.1 MVCC介绍

MVCC（Multi-Version Concurrency Control）即**多版本并发控制**，读取数据时通过一种类似快照的方式（**版本链**）将数据保存下来，这样**读锁就和写锁不冲突了**。通俗讲就是“通过多个版本的记录来实现并发控制” 。不同的事务session会看到自己特定版本的数据，版本链。

> **MVCC只在 READ COMMITTED 和 REPEATABLE READ 两个隔离级别下工作**。其他两个隔离级别够和MVCC不兼容, 因为 READ UNCOMMITTED 总是读取最新的数据行, 而不是符合当前事务版本的数据行。而 SERIALIZABLE 则会对所有读取的行都加锁。



**MVCC 带来的好处是？**

* 多版本并发控制（MVCC）是一种用来**解决读-写冲突的无锁并发控制**，也就是为事务分配单向增长的时间戳，为每个修改保存一个版本，版本与事务时间戳关联，读操作只读该事务开始前的数据库的快照。 所以 MVCC 可以为数据库解决以下问题
  * 在并发读写数据库时，可以做到**在读操作时不用阻塞写操作，写操作也不用阻塞读操作**，提高了数据库并发读写的性能
  * 同时还可以解决脏读，幻读，不可重复读等事务隔离问题，但不能解决更新丢失问题



**当前读和快照读?**

* **当前读**

  它读取的数据库记录，都是`当前最新`的`版本`，会对当前读取的数据进行`加锁`，防止其他事务修改数据。是`悲观锁`的一种操作。

  如下操作都是当前读：

  * select lock in share mode (共享锁)
  * select for update (排他锁)
  * update (排他锁) 
  * insert (排他锁)
  * delete (排他锁)
  * 串行化事务隔离级别

* **快照读**

  快照读的实现是基于`多版本`并发控制，即MVCC，既然是多版本，那么快照读读到的数据不一定是当前最新的数据，有可能是之前`历史版本`的数据。

  如下操作是快照读：

  * 不加锁的select操作（注：事务级别不是串行化）



## 1.2 MVCC的原理

MVCC的目的就是多版本并发控制，在数据库中的实现，就是为了解决`读写冲突`，它的实现原理主要是依赖记录中的 **`版本链`**，**`undo日志`** ，**`Read View`** 来实现的 



### 1.2.1 版本链

我们数据库中的每行数据，除了我们肉眼看见的数据，还有几个`隐藏字段`，得开`天眼`才能看到。分别是`db_trx_id`、`db_roll_pointer`、`db_row_id`。

- `db_trx_id`（最近修改的事务ID）

  6byte，最近修改(修改/插入)`事务ID`：记录`创建`这条记录/`最后一次修改`该记录的`事务ID`。

- `db_roll_pointer`（回滚指针）

  7byte，`回滚指针`，指向`这条记录`的`上一个版本`（存储于rollback segment里）

- `db_row_id`

  6byte，隐含的`自增ID`（隐藏主键），如果数据表`没有主键`，InnoDB会自动以db_row_id产生一个`聚簇索引`。

- 实际还有一个`删除flag`隐藏字段, 记录被`更新`或`删除`并不代表真的删除，而是`删除flag`变了

> ![image-20210917130903367](https://gitee.com/jobim/blogimage/raw/master/img/20210917130903.png)
>
>  如上图，`db_row_id`是数据库默认为该行记录生成的`唯一隐式主键`，`db_trx_id`是当前操作该记录的`事务ID`，而`db_roll_pointer`是一个`回滚指针`，用于配合`undo日志`，指向上一个`旧版本`。



### 1.2.2 **undo log 日志**

Undo log 主要用于`记录`数据被`修改之前`的日志，在表信息修改之前先会把数据拷贝到`undo log`里。

当`事务`进行`回滚时`可以通过undo log 里的日志进行`数据还原`。

**Undo log 的用途**

- 保证`事务`进行`rollback`时的`原子性和一致性`，当事务进行`回滚`的时候可以用undo log的数据进行`恢复`。
- 用于MVCC`快照读`的数据，在MVCC多版本控制中，通过读取`undo log`的`历史版本数据`可以实现`不同事务版本号`都拥有自己`独立的快照数据版本`。

**undo log主要分为两种：**

- `insert undo log`

  代表事务在insert新记录时产生的undo log , 只在事务回滚时需要，并且在事务提交后可以被立即丢弃

- `update undo log`（主要）

  事务在进行update或delete时产生的undo log ; 不仅在事务回滚时需要，在快照读时也需要；

  所以不能随便删除，只有在快速读或事务回滚不涉及该日志时，对应的日志才会被purge线程统一清除

> purge
>
> * 从前面的分析可以看出，为了实现 InnoDB 的 MVCC 机制，更新或者删除操作都只是设置一下老记录的 deleted_bit ，并不真正将过时的记录删除。
>
> * 为了节省磁盘空间，InnoDB 有专门的 purge 线程来清理 deleted_bit 为 true 的记录。为了不影响 MVCC 的正常工作，purge 线程自己也维护了一个read view（这个 read view 相当于系统中最老活跃事务的 read view ）;如果某个记录的 deleted_bit 为 true ，并且 DB_TRX_ID 相对于 purge 线程的 read view 可见，那么这条记录一定是可以被安全清除的。



**undo log执行流程演示：**

1. 比如一个**有个事务插入 person 表插入了一条新记录**，记录如下，name 为 Jerry , age 为 24 岁，隐式主键是 1，事务 ID和回滚指针，我们假设为 NULL

   ![image-20210918132218787](https://gitee.com/jobim/blogimage/raw/master/img/20210918132218.png)



2. **现在来了一个`事务1`对该记录的 name 做出了修改，改为 Tom**

   * 事务 1修改该行(记录)数据时，数据库会先对该行加**排他锁**
   * **然后把该行数据拷贝到 undo log 中，作为旧记录，既在 undo log 中有当前行的拷贝副本**
   * 拷贝完毕后，修改该行name为Tom，并且修改隐藏字段的事务 ID 为当前事务 1的 ID, 我们默认从 1 开始，之后递增，**回滚指针指向拷贝到 undo log 的副本记录**，既表示我的上一个版本就是它
   * 事务提交后，释放锁

   ![image-20210918132236100](https://gitee.com/jobim/blogimage/raw/master/img/20210918132236.png)



3. **又来了个事务 2修改person 表的同一个记录，将age修改为 30 岁**

   * 在事务2修改该行数据时，数据库也先为**该行加锁**
   * **然后把该行数据拷贝到 undo log 中，作为旧记录，发现该行记录已经有 undo log 了，那么最新的旧数据作为链表的表头，插在该行记录的 undo log 最前面**
   * 修改该行 age 为 30 岁，并且修改隐藏字段的事务 ID 为当前事务 2的 ID, 那就是 2 ，**回滚指针指向刚刚拷贝到 undo log 的副本记录**
   * 事务提交，释放锁


   ![image-20210918132245840](https://gitee.com/jobim/blogimage/raw/master/img/20210918132245.png)

> 从上面，我们就可以看出，不同事务或者相同事务的对同一记录的修改，会导致该记录的undo log成为一条记录版本线性表，既链表，undo log 的链首就是最新的旧记录，链尾就是最早的旧记录（当然就像之前说的该 undo log 的节点可能是会 purge 线程清除掉，向图中的第一条 insert undo log，其实在事务提交之后有可能就被删除丢失了）



### 1.2.3 Read View(读视图)

事务进行`快照读`操作的时候生产的`读视图`(Read View)，在该事务执行的快照读的那一刻，会生成数据库系统当前的一个`快照`。

记录并维护系统当前`活跃事务的ID`(没有commit，当每个事务开启时，都会被分配一个ID, 这个ID是递增的，所以越新的事务，ID值越大)，是系统中当前不应该被`本事务`看到的`其他事务id列表`。

**作用：**

* Read View主要是用来做`可见性`判断的, 即当我们`某个事务`执行`快照读`的时候，对该记录创建一个Read View读视图，把它比作条件用来判断`当前事务`能够看到`哪个版本`的数据，既可能是当前`最新`的数据，也有可能是该行记录的undo log里面的`某个版本`的数

* **主要是将`要被修改的数据`的最新记录中的 `DB_TRX_ID`（即当前事务 ID ）取出来，与系统当前其他活跃事务的 ID 去对比（由 Read View 维护），如果 DB_TRX_ID 跟 Read View 的属性做了某些比较，不符合可见性，那就通过 `DB_ROLL_PTR 回滚指针`去取出 `Undo Log 中的 DB_TRX_ID 再比较`，即遍历链表的 DB_TRX_ID（从链首到链尾，即从最近的一次修改查起），直到找到满足特定条件的 DB_TRX_ID , 那么这个 DB_TRX_ID 所在的旧记录就是当前事务能看见的最新老版本**



**Read View几个属性**

- `trx_ids`: 当前系统活跃(`未提交`)事务版本号集合。
- `low_limit_id`: 创建当前read view 时“当前系统`最大事务版本号`+1”。
- `up_limit_id`: 创建当前read view 时“系统正处于活跃事务`最小版本号`”
- `creator_trx_id`: 创建当前read view的事务版本号；



**readview（重点）**

* 开始事务时创建readview，readView维护当前活动的事务id，即未提交的事务id，排序生成一个数组访问数据。然后获取版本链数据中的事务id（），对比readview：
* 首先比较 `DB_TRX_ID< up_limit_id` （比readview都小）, 如果小于，则当前事务能看到 `creator_trx_id` 所在的记录，如果大于等于进入下一个判断
* 接下来判断 `DB_TRX_ID>= low_limit_id` , 如果大于等于则代表 `DB_TRX_ID` 所在的记录在 `Read View` 生成后才出现的，那对当前事务肯定不可见，如果小于则进入下一个判断
* 判断 `DB_TRX_ID`  是否在活跃事务之中，**如果在**，则代表我 Read View 生成时刻，你这个事务还在活跃，还没有 Commit，你修改的数据，**当前事务也是看不见的**；**如果不在**，则说明，你这个事务在 Read View 生成之前就已经 Commit 了，你修改的结果，**当前事务是能看见的。**





**读已提交（RC）和可重复读（RR）的区别就在于它们生成ReadView的策略不同**。

* 读已提交隔离级别下的事务，**在每次查询的开始都会生成一个独立的ReadView**。所以在RC级别下的事务可以看到别的事务提交的更新的原因。
* 而可重复读（RR）隔离级别下的事务，则**在第一次读的时候生成一个ReadView**，之后的读都复用之前的ReadView



# 二、MySQL主从复制原理



> 好的博客：[MySQL主从复制读写分离，看这篇就够了！](https://blog.csdn.net/alitech2017/article/details/108241782?ops_request_misc=%7B%22request%5Fid%22%3A%22163247090216780269819112%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=163247090216780269819112&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~top_positive~default-1-108241782.pc_v2_rank_blog_default&utm_term=mysql主从复制&spm=1018.2226.3001.4450)
>
> [MySQL的主从复制](https://blog.csdn.net/y_zilong/article/details/116950207)

* **主从复制、读写分离**一般是一起使用的。目的很简单，就是**为了提高数据库的并发性能（master复制读写，slave负责读）**。
* 同时，随着业务量的扩展、如果是单机部署的MySQL，会导致I/O频率过高。采用**主从复制、读写分离可以提高数据库的可用性**。



<font color='blue'>**主从复制的原理：**</font>

* **slave的`IO线程`向master请求从指定的binlog日志文件的指定位置之后的`binlog`日志内容**
* salve从库连接master主库，Master有多少个slave就会创建多少个**binlog dump线程**。
* 当Master节点进行insert、update、delete操作时，会按顺序写入到binlog中。
* 当Master节点的binlog发生变化时，**binlog dump 线程会通知所有的salve节点，向其发送二进制事件日志**。
* **`I/O线程`接收到 binlog 内容后，将内容写入到本地的 `relay-log`（中继日志文件）**。
* **`SQL线程`读取I/O线程写入的`relay-log`，并且根据 relay-log 的内容对从数据库做对应的操作**。

![image-20210924164314430](https://gitee.com/jobim/blogimage/raw/master/img/20210924164314.png)



**主从复制三个线程：**

* 主节点：dump Thread：为每个Slave的I/O Thread启动一个dump线程，用于向其发送binary log events
* 从节点：I/O Thread：向Master请求二进制日志事件，并保存于中继日志中
* 从节点：SQL Thread：从中继日志中读取日志事件，在本地完成重放
  




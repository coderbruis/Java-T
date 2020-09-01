<!-- TOC -->
- [一、InnoDB可重复读隔离级别下如何避免幻读](#一innodb可重复读隔离级别下如何避免幻读)
- [二、RR级别下的InnoDB的（快照读）非阻塞读是如何实现的？](#二rr级别下的innodb的快照读非阻塞读是如何实现的)
- [三、InnoDB如何在RR隔离界别下避免幻读——next-key锁](#三innodb如何在rr隔离界别下避免幻读next-key锁)
    - [3.1 行锁](#31-行锁)
	- [3.2 Gap锁](#32-gap锁)
- [四、总结](#四总结)
    - [4.1 InnoDB在RR隔离级别下是如何实现幻读问题的解决的呢？](#41-innodb在rr隔离级别下是如何实现幻读问题的解决的呢)
	- [4.2 InnoDB中非阻塞读（快照读）底层是怎么实现的？](#42-innodb中非阻塞读快照读底层是怎么实现的)
<!-- /TOC -->

## 一、InnoDB可重复读隔离级别下如何避免幻读

在理解什么是幻读之前，先了解下脏读、幻读、不可重复读在实操场景中的现象。

**脏读**：指的就是一个事务读取到了另一个事务还未提交的数据，当该事物将数据回滚，则读取到的就是脏数据。
**脏读造成的结果**：事务拿着脏的数据（还未提交的数据，如果回滚了）去执行业务操作，会影响业务。
**脏读解决方案**：将数据库事务隔离级别改为RC，所以事务只能读取到其他事务已经提交的数据。


**不可重复读**：指的是对于两个事务A、B，A事务进行查询操作，B事务进行更新操作，更新了数据，并且提交了B事务。此时A事务再次来查询该记录，会发现和之前查询的结果不一样了，以A事务的角度来说，A事务什么都没操作（也不知道其他事务是否有操作）就发生了数据改变，即两次读是不重复的，这就代表了不可重复读。
**不可重复读造成的结果**：同一事务中两次读取的结果不一样。
**不可重复读解决方案**：解决方案就是将事务隔离级别由RC提升为RR。如果将事务隔离级别提升为了RR，则不管B事务如何更新数据，A事务中读取的数据都是相同的，另外完全不用担心在A事务中进行操作，会造成数据不一致的问题，因为在A事务中如果进行了数据的修改操作，会使用的B事务更新之后的数据来进行修改操作。


**幻读**：（幻读的复现需要将事务隔离级别降低为RC，因为在InnoDB中RR已经解决了幻读现象）同样的对于两个事务A、B。现在数据库表中有3条记录，A事务将要将更新所有的记录，于此同时B事务新增了一条记录，并提交了事务。之后A事务执行了update操作，会发现成功操作4条记录，从A的角度来说，我什么都没做，怎么就变成了4条记录了呢？这就是幻读现象。【幻读侧重于对于记录的增加以及删除现象，而不可重复读侧重于记录本身的数据】
**幻读造成的结果**：同一事务中记录条数不同，产生幻觉。
**幻读解决方案**：InnoDB的伪MVCC机制。


**表象**：快照读（非阻塞读）--伪MVCC
**内在**：next-key锁（行锁+GAP锁也就是间隙锁）

什么是当前读呢？可以简单理解为，加了锁的增删改查就是当前读。

 - 当前读（查询）：select...lock in share mode，select...for update
   
 - 当前读（增、删、改）：update、insert、delete

不管上的是X锁（排他锁）还是S锁（共享锁）都为当前读。当前读是啥意思呢？意思是当前操作的是最新记录，其他的并发事务不能修改当前记录，对当前记录加锁。其中，select...lock in share mode是使用的S锁，select...for update、update、insert、delete都是使用的X锁。

共享锁和排他锁的区别：

 - 共享锁（s）：又称读锁。允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
 - 排他锁（Ｘ）：又称写锁。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。

使用下图理解当前读在MySQL内部运行机制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114052693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)


而快照读是什么呢？快照读就是不加锁的非阻塞读，也就是select操作。不过这里的快照读是基于非Serializable事务隔离界别下的，因为在Serializable隔离级别下，快照度会退化为当前读。这里的快照读就是MVCC的实现机制。

这里由于篇幅原因，没有贴出实验过程，这里做简单总结：

 1. 在RC隔离界别下，当前读和快照读读取的都是同一版本。 
 2. 在RR隔离界别下，当前读读到的是最新版本数据，而快照度可能读取到的是历史版本数据。

那么在RR隔离级别下，什么时候可以读到最新版本数据呢？如果在进行增、删、改完成之后，再去查询快照读，则此时读取到的是最新版本的数据。如果是在增、删、改之前进行了快照读，在增、删、改之后继续快照读，则读到的就是旧版本数据。

**总结：快照读取决于一开始快照读的时间。**

## 二、RR级别下的InnoDB的（快照读）非阻塞读是如何实现的？
底层实现离不开数据行里的DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID字段，除此之外还需要undo日志，以及read view。

原理实现就是下列几个关键内容：

 - 数据行里的DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID
- undo日志
- read view机制

说起DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID，那就要先知道MySQL一条记录是由记录的额外信息部分和记录的真实数据两部分组成。记录的额外记录部分存有变长字段长度列表、NULL值列表等，而记录的真实数据部分又由真实数据以及DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID这三个隐藏列组成。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114157496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114218704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)

比如现在有一个记录Field1、Field2、Field3数据分别为11、12、13，现在事务要修改该记录，将Field2修改为32。则这条记录首先会加载X锁，首先undo log中会拷贝一条修改前的记录，并赋值DB_ROW_ID。此时被X锁锁住的记录的DB_TRX_ID、DB_ROLL_PTR、DB_ROW_ID分别进行赋值，并且DB_ROLL_PTR的记录会指向undo log中的DB_ROW_ID的值。

如果此时又有一个事务对该记录进行了修改，则undo log日志中又会增加一条日志。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114246913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)

这样就是快照读版本的实现了。

## 三、InnoDB如何在RR隔离界别下避免幻读——next-key锁
其实，真正实现RR隔离级别下的幻读现象，是由next-key锁解决的。next-key锁又分为了（行锁 + gap锁）

### 3.1 行锁

行锁就是Record Lock，就是对单个行记录加的锁。X锁和S锁就是行锁。

### 3.2 Gap锁
Gap就是索引树中，插入新数据的间隙。间隙锁即锁定一个记录的范围，但是不锁定记录本身。间隙锁是为了避免同一事务的两次当前读出现幻读的情况。需要注意的是，Gap锁在RU、RC隔离级别下时不存在的，在RR、Serializable隔离级别下都只支持Gap锁。这就是为什么RU、RC隔离级别下无法避免幻读，RR、Serializable能够避免幻读的原因。

下面讨论的都是在RR隔离级别下出现Gap锁的场景。

 1. 在RR隔离级别下，无论删、改、查，当前读若用到主键索引或者唯一键索引，会使用Gap锁吗？
答：如果where条件全部命中，则不会用Gap锁，只会加记录锁。

怎么去理解where条件全部命中，不用加Gap锁只需要加记录锁就行了呢？这是因为比如A事务需要修改操作所有记录，此时B事务使用主键索引id进行where条件查询、删除操作，此时只需要锁住where命中的id记录即可，那么就能防止事务A出现幻读现象。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114311395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)

如图，tb中name为主键索引，id为唯一索引。某个事务使用delete from tb where id = 9进行删除操作，首先where条件全部命中，所以先会为id为9的这个记录的唯一索引加上行锁，然后会为name为d的主键索引（聚镞索引）加上排他锁。这是为了防止其他事务对where name = d进行操作，导致数据不一致的情况。

2.	在RR隔离级别下，无论删、改、查，当前读若用到主键索引或者唯一键索引，且如果where条件部分命中或者全不命中，则会加Gap锁。对于这种情况，就包含了范围查询以及精确查询非全部命中的情况。
例子1：比如现在事务A要删除一条不存在的id为7的记录，此时事务B要新增一条id为8的记录，会发现事务B一直处于等待中，这是因为精准查询全部都不命中，会对该记录范围加Gap锁。

	例子2：【tb_student中存在id为5,6,9的学生】比如在事务A中使用语句select * from tb_student where id in (5,7,9) lock in share mode;使用当前读（共享锁）来查询学生信息。在另外一个事务B中去进行新增id为6,7,8的学生，发现事务一直在等待中。这里是因为where id in (5,7,9)部分命中，所以会为(5,9]加Gap锁，锁的范围为左开右闭。因此事务B新增id为7,8的记录会被Gap锁锁住，这就是精准查询不全部命中的情况。

3. Gap锁会用在非唯一索引或者不走索引的当前读中
	**非唯一索引**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114422831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)

	比如图中某一事务A执行delete from tb1 where id = 9，因为id是非唯一索引，如果没有加Gap锁，在事务B新增一条id为9的记录时，A事务执行完delete语句后，就会发现成功删除3条记录，出现了幻觉，所以给id为9的记录加上Gap锁来防止幻读的发生。
至于Gap锁的范围，如上为：（-∞,2], (2, 6], (6, 9], (9, 11], (11, 15], (15, +∞）中的 (6, 9], (9, 11]

	**不走索引**

	对于不走索引的情况，InnoDB会为所有的Gap加锁，相当于锁表。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191025114453347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NvZGVyQnJ1aXM=,size_16,color_FFFFFF,t_70)

## 四、总结
### 4.1 InnoDB在RR隔离级别下是如何实现幻读问题的解决的呢？
 1. 表象：快照读（非阻塞读），伪MVCC
 2. 底层：next-key（行锁+Gap锁）
		**a.**	在RU、RC隔离级别下不存在Gap锁，所以在RU、RC隔离级别下无法解决幻读；在RR、Serializable隔离级别下都实现了Gap锁，所以解决了幻读现象。
		**b.** 在RR隔离级别下，如果删、改、查语句的where条件走的是主键索引或者唯一索引
				**i.** where条件全部命中，则给该记录加上记录锁。
				**ii.** where条件不全部命中，则给该记录周围加上Gap锁。
				**iii.** 加上记录锁或者是Gap锁都是为了防止RR隔离级别下发生幻读现象。
		**c.** 在RR隔离级别下，如果删、改、查语句的where条件没有走索引或者是非唯一索引或非主键索引
在当前读where条件如果没有走非唯一索引或者没有走索引，则会使用Gap锁锁住当前记录的Gap，防止幻读的发生

### 4.2 InnoDB中非阻塞读（快照读）底层是怎么实现的？
 1. 记录中存储的隐藏列DB_TRX_ID、DB_ROW_ID、DB_ROLL_ID
 2. undo日志根据上述隐藏列来进行记录数据回滚（版本回滚）
 3. review机制

参考文献：

 - 《剑指Java面试-Offer直通车》
 - 《掘金小册——MySQL是怎样运行的》


本文主要是总结MySQL中的知识点，具体知识点不做具体展开，知识点详细内容以连接的形式展开。

## 1. InnoDB存储引擎的特点？相比于其他的存储引擎有什么作用？

✅ 回答

- InnoDB会将数据分为若干个页，以页作为磁盘和内存之间交互的基本单位。InnoDB中一个页单位为16KB。InnoDB会将16KB的数据从磁盘读取到内存中，或者是以16KB的数据从内存刷新到磁盘中
- InnoDB的行格式是指记录存储在磁盘中的格式，包括了：Compact、Redundant、Dynamic和Compressed四种格式
- InnoDB支持事务机制，默认会为每一条SQL语句封装成事务，然后自动提交，所以默认情况会影响运行速度，最好将多条SQL语言放在begin和commit之间组成一个事务来执行
- InnoDB不支持全文索引
- InnoDB支持表、行（默认）级锁
- InnoDB的4大特性：插入缓冲（insert buffer）、二次写（double write）、自适应哈希索引（ahi），预读（read ahead）

## 2. InnoDB和MyISAM引擎对比

✅ 回答

- 【最大的区别】InnoDB支持事务，MyISAM不支持事务
- InnoDB支持外键，而MyISAM不支持。对一个包含外键的InnoDB表转为MyISAM会失败
- InnoDB是聚镞索引？？？？而MyISAM是非聚镞索引 
    
    底层使用的是B+树作为索引结构，数据文件和（主键）索引绑定在一起的，InnoDB中是默认会为一个没有主键的表添加上主键的，因为主键索引效率高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此主键不应过大，因为主键过大，其他索引也会变大。
    
    MyISAM是非聚集索引，也是使用B+Tree作为索引结构，索引和数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
    
    也就是说：InnoDB的B+树主键索引的叶子节点就是数据文件，辅助索引的叶子节点是主键的值；而MyISAM的B+树主键索引和辅助索引的叶子节点都是数据文件的地址指针。
 
- Innodb不支持全文索引，而MyISAM支持全文索引，在涉及全文索引领域的查询效率上MyISAM速度更快高；PS：5.7以后的InnoDB支持全文索引了
- MyISAM表格可以被压缩后进行查询操作
- InnoDB支持表、行(默认)级锁，而MyISAM支持表级锁
    
     InnoDB的行锁是实现在索引上的，而不是锁在物理行记录上。潜台词是，如果访问没有命中索引，也无法使用行锁，将要退化为表锁。
     ```
        例如：
         
            t_user(uid, uname, age, sex) innodb;
         
            uid PK
            无其他索引
            update t_user set age=10 where uid=1;             命中索引，行锁。
         
            update t_user set age=10 where uid != 1;           未命中索引，表锁。
         
            update t_user set age=10 where name='chackca';    无索引，表锁。
    ```
- InnoDB表必须有主键（用户没有指定的话会自己找或生产一个主键），而Myisam可以没有

> InnoDB和MyISAM如何选择

1. 是否要支持事务，如果要请选择innodb，如果不需要可以考虑MyISAM；
2. 如果表中绝大多数都只是读查询，可以考虑MyISAM，如果既有读也有写，请使用InnoDB。
3. 系统奔溃后，MyISAM恢复起来更困难，能否接受；
4. MySQL5.5版本开始Innodb已经成为Mysql的默认引擎(之前是MyISAM)，说明其优势是有目共睹的，如果你不知道用什么，那就用InnoDB，至少不会差。

## 3. xx

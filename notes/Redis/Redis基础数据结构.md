在Redis中，有五中基础数据结构，包括了：

- string(字符串)
- list（列表）
- hash（字典）
- set（集合）
- zset（有序集合）

## 1. 字符串String

Redis中字符串是一种"动态字符串"，意味着使用者可以修改，字符串在Redis底层中有点类似于Java中的ArrayList，用一个字符数组来存储数据。在Redis底层C语言中，对于字符串这个中数据结构，定义了一个"Simple Dynamic String"。

### 1.1 常用操作命令

在Redis中，String数据结构的常用命令为：SET、GET。

值可以是任何类型的字符串，也包括了二进制数据。

常用操作列在下列：

- set
    
    设置一个key对应的value值
    
- get

    获取一个key对应的value值

- exists

    判断一个key是否存在
    
- expire

    设置一个键过期，通常用法为：expire key_name second

- del

    删除一个key

- mset 

    批量设置

- mget

    批量获取

- setnx

    判断如果key不存在，则设置set成功，否则设置key失败。成功返回1，失败返回0。


## 2. 列表list

在Redis中，列表list相当于Java中的LinkedList链表结构。因为是链表结构，意味着list的插入和删除操作都非常快，时间复杂度为O(1)，但是查询数据则非常慢，需要遍历整个链表结构，时间复杂度为O(n)。

### 2.1 列表的常用操作

- lpush和rpush

    lpus表示的是从list的左边添加一个新的元素；rpush表示从list的右边添加一个新的元素；

- lrange

    lrange表示从list的左边一定范围取出元素；rrange表示从list的右边获得一定范围的元素；

- lindex

   lindex命令表示从list中取出指定下标的元素, 即相当于从Java链表操作中的get()方法;


## 3. 字典Hash

在Redis中，Redis的hash相当于Java中的HashMap，内部实现也差不多类似，就是通过"数组+链表"的链地址法来解决"哈希冲突"。不过实际上，Redis的哈希底层是包含两个哈希table的。

### 3.1 字典的常用操作

- hset

    像字典中添加元素；

- hget

    从字典中获取元素；

- hgetall

    从字典中获取所有元素；

- hmset

    向字典中批量设置元素；
    

## 4. 集合Set（无序）

Redis 的集合相当于 Java 语言中的 HashSet，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。

## 5. 有序列表（SortedSet）

## 6. HyperLogLog

HyperLogLog是在Redis中用来计算基数统计的一种数据结构，在输入元素的数量或者体积非常非常大时，通过HyperLogLog可以以非常小的内存来存储计算结果。 
    
# 7.（布隆过滤器）Bloom Filter

布隆过滤器用于检索一个元素是否在一个集合中。优点就是空间效率和时间效率都远远地超过一般的算法，缺点就是有一定的误识别率和删除困难。


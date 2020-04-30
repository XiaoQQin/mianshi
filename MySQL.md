<!-- TOC -->
- [1. sql的join](#1-sql的join)  
- [2. 数据库存储引擎](#2-数据库存储引擎)  
  - [2.1 InnoDB和MyISAM](#21-InnoDB和MyISAM)
<!-- /TOC -->
## 1. sql的join  
下面都是最基本的join  

- 内连接(inner join)  
  内连接查询能将左表（表 A）和右表（表 B）中能关联起来的数据连接后返回。  
  ![内连接](https://raw.githubusercontent.com/mzlogin/mzlogin.github.io/master/images/posts/database/inner-join.png)  
  ```
  SELECT A.PK AS A_PK, B.PK AS B_PK,
     A.Value AS A_Value, B.Value AS B_Value
  FROM Table_A A
  INNER JOIN Table_B B
  ON A.PK = B.PK;
  ```
- 左连接(left join)  
  左连接查询会返回左表（表 A）中所有记录，不管右表（表 B）中有没有关联的数据。在右表中找到的关联数据列也会被一起返回。  
  ![左连接](https://raw.githubusercontent.com/mzlogin/mzlogin.github.io/master/images/posts/database/left-join.png)  
  ```
  SELECT A.PK AS A_PK, B.PK AS B_PK,
       A.Value AS A_Value, B.Value AS B_Value
  FROM Table_A A
  LEFT JOIN Table_B B
  ON A.PK = B.PK;
  ```
- 右连接(right join)  
  右连接查询会返回右表（表 B）中所有记录，不管左表（表 A）中有没有关联的数据。在左表中找到的关联数据列也会被一起返回。  
  ![右连接](https://raw.githubusercontent.com/mzlogin/mzlogin.github.io/master/images/posts/database/right-join.png)  
  ```
  SELECT A.PK AS A_PK, B.PK AS B_PK,
       A.Value AS A_Value, B.Value AS B_Value
  FROM Table_A A
  RIGHT JOIN Table_B B
  ON A.PK = B.PK;
  ```
- 全连接(full outer join)  
也被称作外连接，外连接查询能返回左右表里的所有记录，其中左右表里能关联起来的记录被连接后返回。  
![全连接](https://raw.githubusercontent.com/mzlogin/mzlogin.github.io/master/images/posts/database/full-outer-join.png)  
```
SELECT A.PK AS A_PK, B.PK AS B_PK,
       A.Value AS A_Value, B.Value AS B_Value
FROM Table_A A
FULL OUTER JOIN Table_B B
ON A.PK = B.PK;
```    
  
还有一些其他join，比如 **CROSS JOIN(笛卡尔集)**,**SELF JOIN**(返回表与自己连接后符合条件的记录，一般用在表里有一个字段是用主键作为外键的情况)等。  

## 2. 数据库存储引擎  
数据库存储引擎是数据库底层软件组织，数据库管理系统（DBMS）使用数据引擎进行创建、查询、更新和删除数据。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎，还可以 获得特定的功能。现在许多不同的数据库管理系统都支持多种不同的数据引擎。MySQL的核心就是存储引擎。我们主要关注两个MySQL存储引擎。  
  
### 2.1 InnoDB和MyISAM  
MySQL在5.5版本之前的数据库存储引擎默认为MyISAM,在5.5版本之后默认的存储引擎为InnoDB。最主要的是弄清楚这两个存储引擎之间的区别。  
  
两者的对比：  
  - **是否支持行锁**： MyISAM 只有表级锁(table-level locking)，而InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁。  
  - **是否支持事务**：MyISAM不支持事务，InnoDB支持事务
  - **是否支持外键**：MyISAM不支持外键，InnoDB支持
  - **是否支持MVCC**: 仅InnoDB支持MVCC,应对高并发事务, MVCC比单纯的加锁更高效。MVCC只有在事务的**读提交**和**可重复读**两个事务级别下工作。  
    
[JavaGuide/存储引擎](https://github.com/Snailclimb/JavaGuide/blob/master/docs/database/MySQL.md#%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E)
## 3. 存储过程
一段SQL语句的预编译集合，可以理解为存储过程是存储在数据库中的一系列SQL操作的批处理。    

**优点**  
  
  - 重复使用
  - 安全性
  - 因为是预先编译的,因此具有很高的性能  
    
**缺点**  
  - 存储过程，往往定制化于特定的数据库上，因为支持的编程语言不同。当切换到其他厂商的数据库系统时，需要重写原有的存储过程。  
  - 存储过程的性能调校与撰写，受限于各种数据库系统。  
## 4. MySQL的存储结构  
因为MySQL的默认数据库存储引擎为InnoDB,所以只分析InnoDB的存储结构
### 4.1 InnoDB行记录存储结构  
我们平时以记录(可以简单的理解为数据表中的一行数据)为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为**行格式或者记录格式**。InnoDB如今有4中不同的**行格式**,分别是**Compact**、**Redundant**、**Dynamic**和**Compressed**，这四种行格式原理大致相同，只分析**Compact**。  
  
可在建表时设置``ROW_FORMAT=行格式名称``来设置表的行格式，也可使用``ALTER TABLE 表名 ROW_FORMAT=行格式名称``来修改行格式。      
InnoDB记录的格式如下所示:  
  
![compact行格式示意图.PNG](https://i.loli.net/2020/04/30/HtdLDr27o8qAfw9.png)  
  
一条完整的记录可以分为**记录的额外信息**和**记录的真实数据**。  
  
#### 记录的额外信息  
记录的额外信息是服务器为了描述这条记录不得不添加的一些信息，额外信息分为3部分:**变长字段长度列表**、**NULL值列表**和**记录头信息**。  

1.  **记录头信息**: 把所有变长类型的列的长度都存放在记录的开头部位形成一个列表，按照列的顺序**逆序存放**。变长字段长度列表中只存储值为**非NULL**的列内容占用的长度，**值为NULL 的列**的长度是不储存的。    
  
    比如表中一行记录中C1,C2,C4的长度为4,3,1 。则根据逆序存放则为 01 03 04 (十六进制)  
    
2.  **NULL值列表**: Compact把值为NULL的列统一管理起来，存储到NULL值列表中。如果表中没有允许存储 NULL 的列，则 NULL值列表 也不存在了，否则将每个允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列。  
      
     比如表中一行记录中c1,c3,c4为NULL，则NULL值列表为一个字节 0 0 0 0 0 1(c4) 1(c3) 0(c1),使用16进制为 06。      
     
  
 ![填充数据.PNG](https://i.loli.net/2020/04/30/wY1cySM8d9sPq45.png)  
   
   
3.  **记录头信息**:它是由固定的5个字节组成。5个字节也就是40个二进制位，不同的位代表不同的意思，如下所示：  
  
  
 ![记录头信息.PNG](https://i.loli.net/2020/04/30/AptUhogelIBLuYW.png)
 
 
   
    
 | 名称       | 大小(bit)         | 描述        |
|:-----------| :-------------:|:-------------:|
| 预留位1  | 1 | 没有使用 |
| 预留位2  | 1 | 没有使用 |
| deleta_mask  | 1 | 标记该记录是否被删除 |
| min_rec_mask | 1 | 标记该记录是否为B+树的非叶子节点中的最小记录 |
| n_owned  | 4 | 表示当前槽管理的记录数 |
| heap_no | 13 | 表示当前记录在记录堆的位置信息  |
| record_type |3 | 表示当前记录的类型，0表示普通记录，1表示B+树非叶节点记录，2表示最小记录，3表示最大记录 |
| next_record | 16 | 表示下一条记录的相对位置 |  
  

## 4. 数据库索引  
索引是一种用于快速查询和检索数据的数据结构，比较常见的的索引结构为:B树，B+树和Hash。数据库可以通过索引加快数据的检索，索引就好比书中的目录一样，可以快速定位数据所在的位置。  

### 4.1 索引的优缺点分析  
**索引的优点**  
  
  - 可以大大加快数据的检索速度，这是最大的优点  
  - 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。  
  - 随机IO变为顺序IO  
**索引的缺点**  
  
  - 创建索引和维护索引需要耗费时间:当对表中的数据进行增删改的时候，如果数据有索引，那么索引也需要动态的修改，会降低SQL执行效率  
  - 占用物理存储空间: 索引需要使用物理文件存储，也会耗费一定空间。  
  
### 4.2 B树和B+树的区别  
  
 - 1. B树的所有节点即存放key也存放data；而B+树只有叶子节点存放Key和data,其他非叶子节点都只存放Key  
 - 2. B树的叶子节点都是独立的，B+树的叶子节点有一条引用链指向与它相邻的叶子节点。  
 - 3. B+树的检索效率更加稳定，任何查找都是从根节点到叶子节点的过程。    
   
#### 关于Hash索引    
   
   
 **Hash索引不支持顺序查找和范围查找是它最大的缺点**。类似下列语句  
   
 ```
 SELECT * FROM tb1 WHERE id < 500;
 ```
 使用B+树索引的话,因为是有序的吗,在这种范围查询中，优势非常大，直接遍历比500小的叶子节点。而hash索引是根据hash算法来定位的，难不成还要把 1 - 499的数据，每个都进行一次hash计算来定位吗。  
 
### 4.3 索引为什么可以加快数据库的检索速度  

#### MySQL的存储结构  
  
MySQL的基本存储结构是**页**,也就是记录都存在**页**里。  
  -  数据页之间组成一个**双向链表**  
  -  数据页里面的记录又会组成一个**单向链表**,每个数据页都会为存储在页里的记录生成一个**页目录** 
       -  通过**主键**在数据页里查询记录时可以在**页目录**里进行**二分查找**快速定位到对应的槽，            然后再遍历该槽对应分组中的记录即可快速找到指定的记录  
       -  以**非主键**为搜索条件，只能从最小记录开始依次遍历单链表中的每条记录。  
       
如果以如下```select * from user where username = 'Java3y'```没有使用任何优化的SQL语句查询,那么数据库会如下进行检索数据，在数据量很大的情况下会很慢：  
  
  - 定位到记录所在的页，需要遍历双向链表，找到所在的页。  
  - 从定位所在的页内查找相应的记录，不是根据主键查询，只能遍历所在页的单链表。  
## 5. 事务

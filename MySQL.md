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
## 4. 数据索引
## 5. 事务

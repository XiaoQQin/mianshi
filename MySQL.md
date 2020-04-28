## 1. sql有几种join  
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

## 2. 数据库引擎  
## 3. 存储过程
## 4. 数据索引
## 5. 事务



# Task03：复杂查询方法-视图、子查询、函数等-天池龙珠计划SQL训练营

## [教程链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.20222307.J_9059755190.6.73dc4cb3ajrbNj&postId=167460)



## 1、视图

- 视图是基于真实表的一张**虚拟的表**，其数据来源均建立在真实表的基础上。
  - 从SQL的角度来说操作视图与操作表看起来是完全相同的

- **“视图不是表，视图是虚表，视图依赖于表”。**

  

### **为什么需要视图？**

- 通过定义视图可以**将频繁使用的SELECT语句保存以提高效率**。
- 通过定义视图可以使用户看到的数据更加清晰。
- 通过定义视图可以**不对外公开数据表全部字段，增强数据的保密性**。
- 通过定义视图可以**降低数据的冗余**。





### **注意事项**

- 在一般的DBMS中定义视图时不能使用ORDER BY语句

  - 因为视图和表一样，**数据行都是没有顺序的**。

    - **特殊：MySql**

      - > 在 MySQL中视图的定义是允许使用 ORDER BY 语句的，但是若从特定视图进行选择，而该视图使用了自己的 ORDER BY 语句，则视图定义中的 ORDER BY 将被忽略。



- **其中视图名在数据库中需要是唯一的，不能与其他视图和表重名。**



### **视图的创建**

- 1、**基于单表的视图**

  - ```mysql
    CREATE VIEW productsum (product_type, cnt_product)
    AS
    SELECT product_type, COUNT(*)
      FROM product
     GROUP BY product_type ;
    ```

- 2、**基于多表的视图**

  - ```mysql
    CREATE VIEW view_shop_product(product_type, sale_price, shop_name)
    AS
    SELECT product_type, sale_price, shop_name
      FROM product,
           shop_product
     WHERE product.product_id = shop_product.product_id;
    ```

  - 基于2个表：product 和 shop_product

### **修改视图**

- ```mysql
  ALTER VIEW productSum
      AS
          SELECT product_type, sale_price
            FROM Product
           WHERE regist_date > '2009-09-11';
  ```

- 如果原表可以更新，那么 视图中的数据也可以更新
  - 如果视图发生了改变，而原表没有进行相应更新的话，就无法保证数据的一致性了。
- 视图只是原表的一个窗口，所以**它修改也只能修改透过窗口能看到的内容。**



### 删除视图

需要有相应的权限才能成功删除

```mysql
DROP VIEW productSum;
```

---





## 2、子查询

简单来说，就是在查询得到的一个结果中，再进行查询

```mysql
-- 实例
SELECT stu_name
FROM (
         SELECT stu_name, COUNT(*) AS stu_cnt
          FROM students_info
          GROUP BY stu_age) AS studentSum;
```





注意：在子查询中像**标量子查询**，**嵌套子查询**或者**关联子查询**可以看作是**子查询的一种操作方式**即可。

----



### 子查询与视图的区别

- 子查询就是将用来定义视图的 **SELECT 语句直接用于 FROM 子句当中**。其中AS studentSum可以看作是子查询的名称，而且由于**子查询是一次性**的，所以子查询不会像视图那样保存在存储介质中， 而是在 SELECT 语句执行之后就消失了。





### 嵌套子查询

```mysql
SELECT product_type, cnt_product
FROM (SELECT *
        FROM (SELECT product_type, 
                      COUNT(*) AS cnt_product
                FROM product 
               GROUP BY product_type) AS productsum
       WHERE cnt_product = 4) AS productsum2;
```







### 标量子查询

执行的SQL语句只能返回一个值，也就是要返回表中具体的**某一行的某一列**。

**也即，返回一个具体的数值**

#### 作用

标量子查询不仅仅局限于 WHERE 子句中，通常任何可以使用单一值的位置都可以使用。也就是说， 能够使用常数或者列名的地方，无论是 SELECT 子句、GROUP BY 子句、HAVING 子句，还是 ORDER BY 子句，**几乎所有的地方都可以使用。**



- **实例**

  - ```mysql
    SELECT product_id, product_name, sale_price
      FROM product
     WHERE sale_price > (SELECT AVG(sale_price) FROM product);
    ```

  - 这就是一个**标量子查询**，我们需要注意的是这样完整的一个SQL语句称之为一个子查询，而不是SQL语句中括号内的语句才称为子查询。

  - 这个查询称之为**标量子查询**，是因为其括号内语句`SELECT AVG(sale_price) FROM product`只返回一个唯一值



### 关联子查询

**实例：**选取出各商品种类中高于该商品种类的平均销售单价的商品

```mysql
SELECT product_type, product_name, sale_price
  FROM product AS p1
 WHERE sale_price > (SELECT AVG(sale_price)
   FROM product AS p2
                      WHERE p1.product_type =p2.product_type
   GROUP BY product_type);
```

- 在第二条SQL语句也就是关联子查询中我们将外面的product表标记为p1，将内部的product设置为p2，而且通过WHERE语句连接了两个查询。
- **执行过程:**
  - 1、首先执行不带WHERE的主查询p1
  - 2、根据主查询讯结果匹配product_type，获取子查询p2结果
  - 3、将子查询结果再与主查询结合执行完整的SQL语句







---

**习题3.4**

```mysql
SELECT product_id,
 product_name,
 product_type,
 sale_price,
 -- select子句中
 (SELECT AVG(sale_price)
 FROM product AS p2
 WHERE p1.product_type = p2.product_type
 GROUP BY p1.product_type) AS avg_sale_price
 -- p2中，select子句中没有没有product_type，因此GROUP BY中不能用product_type，但是可以用p1得product_type
FROM product AS  p1;
```

---





## 3、函数

### **函数分类**

- 算术函数 （用来进行数值计算的函数）
- 字符串函数 （用来进行字符串操作的函数）
- 日期函数 （用来进行日期操作的函数）
- 转换函数 （用来转换数据类型和值的函数）
- 聚合函数 （用来进行数据聚合的函数）

#### 算数函数

- ABS – 绝对值

语法：`ABS( 数值 )`

- MOD – 求余数

语法：`MOD( 被除数，除数 )`

- ROUND – 四舍五入

语法：`ROUND( 对象数值，保留小数的位数 )`

#### 字符串函数




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

- **CONCAT – 拼接**

  语法：`CONCAT(str1, str2, str3)`

  - **如果str1，2，3中有一个为NULL，则结果为NULL**

- **LENGTH – 字符串长度**

  语法：`LENGTH( 字符串 )`

- **LOWER – 小写转换**

  LOWER 函数**只能针对英文字母使用**，它会将参数中的字符串全都转换为小写。该函数不适用于英文字母以外的场合，不影响原本就是小写的字符。

  类似的， **UPPER** 函数用于大写转换。



- **REPLACE – 字符串的替换**

  语法：`REPLACE( 对象字符串，替换前的字符串，替换后的字符串 )`

- **SUBSTRING – 字符串的截取**

  语法：`SUBSTRING （对象字符串 FROM 截取的起始位置 FOR 截取的字符数）`

  **计数从1开始**



- **（扩展内容）SUBSTRING_INDEX – 字符串按索引截取**
  - SELECT SUBSTRING_INDEX('www.mysql.com', '.', 1);  -- www
    SELECT SUBSTRING_INDEX('www.mysql.com', '.', -1); -- com
    SELECT SUBSTRING_INDEX('www.mysql.com', '.', 2); -- www.mysql
    SELECT SUBSTRING_INDEX('www.mysql.com', '.', -2); -- mysql.com



<img src="https://img.alicdn.com/imgextra/i3/O1CN01TZlSc01FJTDyUH8Ud_!!6000000000466-2-tps-801-534.png" style="zoom:80%;" />









#### 日期函数

注意，不同DBMS的日期函数可能存在不同。

- **SELECT CURRENT_DATE;**

  - CURRENT_DATE – 获取当前日期

- **SELECT CURRENT_TIME;**

  - **CURRENT_TIME – 当前时间**

- **SELECT CURRENT_TIMESTAMP;**

  - CURRENT_TIMESTAMP – 当前日期和时间

- **EXTRACT – 截取日期元素**

  语法：`EXTRACT(日期元素 FROM 日期)`

  使用 EXTRACT 函数可以截取出日期数据中的一部分，例如“年”

  “月”，或者“小时”“秒”等。该函数的返回值并不是日期类型而是**数值类型**

  - ```mysql
    SELECT CURRENT_TIMESTAMP as now, 
    EXTRACT(YEAR   FROM CURRENT_TIMESTAMP) AS year, 
    EXTRACT(MONTH  FROM CURRENT_TIMESTAMP) AS month, 
    EXTRACT(DAY    FROM CURRENT_TIMESTAMP) AS day,
    EXTRACT(HOUR   FROM CURRENT_TIMESTAMP) AS hour, 
    EXTRACT(MINUTE FROM CURRENT_TIMESTAMP) AS MINute, 
    EXTRACT(SECOND FROM CURRENT_TIMESTAMP) AS second;
    ```



#### 转换函数

“转换”这个词的含义非常广泛，在 SQL 中主要有两层意思：

一是数据类型的转换，简称为**类型转换**，在英语中称为`cast`；

另一层意思是**值的转换**。

 

- CAST – 类型转换

语法：`CAST（转换前的值 AS 想要转换的数据类型）`

```mysql
SELECT CAST('0001' AS SIGNED INTEGER) AS int_col;
```

- **COALESCE – 将NULL转换为其他值**

语法：`COALESCE(数据1，数据2，数据3……)`

COALESCE 是 SQL 特有的函数。该函数会**返回可变参数 A 中左侧开始第 1个不是NULL的值**。参数个数是可变的，因此可以根据需要无限增加。

在 SQL 语句中将 NULL 转换为其他值时就会用到转换函数。

```mysql
SELECT COALESCE(NULL, 11) AS col_1,
COALESCE(NULL, 'hello world', NULL) AS col_2,
COALESCE(NULL, NULL, '2020-11-01') AS col_3;
+-------+-------------+------------+
| col_1 | col_2       | col_3      |
+-------+-------------+------------+
|    11 | hello world | 2020-11-01 |
+-------+-------------+------------+
1 row in set (0.00 sec)
```





---







## 4、谓词

谓词就是返回值为真值的函数。包括`TRUE / FALSE / UNKNOWN`。

谓词主要有以下几个：

- **LIKE**
- **BETWEEN**
- **IS NULL、IS NOT NULL**
- **IN**
- **EXISTS**



#### LIKE谓词 – 用于字符串的部分一致查询

当需要进行字符串的部分一致查询时需要使用该谓词。

部分一致大体可以分为前方一致、中间一致和后方一致三种类型。

- 前方一致：选取出“dddabc”

  - ```my
    SELECT *
    FROM samplelike
    WHERE strcol LIKE 'ddd%';
    ```

  - 其中**的`%`是**代表“**零个或多个任意字符串**”的特殊符号，本例中代表“以ddd开头的所有字符串”。

- 中间一致：选取出“abcddd”,“dddabc”,“abdddc”

  - ```my
    SELECT *
    FROM samplelike
    WHERE strcol LIKE '%ddd%';
    ```

- 后方一致：选取出“abcddd“

  - ```my
    SELECT *
    FROM samplelike
    WHERE strcol LIKE '%ddd';
    ```

  综合如上三种类型的查询可以看出，查询条件最宽松，也就是能够取得最多记录的是**`中间一致`**。这是因为它同时包含前方一致和后方一致的查询结果。

- `_`下划线匹配任意 1 个字符

  使用 _（下划线）来代替 %，与 % 不同的是，它代表了“任意 1 个字符”。





####  BETWEEN谓词 – 用于范围查询

```mysql
-- 选取销售单价为100～ 1000元的商品
SELECT product_name, sale_price
FROM product
WHERE sale_price BETWEEN 100 AND 1000;
```

BETWEEN 的特点就是结果中会包含 100 和 1000 这两个临界值，也就是**闭区间**。如果不想让结果中包含临界值，那就必须使用 < 和 >。







####  IS NULL、 IS NOT NULL – 用于判断是否为NULL

#### IN谓词 – OR的简便用法

- 谓词 无法与 NULL 进行比较
- 
- **NOT IN 的参数中不能包含 NULL**，否则，查询结果通常为空

#### 使用子查询作为IN谓词的参数

###  EXIST 谓词





## 5、case表达式

- 不要忘记加 END





- **当待转换列为数字时，可以使用**`SUM AVG MAX MIN`**等聚合函数；**
- **当待转换列为文本时，可以使用**`MAX MIN`**等聚合函数**

---

习题3.7：

```mysql
SELECT SUM(CASE WHEN sale_price <= 1000 THEN 1 ELSE 0 END) AS
low_price,
 SUM(CASE WHEN sale_price BETWEEN 1001 AND 3000 THEN 1 ELSE 0 END) AS
mid_price,
 SUM(CASE WHEN sale_price >= 3001 THEN 1 ELSE 0 END) AS
high_price
 FROM product;
```

SUM不能换成  COUNT， 否则结果不对


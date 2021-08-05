# 阿里云天池MySql训练营

# Task02：SQL基础查询与排序-天池龙珠计划SQL训练营



## 聚合函数

在聚合函数的参数中使用DISTINCT，可以删除重复数据。

- COUNT：计算表中的记录数（行数）

  - 聚合函数会将NULL排除在外。**但COUNT(*)例外，并不会排除NULL。**
  - 想要计算值的种类时，可以在COUNT函数的参数中使用DISTINCT。

- SUM：计算表中数值列中数据的合计值

- AVG：计算表中数值列中数据的平均值

  - SUM/AVG函数只适用于数值类型的列。

- MAX：求出表中任意列中数据的最大值

- MIN：求出表中任意列中数据的最小值

  - MAX和MIN也可用于非数值型数据

  - ```mysql
    SELECT MAX(regist_date), MIN(regist_date)FROM product;
    ```



需要注意得是，其中每个只能函数一列，如：

```mysql
SELECT SUM(sale_price), SUM(purchase_price) FROM product;
```

是合法的，但是，下面的语句就是**不合法**的：

```mysql
SELECT SUM(sale_price,purchase_price) FROM product;
```



##  对表进行分组

### GROUP BY语句

```mysql
SELECT product_type, COUNT(*)
  FROM product
 GROUP BY product_type;/*按product_type 分组*/
```



- 在 GROUP BY 子句中指定的列称为 **聚合键** 或者 **分组列**。
- 如果聚合键中包含NULL，则会将NULL作为一组特殊数据进行处理。



#### **语句书写顺序**：

- GROUP BY的子句书写顺序有严格要求，不按要求会导致SQL无法正常执行，目前出现过的子句**书写顺序**为：



- **1.SELECT → 2. FROM → 3. WHERE → 4. GROUP BY**
  - 其中前三项用于筛选数据，GROUP BY对筛选出的数据进行处理





## 为聚合结果指定条件

### HAVING：用以得到特定分组

**可以理解为HABING是一个定语，用来限定分组后想要得到的数据**

- 将表使用GROUP BY分组后，怎样才能只取出其中两组？
  - WHERE子句只能指定记录（行）的条件，而不能用来指定组的条件（例如，“数据行数为 2 行”或者“平均值为 500”等）。
  - **可以在GROUP BY后使用HAVING子句。**HAVING的用法类似WHERE

HAVING 特点：

- HAVING子句用于对分组进行过滤，可以使用数字、聚合函数和GROUP BY中指定的列名（聚合键）。

  - ```mysql
    -- 错误形式（因为product_name不包含在GROUP BY聚合键中）
    SELECT product_type, COUNT(*)
      FROM product
     GROUP BY product_type
    HAVING product_name = '圆珠笔';
    ```



---

## 对查询结果进行排序

### ORDER BY

SQL中的执行结果是随机排列的，当需要按照特定顺序排序时，可已使用**ORDER BY**子句。

- 默认为升序排列，降序排列为 **DESC**

  - ```mysql
    -- 降序排列
    SELECT product_id, product_name, sale_price, purchase_price
      FROM product
     ORDER BY sale_price DESC;
    ```

- 可以参照多个基准进行排序，前一个基准相同时，按第二个基准排序

  - ```mysql
    -- 多个排序键
    SELECT product_id, product_name, sale_price, purchase_price
      FROM product
     ORDER BY sale_price, product_id;
    ```

- ```mysql
  -- 当用于排序的列名中含有NULL时，NULL会在开头或末尾进行汇总。
  SELECT product_id, product_name, sale_price, purchase_price
    FROM product
   ORDER BY purchase_price;
  ```



**注意：**

- GROUP BY 子句中不能使用SELECT 子句中定义的别名，但是在 ORDER BY 子句中却可以使用别名。



- **SQL在使用 HAVING 子句时 SELECT 语句的执行顺序为：**
  - **FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY**

其中SELECT的执行顺序在 GROUP BY 子句之后，ORDER BY 子句之前。也就是说，当在ORDER BY中使用别名时，已经知道了SELECT设置的别名存在，但是在GROUP BY中使用别名时还不知道别名的存在，所以**在ORDER BY**中可以使用别名，但是在**GROUP BY**中不能使用别名





---



#### 常见错误

- **1、在聚合函数的SELECT子句中写了聚合健以外的列**
  - 使用COUNT等聚合函数时，*SELECT子句中如果出现列名，只能是GROUP BY子句中指定的列名（也就是聚合键）。*
- **2、在GROUP BY子句中使用列的 别名**
  - SELECT子句中可以通过AS来指定别名，但在GROUP BY中不能使用别名。因为*在DBMS中 ,SELECT子句在GROUP BY子句后执行。*
- **3、在WHERE中使用聚合函数**
  - 原因是***聚合函数的使用前提是结果集已经确定***，而WHERE还处于确定结果集的过程中，所以相互矛盾会引发错误。 如果想指定条件，可以在SELECT，HAVING（下面马上会讲）以及ORDER BY子句中使用聚合函数。



---

练习题6：

```mysql
SELECT `product_type`, SUM(sale_price), SUM(purchase_price) FROM `product` GROUP BY product_type HAVING SUM(sale_price)>1.5*SUM(purchase_price);
```

练习题7：

```mysql
 SELECT * FROM `product` ORDER BY `regist_date` DESC , `sale_price`;

```


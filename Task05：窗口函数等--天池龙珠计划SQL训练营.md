# Task05：窗口函数等--天池龙珠计划SQL训练营



## [原文链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.20222307.J_9059755190.2.73dc4cb3qvp6Oc&postId=170315)

- ## 教程中本节内容主要来源:

## [《SQL基础教程》](https://m.ituring.com.cn/book/1880)





## 窗口函数

窗口函数也称为**OLAP**函数,OLAP 是OnLine AnalyticalProcessing 的简称，意思是对数据库数据进行实时分析处理。

- 常规的SELECT语句都是对整张表进行查询，而窗口函数可以让我们**有选择的去某一部分数据进行汇总、计算和排序。**

  窗口函数的通用形式：

  ```mysql
  <窗口函数> OVER ([PARTITION BY <列名>]
                       ORDER BY <排序用列名>)  
  ```

  - **PARTITON BY**是用来分组，即选择要看哪个窗口，类似于GROUP BY 子句的分组功能，但是PARTITION BY 子句并不具备GROUP BY 子句的汇总功能，并**不会改变原始表中记录的行数**。

    - #### 如果不加 PARTITON BY 关键字，则整张表就是一个**窗口**

      - #### 注意，在MySql是本行以及之前的行 为一个窗口

  - **ORDER BY**是用来排序，即决定窗口内，是按哪种规则(字段)来排序的。

---



- 实例：

  - ```mysql
    SELECT product_name
           ,product_type
           ,sale_price
           ,RANK() OVER (PARTITION BY product_type
                             ORDER BY sale_price) AS ranking
      FROM product  
    ```

  - 本例中，就是 按product_type分组，然后在分组内，按sale_price排序。注意，这里按product_type分组并不会把某个分组的合并会一行。

  - 结果：

    - <img src="https://img.alicdn.com/imgextra/i4/O1CN01hMPJM61V3TF4JmnmT_!!6000000002597-2-tps-800-317.png" style="zoom: 80%;" />



### 窗口函数的使用位置

**一般只能在select子句中使用**

- 语法上，除了SELECT子句，ORDER BY子句或者UPDATE语句的SET子句中也可以使用。但因为几乎没有实际的业务示例，所以开始的时候大家只要记得“只能在SELECT子句中使用”就可以了。

**Why？**

- 在DBMS 内部，窗口函数是对WHERE 子句或者GROUPBY 子句处理后的“结果”进行的操作

---



### 窗口函数分类

大体可以分为2类:

- 一是 将**SUM、MAX、MIN**等**聚合函数**用在窗口函数中

- 二是 **RANK、DENSE_RANK**等排序用的**专用窗口函数**



#### 专用窗口函数：

- **RANK**
  - 有 3 条记录排在第 1 位时：1 位、1 位、1 位、4 位……

- **DENSE_RANK**

  - 有 3 条记录排在第 1 位时：1 位、1 位、1 位、2 位……

- **ROW_NUMBER**

  - 有 3 条记录排在第 1 位时：1 位、2 位、3 位、4 位

- 实例：

  - ```mysql
    SELECT  product_name
           ,product_type
           ,sale_price
           ,RANK() OVER (ORDER BY sale_price) AS ranking
           ,DENSE_RANK() OVER (ORDER BY sale_price) AS dense_ranking
           ,ROW_NUMBER() OVER (ORDER BY sale_price) AS row_num
      FROM product  
    ```

  - <img src="https://img.alicdn.com/imgextra/i1/O1CN01z73HeN1f4yhiQefvw_!!6000000003954-2-tps-655-252.png"  />



---



### 聚合函数在窗口函数中的应用

聚合函数在窗口函数使用时，计算的是**累积到当前行的所有的数据的聚合。**

所有的聚合函数都能用作窗口函数，其语法和专用窗口函数完全相同。



实例：

- ```mysql
  SELECT product_id, product_name, sale_price,
  　SUM (sale_price) OVER (ORDER BY product_id) AS current_sum
  FROM Product;
  ```

  - 结果：**计算出商品编号“小于自己”的商品的销售单价的合计值。**
  - ![](https://img.alicdn.com/imgextra/i1/O1CN01kuRiqt1r7ygelUfjJ_!!6000000005585-2-tps-961-375.png)

---

### 计算移动平均

可以对包含在窗口内的行进行更加详细的汇总范围指定：汇总范围称为 **框架**。

- #### 实例

  - ```mysql
    -- 代码清单8-6　指定“最靠近的3行”作为汇总对象
    SELECT product_id, product_name, sale_price,
    	AVG (sale_price) OVER (ORDER BY product_id
    							ROWS 2 PRECEDING) AS moving_avg
    FROM Product;
    ```

  - ![SQL代码清单8-6](C:\Users\NickShan\iCloudDrive\3L68KQB4HG~com~readdle~CommonDocuments\100-工作\160-天池\天池训练营\MySql训练营\新建文件夹\SQL代码清单8-6.png)

  - #### `ROWS 2 PRECEDING` 前2行以及此行，或：往上最靠近的3行

    - 这里我们使用了**ROWS**（“行”）和**PRECEDING**（“之前”）两个关键字，将框架指定为“截止到之前~ 行”，因此“ROWS 2 PRECEDING”就是将框架指定为“截止到之前2 行”，也就是将作为汇总对象的记录限定为如下的“最靠近的3 行”。

  - ##### 如果将条件中的数字变为“ROWS 5 PRECEDING”，就是“截止到之前5 行”（最靠近的6 行）的意思

- ## 这样的统计方法称为移动平均（moving average）



**类似的：**

- 将框架指定为截止到当前记录之后2行（最靠近的3行）： 

  - #### ROWS 2 FOLLOWING

- 如果希望将当前记录的前后行作为汇总对象时，可以同时使用PRECEDING（“之前”）和FOLLOWING（“之后”）关键字来实现。

  - ```mysql
    SELECT product_id, product_name, sale_price,
    AVG (sale_price) OVER (ORDER BY product_id
    						ROWS BETWEEN 1 PRECEDING AND 
    						1 FOLLOWING) AS moving_avg
    FROM Product;
    ```

    - 将“1 PRECEDING”（之前1 行）和“1 FOLLOWING”（之后1 行）的区间作为汇总对象

      ##### ● 之前1行的记录

      ##### ● 自身（当前记录）

      ##### ● 之后1行的记录

- ## 注意：

  - ##### OVER 子句中的ORDER BY 只是用来决定窗口函数按照什么样的顺序进行计算的，对结果的排列顺序并没有影响。

  - 如果想让记录确实按照 窗口函数中的 ORDER BY子句的顺序排序，则需要在主查询语句后面对结果排序：

    - ```mysql
      SELECT product_name, product_type, sale_price,
      RANK () OVER (ORDER BY sale_price) AS ranking
      FROM Product
      ORDER BY ranking;
      ```

    - 最后加了 `ORDER BY ranking;`



## GROUPING运算符



- ### GROUPING 运算符包含以下3 种

  - ##### ROLLUP

  - ##### CUBE

  - ##### GROUPING SETS

- ROLLUP

  - 可以同时计算出：小计行 & 合计行

  - 实例:

    - ```mysql
      SELECT product_type, regist_date, SUM(sale_price) AS sum_price
      FROM Product
      GROUP BY product_type, regist_date with ROLLUP;
      ```

    - ![](https://img.alicdn.com/imgextra/i1/O1CN01a2wvRr20h5aecQ6Zq_!!6000000006880-2-tps-416-219.png)





---

- ### 更详细内容建议参考文章开头所列书籍



## 习题解答：

5.2 

继续使用product表，计算出按照登记日期（regist_date）升序进行排列的各日期的销售单价（sale_price）的总额。排序是需要将登记日期为NULL 的“运动 T 恤”记录排在第 1 位（也就是将其看作比其他日期都早）

答：

```mysql
-- 不对null进行排列在第一行的处理的答案：
SELECT product_type,regist_date,SUM(sale_price) AS sum_price 
FROM product 
GROUP BY regist_date,product_type WITH ROLLUP ;
```


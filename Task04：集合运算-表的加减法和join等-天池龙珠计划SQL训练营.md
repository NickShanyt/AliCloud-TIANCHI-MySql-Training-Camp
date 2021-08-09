# Task04：集合运算-表的加减法和join等-天池龙珠计划SQL训练营

## [教程链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.20222307.J_9059755190.29.73dc4cb3Ebhijf&postId=167461)



**COMMENTS**:

这部分内容对于未接触增进没有接触过SQL语言的读者来说可能有点冗长和费劲，一方面语句的理解有些困难，另一方面本章文字内容非常多，非常容易让读者放弃。

当然了，文中的例子质量非常高，想必是作者在实际工作中遇到的很多情况的影子。

---



## UNION

**UNION** 等集合运算符通常都会除去重复的记录.

**UNION ALL** 进行不去重地合并. 



#### MySQL 8.0 还不支持 EXCEPT 运算，不支持交运算INTERSECT

差集 EXCEPT

- 集合A和B做减法只是将集合A中也同时属于集合B的元素减掉。
- 可以使用 NOT IN 来实现 差集的功能

交集 INTERSECT





其余部分请点击链接查看原教程。

---





# 联结查询

- ## 内联结(INNER JOIN)

- ## 外连结(OUTER JOIN)

- ## 自连结(SELF JOIN)

  - 自连结并不是区分于内连结和外连结的第三种连结, **自连结可以是外连结也可以是内连结**, 它是不同于内连结外连结的另一个连结的分类方法.

- ## 自然连结(NATURAL JOIN)

  - 内联结的一种特例：当两个表进行自然连结时, **会按照两个表中都包含的列名来进行等值内连结, 此时无需使用 ON 来指定连接条件.**

    ```sql
    SELECT *  FROM shopproduct NATURAL JOIN product
    ```

  - 在进行自然连结时, 实际上是在**逐字段进行等值连结,** 两个缺失值用等号进行比较, 结果不为真. 而**连结只会返回对连结条件返回为真的那些行.**

## 内联结(INNER JOIN)

**内联结，也称为`等值联结`**

```mysql
FROM <tb_1> INNER JOIN <tb_2> ON <condition(s)>
```

示例:

```mysql
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.product_type
       ,P.sale_price
       ,SP.quantity
  FROM shopproduct AS SP
 INNER JOIN product AS P
    ON SP.product_id = P.product_id;
    
    //通过 ON后面的条件：SP.product_id = P.product_id 来连结两张表：shopproduct 和 product
```





实际上，下面两份代码的效果是等效的，使用where语句相当于限定了ON的条件：

- ANSI SQL 规范首选INNER JOIN 语法

```mysql
SELECT vend_name, prod_name, prod_price
FROM Vendors INNER JOIN Products
ON Vendors.vend_id = Products.vend_id;
```

```mysql
SELECT vend_name, prod_name, prod_price
FROM Vendors, Products
WHERE Vendors.vend_id = Products.vend_id;
```





### 要点：

- 1、**进行连结时需要在 FROM 子句中使用多张表.**
  - FROM shopproduct AS SP INNER JOIN product AS P
    - 用INNER JOIN 连结两张表
- 2、**必须使用 ON 子句来指定连结条件.**
  - ON 子句是专门用来指定连结条件的, 
- 3、**SELECT 子句中的列最好按照 表名，列名 的格式来使用.**
  - 





在结合 WHERE 子句使用内连结的时候, 我们也可以更改任务顺序, 并采用任务分解的方法,**先分别在两个表使用 WHERE 进行筛选,然后把上述两个子查询连结起来.**

```sql
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.product_type
       ,P.sale_price
       ,SP.quantity
  FROM (-- 子查询 1:从shopproduct 表筛选出东京商店的信息
        SELECT *
          FROM shopproduct
         WHERE shop_name = '东京' ) AS SP
 INNER JOIN -- 子查询 2:从 product 表筛选出衣服类商品的信息
   (SELECT *
      FROMproduct
     WHERE product_type = '衣服') AS P
    ON SP.product_id = P.product_id;
```

先分别在两张表里做筛选, 把复杂的筛选条件按表分拆, 然后把筛选结果(作为表)连接起来, 避免了写复杂的筛选条件, 因此这种看似复杂的写法, 实际上整体的逻辑反而非常清晰. 

在写查询的过程中, 首先要按照最便于自己理解的方式来写, 先把问题解决了, 再思考优化的问题.

---



## 不使用过时的SQL语句的原因：

- 1、使用**过时的**语法无法马上判断出到底是内连结还是外连结（又或者是其他种类的连结）.

- 2、由于连结条件都写在 WHERE 子句之中,因此无法在短时间内分辨出哪部分是连结条件,哪部分是用来选取记录的限制条件.

- 3、我们不知道这样的语法到底还能使用多久.每个 DBMS 的开发者都会考虑放弃过时的语法,转而支持新的语法.虽然并不是马上就不能使用了,但那一天总会到来的.

  

  虽然这么说,但是现在使用这些过时语法编写的程序还有很多,到目前为止还都能正常执行.我想大家很可能会碰到这样的代码,因此还是希望大家能够了解这些知识.





---

### 练习题

4.1 找出 product 和 product2 中售价高于 500 的商品的基本信息。

- ```mysql
  (SELECT * FROM `product2` WHERE `sale_price` >500  ORDER BY `product_id`) 
  UNION 
  (SELECT * FROM `product` WHERE `sale_price` >500 ORDER BY `product_id`) 
  ORDER BY `product_id` ;
  ```



4.2 借助对称差的实现方式, 求product和product2的交集。

```mysql
-- 1、对称差：
select * from product
where product_id not in (select product_id from product2)
union
select * from product2
where product_id not in (select product_id from product)
-- 2、交集  = 并集 - 对称差集
-- 并集：
select * from product union select * from product2

-- 交集：
select * from
(
select * from product union select * from product2
) as u
where product_id not in
(
select * from product
where product_id not in (select product_id from product2)
union
select * from product2
where product_id not in (select product_id from product)
)
```







4.3 每类商品中售价最高的商品都在哪些商店有售 ？

```mysql
-- 1、查询每类商品中售价最高的
-- 这里在SELECT 中只要一列，因为这个是作为子查询要用的，用于 in  关键字，故而只能保留一列
SELECT  MAX(`sale_price`) AS `max_price` FROM `product` GROUP BY `product_type`

-- 2、找出这些最高售价对应的商品信息
SELECT `product_id`, `sale_price`, `product_type`   FROM `product`AS A WHERE `sale_price` in
(
SELECT MAX(`sale_price`) AS `max_price` FROM `product` AS B WHERE  A.`product_type` =B.`product_type` GROUP BY `product_type`
    )
    -- 这里子查询中WHERE  A.`product_type` =B.`product_type` 如果放在主查询，则会提示并没有B这个表，因为B这个表是在子查询中定义的
    
-- 3、通过product_id 匹配 shop_id
SELECT `shop_id`
FROM `shopproduct` AS sp
WHERE  sp.product_id IN  
(
    SELECT `product_id` FROM `product`AS A WHERE `sale_price` in
(
SELECT MAX(`sale_price`) AS `max_price` FROM `product` AS B WHERE  A.`product_type` =B.`product_type` GROUP BY `product_type`
    )
    );
```



4.4 分别使用内连结和关联子查询每一类商品中售价最高的商品。

```mysql
-- 1、关联子查询
-- 1.1、查询每类商品中售价最高的
-- 这里在SELECT 中只要一列，因为这个是作为子查询要用的，用于 in  关键字，故而只能保留一列
SELECT  MAX(`sale_price`) AS `max_price` FROM `product` GROUP BY `product_type`

-- 1.2、找出这些最高售价对应的商品信息
SELECT `product_id`, `sale_price`, `product_type`   FROM `product`AS A WHERE `sale_price` in
(
SELECT MAX(`sale_price`) AS `max_price` FROM `product` AS B WHERE  A.`product_type` =B.`product_type` GROUP BY `product_type`
    )
    
-- 2、内联结

select a.product_id , a.product_type,a.product_name,a.sale_price
from product2 a
inner join
(
select product_type, max(sale_price) sale_price
from product2
group by product_type) b
on a.product_type = b.product_type and a.sale_price = b.sale_price;
-- 注意，在此内联结中，表b是一个查询出来的虚拟表，并且表b的列并未作为最终select查询的列显示，而是作为select查询的筛选条件
```





4.5 用关联子查询实现：在`product`表中，取出 product_id, produc_name, slae_price, 并按照商品的售价从低到高进行排序、对售价进行累计求和。

```mysql
-- 1、按sale_price排序
-- 不含子关联查询的情况
select product_id, product_type, product_name, sale_price
from product p1
ORDER BY sale_price;

-- 2、带子关联查询， 但不处理 sale_price 相同的情况
select product_id, product_type, product_name, sale_price,
(select sum(sale_price)from product p2 where p1.sale_price >= p2.sale_price) 累计求和
from product2 p1
ORDER BY sale_price;

-- 注意子查询中语句的理解，子查询中的 where限定条件，是限定子查询的条件，p1.sale_price >= p2.sale_price中，是用来限定p2.sale_price，限定它干什么呢？ 我们看select中的语句，是要计算sum(sale_price)，也就是说，where语句限制了子查询，在每次计算sum(sale_price)时，sale_price的选取，要选取 <= p1.sale_price的部分
select sum(sale_price) from product2 p2 where p1.sale_price >= p2.sale_price
-- 同时请注意当sale_price有相同情况时，此语句会处理错误，将价格小于等于其的都加起来，得不到我们想要的效果。

-- 3、带子关联查询， 处理 sale_price 相同的情况
select product_id, product_type, product_name, sale_price,
(select sum(sale_price)from product p2
where p1.sale_price > p2.sale_price OR (p1.sale_price = p2.sale_price and p1.product_id >= p2.product_id)) 
from product2 p1
ORDER BY sale_price;

-- 因为当sale_price相同时，其计算累加会出问题，因此对相同售价的情况，做了处理：p1.product_id >= p2.product_id，因为ID是唯一的，因此这样再次判断了可以保证其只累加了小于其自己的。注意，这里可以这样做和ORDER BY sale_price是有关系的
```




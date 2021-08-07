# Task04：集合运算-表的加减法和join等-天池龙珠计划SQL训练营

## [教程链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.20222307.J_9059755190.29.73dc4cb3Ebhijf&postId=167461)



## 要点、易错点记录





UNION 等集合运算符通常都会除去重复的记录.

UNION ALL 进行不去重地合并. 





#### MySQL 8.0 还不支持 EXCEPT 运算，不支持交运算INTERSECT

差集 EXCEPT

- 集合A和B做减法只是将集合A中也同时属于集合B的元素减掉。
- 可以使用 NOT IN 来实现 差集的功能






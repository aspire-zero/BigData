## MySQL的插入、修改删除
### 插入操作
方式一：经典的插入
```mysql
insert into table_name(col, ...) value(val, ...)
```
注意点：  
* 插入的值的类型必须和列的类型一致或者兼容
* 不允许为null的必须插入值
* 列的顺序可以调换
* 列的数量和值的数量一致
* 可以省略列名，但值必须顺序一致  

方式二：
```mysql
insert into table_name 
set col_name=val, cal_name1=val1...
```
两个方式对比：  
* 方式一可以一次插入多条数据，方式二不行
* 方式一可以插入子查询结果，方式二不行
### 修改操作
语法：
```mysql
update table_name set col=val 
where condition
# 修改多表
update 表1 别名,表2 别名
set 列=值,...
where 连接条件
and 筛选条件;
# 修改多表2
update 表1 别名
inner/left/right join 表2 别名
on 连接条件
set 列=值,...
where 筛选条件
```
案例: 修改没有男朋友的女神的男朋友都为2号
```mysql
update boys bo 
right join beauty b on bo.id = b.boyfriend_id
set b.boyfriend_id = 2
where b.id is null
```
### 删除操作
方式一：delete
语法:
1.单表的删除★★★
delete from 表名 where 筛选条件
2.多表的删除[补充]
语法:
delete 表1/表2的别名 from 表1 别名,表2 别名 where 连接条件 and 筛选条件;
案例：删除张无忌女朋友的信息
```mysql
delete b
from beauty b
inner join boys bo on b.boyfriend_id=bo.id
where bo.name='zhangwuji;
```
语法:
delete 表1/表2的别名 from 表1 别名
inner/left/right join 表2 别名
on 连接条件
where 筛选条件;
案例：删除张无忌女朋友的信息以及张无忌的信息
```mysql
delete b, bo
from beauty b
inner join boys bo on b.boyfriend_id=bo.id
where bo.name='zhangwuji;
```
方式二:truncate 或者drop  
语法:truncate table 表名;  
drop table 表名;
```mysql
truncate table boys;
```
两者比较  
1.truncate删除,效率高一丢丢, 不允许加where  
2.假如要删除的表中有自增长列,如果用delete删除后,再插入数据,自增长的值从断点开始,
而truncate删除后,再插入数据,自增长列的值从1开始.  
3.truncate删除没有返回值,delete删除有返回值  
4.truncate删除不能回滚,delete删除可以回滚
5.truncate会清空表中的所有行，但表结构及其约束、索引等保持不变；drop会删除表的结构及其所依赖的约束、索引等。  
可以简单理解为：一本书，delete是把目录撕了，truncate是把书的内容撕下来烧了，drop是把书烧了
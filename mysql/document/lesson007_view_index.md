### [MYSQL 视图 索引](#top) <b id="top"></b>

> `MYSQL 非常常用 是一张虚拟的表，一个有多张表组成的一张表,本质上存储的sql语句 用于也已对视图进行增删改查 建立索引的目的:加快查询速度`

-----
- [x] [`1.创建一个简单的视图`](#aim1)
- [x] [`2.修改视图`](#aim2)
- [x] [`3.删除视图`](#aim3)
- [x] [`4.建立索引`](#aim4)
- [x] [`5.设计MySql索引的时候有一下几点注意`](#aim5)


----
##### [1.创建一个简单的视图](#top) <b id="aim1"></b>
`视图有两个优点，第一简单化，第二安全性`

`一个简单的视图`
```sql
create view my_view_name as 
select A.Name as Employee from Employee as A 
where 
A.Salary > (select Salary from  Employee where Id = A.ManagerId);
```

`查看视图的结构`
```sql
describe my_view_name;
-- or
desc my_view_name;
```
`视图详细信息`
```sql
show create view my_view_name;
```
##### [2.修改视图](#top) <b id="aim2"></b>
```sql
create or replace view my_view_name
as
sql.....
```
`使用alter语句`

```sql
alter view my_view_name 
as 
sql ...
```

##### [3.删除视图](#top) <b id="aim3"></b>
```sql
drop view [if Exists] my_view_name;
```
##### [4.建立索引](#top) <b id="aim4"></b> 

`索引分类`
- `普通索引和唯一索引`:`普通索引是MYSQL 中的基本索引类型，允许重复值和空值，唯一索引，索引列必须唯一，但允许有空值，如果是组合索引，则列值的组合必须唯一，主键索引是一种特殊的唯一索引，不允许有空值`
- `单列索引和组合索引`:`顾名思义，不做解释。就是用几个列来做索引`

- `全文索引`:`新版的MySQL5.6.24上InnoDB引擎也加入了全文索引 MYISAM 早就支持 只有字段的数据类型为  char、varchar、text 及其系列才可以建全文索引` 
- `空间索引`:`MySQL在5.7之后的版本支持了空间索引，而且支持OpenGIS几何数据模型`

##### 创建索引
```sql
CREATE INDEX index_name ON table_name (column_list);

CREATE TABLE article ( 
    id INT AUTO_INCREMENT NOT NULL PRIMARY KEY, 
    title VARCHAR(200),
    INDEX(title)
) TYPE=MYISAM; 
```
`唯一索引`:`PRIMARY KEY索引仅是一个具有名称PRIMARY的UNIQUE索引`
```sql
CREATE UNIQUE INDEX index_name ON table_name (column_list);
```
##### 删除索引
```sql
DROP INDEX index_name ON talbe_name

ALTER TABLE table_name DROP INDEX index_name

ALTER TABLE table_name DROP PRIMARY KEY
```

##### 查看索引
```sql
show index from tblname;

show keys from tblname;
```
##### 全文索引
```sql
--创建表的同时创建全文索引
CREATE TABLE article ( 
    id INT AUTO_INCREMENT NOT NULL PRIMARY KEY, 
    title VARCHAR(200), 
    body TEXT, 
    FULLTEXT(title, body) 
) TYPE=MYISAM; 

-- 通过 alter table 的方式来添加
ALTER TABLE `student` ADD FULLTEXT INDEX ft_stu_name  (`name`) #ft_stu_name是索引名，可以随便起
--或者：
ALTER TABLE `student` ADD FULLTEXT ft_stu_name  (`name`)
--- 直接通过create index的方式
CREATE FULLTEXT INDEX ft_email_name ON `student` (`name`)
-- 也可以在创建索引的时候指定索引的长度：
CREATE FULLTEXT INDEX ft_email_name ON `student` (`name`(20))
-- 删除全文索引
 DROP INDEX full_idx_name ON tommy.girl ;
--or
 ALTER TABLE tommy.girl DROP INDEX ft_email_abcd;
```
##### [5.设计MySql索引的时候有一下几点注意](#top) <b id="aim5"></b> 

* `1，创建索引`:`对于查询占主要的应用来说，索引显得尤为重要。很多时候性能问题很简单的就是因为我们忘了添加索引而造成的，或者说没有添加更为有效的索引导致。如果不加索引的话，那么查找任何哪怕只是一条特定的数据都会进行一次全表扫描，如果一张表的数据量很大而符合条件的结果又很少，那么不加索引会引起致命的性能下降。但是也不是什么情况都非得建索引不可，比如性别可能就只有两个值，建索引不仅没什么优势，还会影响到更新速度，这被称为过度索引。`

* `2，复合索引`:`比如有一条语句是这样的：select * from users where area=’beijing’ and age=22;如果我们是在area和age上分别创建单个索引的话，由于mysql查询每次只能使用一个索引，所以虽然这样已经相对不做索引时全表扫描提高了很多效率，但是如果在area、age两列上创建复合索引的话将带来更高的效率。如果我们创建了(area, age,salary)的复合索引，那么其实相当于创建了(area,age,salary)、(area,age)、(area)三个索引，这被称为最佳左前缀
特性。因此我们在创建复合索引时应该将最常用作限制条件的列放在最左边，依次递减。`

* `3，索引不会包含有NULL值的列只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为NULL。`

* `4，使用短索引对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR(255)的 列，如果在前10 个或20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。`

* `5，排序的索引问题mysql查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。`

* `6，like语句操作一般情况下不鼓励使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引而like “aaa%”可以使用索引。`

* `7，不要在列上进行运算 select * from users where YEAR(adddate)`

* `8，不使用NOT IN和操作NOT IN和操作都不会使用索引将进行全表扫描。NOT IN可以NOT EXISTS代替，id3则可使用id>3 or id`

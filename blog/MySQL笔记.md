# 常见命令

|              SELECT DISTINCT id FROM products;               | 显示唯一的行                                                 |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
|                 show create database python;                 | 查看指定数据库                                               |
|                         use python;                          | 使用数据库                                                   |
|                      select database();                      | 查看当前数据库                                               |
|              SELECT id FROM products LIMIT 5,5;              | 第一个数为开始位置，第二个数为检索行数                       |
| SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 1; | 找出一列中最大值                                             |
| SELECT pro_price FROM products WHERE prod_price BETWEEN 5 AND 10; | 范围之间用BETWEEN                                            |
|   SELECT cust_id FROM customers WHERE cust_email IS NULL;    | 空值判定用IS NULL                                            |
| SELECT pro_price FROM products WHERE(id = 1002 OR id = 1003) AND pro_price >= 10; | 注意：无括号时AND优先级高于OR                                |
| SELECT price FROM products WHERE id IN (1002,1003) ORDER BY prod_name; | IN操作符判断存在；长清单时使用，执行速度快                   |
| SELECT price FROM products WHERE id NOT IN (1002,1003) ORDER BY prod_name; | NOT取反                                                      |
| select concat(last_name," ",first_name) as Name from employees; | 拼接字符串                                                   |
|             Insert into 表名  values(具体数据值)             | 增加一行数据                                                 |
|         Insert into 表名(字段名) values(具体数据值)          | 增加部分数据                                                 |
|     Insert into 表名  values (具体数据值), (具体数据值);     | 增加多行数据                                                 |
|                Delete from 表名  where 条件;                 | 删除数据                                                     |
|                      Delete from 表名;                       | 删除所有数据，但是不重置主键的计数(清空）                    |
|                     Truncate table 表名;                     | 删除表所有数据，重置主键的计数（截断）                       |
|                       Drop table 表名;                       | 删除表（字段和数据均不再存在）                               |
|             Update 表名 set  age=1 where 条件；              | 修改数据                                                     |
|                      ON DELETE RESTRICT                      | 删除主表数据时，如果有关联记录，不删除；                     |
|                      ON UPDATE CASCADE                       | 更新主表时，如果字表有关联记录，更新字表记录；               |
| insert ignore into actor values(``"3"``,``"ED"``,``"CHASE"``,``"2006-02-15 12:34:33"``); | insert ignore表示，如果中已经存在相同的记录，则忽略当前新数据； |
| create table actor (<br/>    actor_id smallint(5) not null PRIMARY KEY,<br/>    first_name varchar(45) not null,<br/>    last_name varchar(45) not null,<br/>    last_update date not null); | 创建数据表时，表名和字段名不需要用引号括起来                 |
| **create TABLE if not exists actor_name**(<br/>**    first_name varchar(45) not null,<br/>    last_name varchar(45) not null); | 建表语句                                                     |
| CREATE UNIQUE INDEX uniq_idx_firstname ON actor(first_name);<br />ALTER TABLE actor ADD UNIQUE INDEX  uniq_idx_firstname(first_name); | 增加索引                                                     |





# MySQL刷题记录

第10题第二种思路

第11题



42题 删除emp_no重复的记录

错误方法）

```
DELETE FROM titles_test``WHERE id NOT IN(SELECT MIN(id)`FROM titles_test`GROUP BY emp_no);
```

MySQL中不允许在子查询的同时删除表数据（不能一边查一边把查的表删了）

正确方法）

```
DELETE FROM titles_test``WHERE id NOT IN(`SELECT * FROM(SELECT MIN(id)`FROM titles_test`GROUP BY emp_no)a); 
```

第11题

# MySQL基础命令

## 索引

索引：帮助MySQL高效获取数据的数据结构

![image-20211223103518331](C:\Users\DESKTOP\AppData\Roaming\Typora\typora-user-images\image-20211223103518331.png)

BTREE结构

```
BTree又叫多路平衡搜索树，一颗m叉的BTree特性如下：

​	树中每个节点最多包含m个孩子

​	除根节点与叶子结点外，每个节点至少有[ceil(m/2)]个孩子

​	若根节点不是叶子结点，则至少有两个孩子

​	所有的叶子结点都在同一层

​	每个非叶子结点由n个key与n+1个指针组成，其中[ceil(m/2)-1]<= n <= m-1
```

MySQL索引数据结构对经典的B+Tree进行了优化，增加了一个指向相邻叶子结点的链表指针，**便于范围搜索**

### 索引分类

1）单值索引：一个索引只包含单个列，一个表可以有多个单列索引

2）唯一索引：索引列的值必须唯一，但允许有空值

3）复合索引：一个索引包含多个列

### 索引语法

#### 创建索引

create [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name [USING index_type] ON tbl_name(index_col_name,…);

#### 查看索引

show index from tal_name (\G);

#### 删除索引

drop index index_name ON tal_name;

#### ALTER命令

```sql
1）alter table tbl_name add primary key(column_list);
	添加主键，索引值必须唯一
2) alter table tbl_name add unique index_name(column_list);
	创建索引的值必须唯一（NULL可以出现多次）
3) alter table tbl_name add index index_name(column_list);
	添加普通索引，索引值可以出现多次
4) alter table tbl_name add fulltext index_name(column_list);
	指定索引为FULLTEXT，用于全文索引
```

#### 索引设计原则

```
对查询频次高，且数据量比较大的表建立索引

索引字段的的选择，最佳候选列应当从where字句的条件中提取，选最常用，过滤效果最好的列

使用唯一索引，区分度越高，使用索引的效率越高

索引过多，会引入相当高的维护代价，降低DML的效率

使用短索引，会占用磁盘口弄简2u8

利用最左前缀，N个列组合而成的组合索引，相当于是创建了N个索引，如果查询时where字句中使用了组成该索引的前几个字段，那么这条查询SQL可以利用组合索引来提升查询效率
```

## 视图

视图的优势：简单、安全、数据独立

#### 创建视图

```sql
CREATE [OR REPLACE] VIEW view_name [(column_list)] AS select_statment
```

#### 修改视图

```sql
ALTER VIEW view_name [(column_list)] AS select_statment
```

#### 查看视图

```sql
SHOW tables;

SHOW CREATE view view_name;查看视图创建时的命令
```

#### 删除视图

```sql
drop view if exists view view_name;
```

## 存储过程和函数

函数：是一个有返回值的过程

过程：是一个没有返回值的函数

#### 创建存储过程

```sql
CREATE PRCEDURE procedure_name ([proc_parameter[……]])

begin

​	-- SQL语法

end;
```

#### 调用存储过程

```sql
call procedure_name();
```

#### 查看存储过程

```sql
查询db_name数据库中的所有的存储过程

select name from mysql.proc where db='db_name';

查询存储过程的状态信息

show procedure status;

查询某个存储过程的定义

shw create procedute pro_test1 \G;
```

#### 删除存储过程

```sql
DROP PROCEDURE if exists sp_name;
```

## 触发器

确保数据完整性，日志记录，数据校验等操作

# MySQL存储引擎及优化

## 存储引擎

存储引擎就是存储数据，建立索引，更新索引，更新查询数据的等技术的实现方式。存储引擎是基于表的，而不是基于库的

InnoDB：支持事务，支持行级锁，支持外键

InnoDB：.frm文件中存储的是表结构，.idb中存储的是数据和索引，最新的MySQL8没有frm文件了

MyISAM：访问速度快，对事务的完整性没有要求，或者以SELECT、INSERT为主的应用，.frm存储表定义，.MYD存储数据，.MYI存储索引

MEMORY：表的数据存放在内存中

## 优化SQL步骤

### 查看SQL执行频率

```sql
show [session |global] status like 'Com_______';

show [session |global] status like 'Innodb_rows_%';
```

### 定位低效率执行SQL

慢查询日志：mysqld写一个包含所有执行时间超过long_query_time秒的SQL语句的日志文件

show processlist：查看当前MySQL在进行的线程，包括线程的状态，是否锁表等

### explain分析执行计划

在相关语句前加explain

### show profile执行SQL

通过having_profiling 查看

```sql
select @@having_profiling;查看是否支持
select @@profiling;查看是否开启
set profiling;开启profiling
show profiles;查看操作的耗时
show profile for query 5; 指的是上面show profiles执行后的次序为5的命令的详细耗时
show profile all for query 5;更详细的信息
```

## 

# MySQL练习

[有道云笔记 (youdao.com)](https://note.youdao.com/ynoteshare/index.html?id=45d4298f42397bd52ccf6fc716e27ee9&type=note&_time=1666250958696#/)

数据表
--1.学生表
Student(SId,Sname,Sage,Ssex)

--SId 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

--2.课程表
Course(CId,Cname,TId)
--CId --课程编号,Cname 课程名称,TId 教师编号

--3.教师表
Teacher(TId,Tname)
--TId 教师编号,Tname 教师姓名

--4.成绩表
SC(SId,CId,score)
--SId 学生编号,CId 课程编号,score 分数

创建测试数据

学生表 Student

```sql
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2017-12-30' , '女');
insert into Student values('12' , '赵六' , '2017-01-01' , '女');
insert into Student values('13' , '孙七' , '2018-01-01' , '女');
```

科目表 Course

```sql
create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10))
insert into Course values('01' , '语文' , '02')
insert into Course values('02' , '数学' , '01')
insert into Course values('03' , '英语' , '03')
```

教师表 Teacher

```sql
create table Teacher(TId varchar(10),Tname varchar(10))
insert into Teacher values('01' , '张三')
insert into Teacher values('02' , '李四')
insert into Teacher values('03' , '王五')
```

成绩表 SC

```sql
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1))
insert into SC values('01' , '01' , 80)
insert into SC values('01' , '02' , 90)
insert into SC values('01' , '03' , 99)
insert into SC values('02' , '01' , 70)
insert into SC values('02' , '02' , 60)
insert into SC values('02' , '03' , 80)
insert into SC values('03' , '01' , 80)
insert into SC values('03' , '02' , 80)
insert into SC values('03' , '03' , 80)
insert into SC values('04' , '01' , 50)
insert into SC values('04' , '02' , 30)
insert into SC values('04' , '03' , 20)
insert into SC values('05' , '01' , 76)
insert into SC values('05' , '02' , 87)
insert into SC values('06' , '01' , 31)
insert into SC values('06' , '03' , 34)
insert into SC values('07' , '02' , 89)
insert into SC values('07' , '03' , 98)
```

## 练习题目

1. 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
   1.1 查询同时存在" 01 "课程和" 02 "课程的情况
   1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
   1.3 查询不存在" 01 "课程但存在" 02 "课程的情况
2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
3. 查询在 SC 表存在成绩的学生信息
4. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )
   4.1 查有成绩的学生信息
5. 查询「李」姓老师的数量
6. 查询学过「张三」老师授课的同学的信息
7. 查询没有学全所有课程的同学的信息
8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
9. 查询和" 01 "号的同学学习的课程 完全相同的其他同学的信息
10. 查询没学过"张三"老师讲授的任一门课程的学生姓名
11. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
14. 查询各科成绩最高分、最低分和平均分：
    以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
    及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
    要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
    15.1 按各科成绩进行排序，并显示排名， Score 重复时合并名次
16. 查询学生的总成绩，并进行排名，总分重复时保留名次空缺
    16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
18. 查询各科成绩前三名的记录
19. 查询每门课程被选修的学生数
20. 查询出只选修两门课程的学生学号和姓名
21. 查询男生、女生人数
22. 查询名字中含有「风」字的学生信息
23. 查询同名同性学生名单，并统计同名人数
24. 查询 1990 年出生的学生名单
25. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
26. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
30. 查询不及格的课程
31. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
32. 求每门课程的学生人数
33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
34. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
36. 查询每门功成绩最好的前两名
37. 统计每门课程的学生选修人数（超过 5 人的课程才统计）。
38. 检索至少选修两门课程的学生学号
39. 查询选修了全部课程的学生信息
40. 查询各学生的年龄，只按年份来算
41. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
42. 查询本周过生日的学生
43. 查询下周过生日的学生
44. 查询本月过生日的学生
45. 查询下月过生日的学生


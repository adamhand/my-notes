# leetcode之mysql题解
---

## [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)
```
Write a SQL query to get the second highest salary from the Employee table.
```
```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```
```
For example, given the above Employee table, the query should return 200 as the second highest salary. If there is no second highest salary, then the query should return null.
```
```
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

需要注意对查询条件为`null`这种情况的处理。下面给出建表、插入数据和选择语句。
```
create table Employee(
	Id bigint not null primary key auto_increment,
	Salary bigint not null
)engine=InnoDB;

insert into Employee (Id,Salary)values(1,100),(2,200),(3,300);

select 
	ifnull(
		(select distinct Salary
			from Employee
			order by Salary desc
			limit 1 offset 1),null)
			as SecondHighestSalary;
```
`ifnull()(expr1,expr2)`函数的含义是：如果第一个参数不为空，则返回第一个参数，否则返回第二个参数。

## [196. Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)
题目描述：
```
Write a SQL query to delete all duplicate email entries in a table named Person, keeping only unique emails based on its smallest Id.
```
```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
```
```
Id is the primary key column for this table.
For example, after running your query, the above Person table should have the following rows:
```
```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```
```
Note:

Your output is the whole Person table after executing your sql. Use delete statement.
```

思路：
这个题可以使用`cross join`来解决。`cross join`其实就是**笛卡尔积**，执行`cross join`时会将两个表相乘。它的写法有：
```sql
SELECT * FROM table1 CROSS JOIN table2

SELECT * FROM table1,table2
```
比如，有一个`Person`表和一个`Employee`表，它们的结构分别如下：
```
# Person
+----+------------------+
| Id | Email            |
+----+------------------+
|  1 | john@example.com |
|  2 | bob@example.com  |
|  3 | john@example.com |
+----+------------------+
#Employee
+----+--------+
| Id | Salary |
+----+--------+
|  1 |    100 |
|  2 |    200 |
|  3 |    300 |
+----+--------+
```
当执行
```
select* from Employee e, Person p;
```
时的结果如下：
```
+----+--------+----+------------------+
| Id | Salary | Id | Email            |
+----+--------+----+------------------+
|  1 |    100 |  1 | john@example.com |
|  2 |    200 |  1 | john@example.com |
|  3 |    300 |  1 | john@example.com |
|  1 |    100 |  2 | bob@example.com  |
|  2 |    200 |  2 | bob@example.com  |
|  3 |    300 |  2 | bob@example.com  |
|  1 |    100 |  3 | john@example.com |
|  2 |    200 |  3 | john@example.com |
|  3 |    300 |  3 | john@example.com |
+----+--------+----+------------------+
```
可以看到，**最终得到的表结构是将前一个表的每一行分别和后一个表的第一行、第二行...连接形成的**。

对于本题而言，就可以按照如下语句选择出合适的行：
```
SELECT p1.*
FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id
;
```
将`select`语句替换成`delete`语句就可以了：
```
delete p1 from Person p1,
	Person p2
where
	p1.Email = p2.Email and p1.Id > p2.Id;
```
最后附上建表和插入语句：
```
drop table if exists Person;			
create table Person(
	Id tinyint not null primary key auto_increment,
	Email varchar(30) not null
)engine=InnoDB;

insert into Person (Id, Email)values(1,'john@example.com'),(2,'bob@example.com'),(3,'john@example.com');
```

---
### mysql中的join小结
`join`称为连接，连接的主要作用是根据两个或多个表中的列之间的关系，获取存在于不同表中的数据。连接可以分为以下几类：

- 内连接：在`mysql`中以关键字`inner join`表示
- 外连接
    - 左外连接：在`mysql`中以关键字`left join`表示
    - 右外连接：在`mysql`中以关键字`rign join`表示
- 交叉连接：在`mysql`中以关键字`cross join`表示，在`mysql`中，交叉连接和内连接功能等价
- 全连接：`mysql`没有具体的关键字来表示，但是可以通过`union`关键字实现；`oracle`提供了 `full join`关键字完成这一功能。


下面通过具体的例子来分析。假如有两个表`t1`和`t2`，建表和插入语句如下：
```sql
drop table if exists t1;
create table t1(
	id tinyint not null primary key auto_increment,
	name varchar(30) not null,
	description varchar(255) default null
)engine=InnoDB;

create table t2 like t1;

insert into t1 values(1,'林丹','获得12奥运会冠军'),(2,'李宗伟','获得12奥运会亚军'),(3,'谌龙','获得12奥运会季军');

insert into t2 values(1,'谌龙','获得16奥运会冠军'),(2,'李宗伟','获得16奥运会亚军'),(3,'安赛龙','获得16奥运会季军');
```
两个表的结构分别如下：

<center>
表`t1`
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/table_t1.PNG">
</center>

<center>
表`t2`
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/table_t2.PNG">
</center>

#### **交叉连接(cross join)**
`cross join`指的就是两个表的笛卡尔积。上面说了，在`Mysql`中，`Cross Join` 和 `Inner Join` 是等价的，但是在标准`SQL`中，它们并不等价，`Inner Join` 用于带有`on`表达式的连接，反之用`Cross Join`。

以下几种方式均可以产生笛卡尔积：
```
select * from t1 cross join t2;
select * from t1 inner join t2;
select * from t1 join t2;
select * from t1,t2;
select * from t1 nature join t2;
select * from t1 natura join t2;
```
产生的结果集为：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_cross_join_1.PNG">
</center>

#### **内连接(inner join)**
通过上面的可以看到，内连接和交叉连接在`mysql`中的效果是一样的。但是内连接一般会配合`on`一起使用，用于在结果选出符合某个条件的记录，即两个表的交集。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_1.png">
</center>

下面的语句是在`cross join`集中选择`name`字段相等的记录。
```
select * from t1 inner join t2 on t1.name=t2.name;
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/inner_join_1.PNG">
</center>

下面的语句是只在`t1`中选择，而不是在`cross join`集中。`tt1`和`tt2`是交叉表中左右两个表的名字。
```
select tt1.* 
	from 
		t1 tt1 
		inner join 
		t2 tt2 
		on tt1.name=tt2.name
;
# 不取实例，和上面语句的结果一样
select t1.* 
	from 
		t1 
		inner join 
		t2 
		on t1.name=t2.name
;
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/inner_join_2.PNG">
</center>

#### **左外连接(left join)**
从左表产生一套完整的记录，还有右边匹配的记录，如果没有匹配就包含null。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_2.png">
</center>

```
select * 
	from 
		t1 tt1 
		left join 
		t2 tt2 on tt1.name=tt2.name;
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/left_join_1.PNG">
</center>

只查询左表的数据，不包含右表的，使用`where` 限制右表`key为null`。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_4.png">
</center>

```
select * 
	from 
		t1 tt1 
		left join 
		t2 tt2 
		on tt1.name=tt2.name 
		where tt2.name is null;
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/left_join_3.PNG">
</center>

#### **右外连接(right join)**
从右表产生一套完整的记录，还有左边匹配的记录，如果没有匹配就包含null。和上述左连接类似。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_3.png">
</center>

需要注意的是，**左连接表`1`、表`2`等价于右连接表`2`、表`1`**。
```
#这两个语句等价
select * from t2 right join t1 using(name);
select * from t1 left join t2 using(name);
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/left_join_and_right_join.PNG">
</center>

只查询右表的数据，不包含左表的，使用`where` 限制右表`key为null`。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_5.png">
</center>

```
select * 
	from 
		t1 tt1 
		right join 
		t2 tt2 
		on tt1.name=tt2.name 
		where tt1.name is null;
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/right_join.PNG">
</center>

#### **全连接(full join)**
使用`union`语句实现全连接。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_6.png">
</center>

```
select * 
	from 
		t1	
		left join 
		t2
		on t1.name=t2.name
	union
select  *
	from
		t1
		right join
		t2
		on t1.name=t2.name;
```
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/full_join.PNG">
</center>

#### **求差集**
两表的全连接中除去重合的部分，即两张表分别的特有部分的合集。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/join_7.png">
</center>

```
select * 
	from 
		t1	
		left join 
		t2
		on t1.name=t2.name
		where t2.name is null
	union
select  *
	from
		t1
		right join
		t2
		on t1.name=t2.name
		where t1.name is null;
```		
产生的结果集如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_chaji_join.PNG">
</center>

#### **参考**
[MySQL的join关键字详解](https://www.jianshu.com/p/76c90b03b7bd)
[MySQL之join语句](https://blog.csdn.net/jyxmust/article/details/70304333)
[MySQL的JOIN用法](https://blog.csdn.net/yanqianglifei/article/details/82883226)
[MySQL的Join使用](https://blog.csdn.net/taylor_tao/article/details/7068511)
[Mysql：ON 与 WHERE 的区别](https://www.jianshu.com/p/d923cf8ae25f)
[SQL中的ON和WHERE的区别](https://blog.csdn.net/liitdar/article/details/80817957)

---

##[197. Rising Temperature](https://leetcode.com/problems/rising-temperature/submissions/)
描述：
```
Given a Weather table, write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.
```
```
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
For example, return the following Ids for the above Weather table:
```
```
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```
思路：
使用`inner join`。下面使用到的`TO_DAYS`函数返回一个天数： 从年份`0`开始的天数。
比如`SELECT TO_DAYS(‘1997-10-07′);`的结果Wie`729669`，就是从`0`年开始 到`1997`年`10`月`7`号之间的天数。


```
SELECT t1.Id
    FROM 
        Weather t1
            INNER JOIN 
        Weather t2
        ON TO_DAYS(t1.RecordDate) = TO_DAYS(t2.RecordDate) + 1
        WHERE t1.Temperature > t2.Temperature;    
```
还可以使用`datediff()`函数。它的语法为`DATEDIFF(date1,date2)`。比如`SELECT DATEDIFF('2008-12-30','2008-12-29') AS DiffDate`的结果为`1`。
```
select 
	w1.Id as 'id'
from 
	Weather w1
        inner join
	Weather w2
	on datediff(w1.RecordDate, w2.RecordDate)=1
	where w1.Temperature > w2.Temperature
;
```

## [627. Swap Salary](https://leetcode.com/problems/swap-salary/)
描述：
```
Given a table salary, such as the one below, that has m=male and f=female values. Swap all f and m values (i.e., change all f values to m and vice versa) with a single update query and no intermediate temp table.
``` 

For example:
 
```
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
```
After running your query, the above salary table should have the following rows:
```
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

思路：
可以使用`case-when-else`语句。
```
UPDATE salary
    SET sex  = (CASE WHEN sex = 'm' 
        THEN  'f' 
        ELSE 'm' 
        END);
```
还可以使用`if`语句。
```
UPDATE salary SET sex = IF(sex = 'm', 'f', 'm')
```

### case语句小结
`case` 具有两种格式：

- 简单 `case` 函数：将某个表达式与一组简单表达式进行比较以确定结果。
- `case` 搜索函数：计算一组布尔表达式以确定结果。

```
# 简单case函数
case input_expression
    when when_expression_1 then
        result_expression_1
    when when_expression_2 then
        result_expression_2
    else
        else_result_expression
end

# case搜索函数
case
    when Boolean_expression_1 then
        result_expression_1
    when Boolean_expression_2 then
        result_expression_2    
    else
        else_result_expression
end
```
下面举个例子，首先建立`casetest`表并插入语句，建表语句和插入数据语句如下：
```
drop table if exists casetest;
create table casetest(
	id tinyint not null primary key auto_increment,
	name varchar(30) not null,
	gender varchar(13) not null,
	birthday date not null
)engine=InnoDB;

insert into casetest values(1,'Bob','male','1895-03-26'),(2,'Alice','female','1999-3-10'),(3,'Tom','male','1995-02-21'),(4,'Jerry','male','1885-03-14'),(5,'Dog','female','1996-05-23');
```
表的结构如下：
```
+----+-------+--------+------------+
| id | name  | gender | birthday   |
+----+-------+--------+------------+
|  1 | Bob   | male   | 1895-03-26 |
|  2 | Alice | female | 1999-03-10 |
|  3 | Tom   | male   | 1995-02-21 |
|  4 | Jerry | male   | 1885-03-14 |
|  5 | Dog   | female | 1996-05-23 |
+----+-------+--------+------------+
```

#### **使用简单case函数进行选择**
```
select *,	
	case
		when birthday<'1981' then 'old'
		when birthday>'1988' then 'yong'
		else 'ok' end yorn
	from casetest
;
```
结果集如下：
```+----+-------+--------+------------+------+
| id | name  | gender | birthday   | YORN |
+----+-------+--------+------------+------+
|  1 | Bob   | male   | 1895-03-26 | old  |
|  2 | Alice | female | 1999-03-10 | yong |
|  3 | Tom   | male   | 1995-02-21 | yong |
|  4 | Jerry | male   | 1885-03-14 | old  |
|  5 | Dog   | female | 1996-05-23 | yong |
+----+-------+--------+------------+------+
```

#### **使用case搜索函数进行选择**
```
select *,	
	case name
		when 'Bob' then 'old'
		when 'Alice' then 'yong'
		when 'Tom' then 'old'
		else 'ok' end yorn
	from casetest
;
```
结果集如下：
```
+----+-------+--------+------------+------+
| id | name  | gender | birthday   | YORN |
+----+-------+--------+------------+------+
|  1 | Bob   | male   | 1895-03-26 | old  |
|  2 | Alice | female | 1999-03-10 | yong |
|  3 | Tom   | male   | 1995-02-21 | old  |
|  4 | Jerry | male   | 1885-03-14 | ok   |
|  5 | Dog   | female | 1996-05-23 | ok   |
+----+-------+--------+------------+------+
```

#### 使用cas搜索函数进行更新
下面语句的功能是将表中的`male`和`female`互换。
```
update casetest
	set gender = (
			case gender	
				when 'male' then 'female'
				else 'male'
				end
	)
;
```
之后使用`select`进行选择，结果集如下：
```
+----+-------+--------+------------+
| id | name  | gender | birthday   |
+----+-------+--------+------------+
|  1 | Bob   | female | 1895-03-26 |
|  2 | Alice | male   | 1999-03-10 |
|  3 | Tom   | female | 1995-02-21 |
|  4 | Jerry | female | 1885-03-14 |
|  5 | Dog   | male   | 1996-05-23 |
+----+-------+--------+------------+
```

### if语句小结
基本语法为：
```
IF(condition, value_if_true, value_if_false)
```
IF函数根据条件的结果为true或false，返回第一个值，或第二个值。例子如下。
```
select *,if(gender='male',1,2) as gender_id from casetest;
+----+-------+--------+------------+-----------+
| id | name  | gender | birthday   | gender_id |
+----+-------+--------+------------+-----------+
|  1 | Bob   | female | 1895-03-26 |         2 |
|  2 | Alice | male   | 1999-03-10 |         1 |
|  3 | Tom   | female | 1995-02-21 |         2 |
|  4 | Jerry | female | 1885-03-14 |         2 |
|  5 | Dog   | male   | 1996-05-23 |         1 |
+----+-------+--------+------------+-----------+
```

## [184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)
描述：
```
The Employee table holds all employees. Every employee has an Id, a salary, and there is also a column for the department Id.
```
```
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
```
```
The Department table holds all departments of the company.
```
```
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
Write a SQL query to find employees who have the highest salary in each of the departments. For the above tables, Max has the highest salary in the IT department and Henry has the highest salary in the Sales department.

+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```
思路：可以使用`group by`和`in`来操作。先给出建表语句和插入行的语句：
```
drop table if exists employee;
create table employee(
	id tinyint not null primary key auto_increment,
	name varchar(30) not null,
	salary int not null,
	departmentid tinyint not null
)engine=InnoDB;

drop if exists department;
create table department(
	id tinyint not null primary key auto_increment,
	name varchar(30) not null
)engine=InnoDB;

insert into employee values(1,'Joe',70000,1),(2,'Henry',80000,2),(3,'Sam',60000,2),(4,'Max',90000,1);
insert into department values(1,'IT'),(2,'Sales');
```
使用
```
select 
	departmentid as departmentid,
	max(salary) as maxsalary
from 
	employee
group by departmentid;
```
可以查找到如下结果集：
```
+--------------+-----------+
| departmentid | maxsalary |
+--------------+-----------+
|            1 |     90000 |
|            2 |     80000 |
+--------------+-----------+
```
将这个结果集作为`in`的一部分，可以得到最终语句：
```
select 
	department.name as Department,
	employee.name as Employee,
	employee.salary as Salary
from
	employee
		join
	department on employee.departmentid=department.id
where
	(employee.departmentid, salary) in
	(
		select
			departmentid, max(salary)
		from
			employee group by departmentid
	)
;
```

## [178. Rank Scores](https://leetcode.com/problems/rank-scores/)
```
Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no "holes" between ranks.

+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
For example, given the above Scores table, your query should generate the following report (order by highest score):

+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

先给出建表语句：
```
drop table if exists scores;
create table scores(
	id int not null primary key auto_increment,
	score float not null
)engine=InnoDB;

insert into scores values(1,3.50),(2,3.65),(3,4.00),(4,3.85),(5,4.00),(6,3.65);
```

解法：
```
select score, 
	(select count(distinct score) from scores where score>=s.score) as `rank` 
	from scores s order by score desc;
```
分析：这个语句中有4个score，先不看中间嵌套的select语句，这个语句的主干是：`select score from scores s order by score desc;`，所以，第一个score是属于表s的；而第二个和第三个score是数据中间嵌套的select语句的，这样这个语句就分解清楚了，搜索结果包括两列：**第一列：分数**；**第二列：大于等于此分数的分数值的不重复个数**；按分数降序排列。

## [180. Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)
```
Write a SQL query to find all numbers that appear at least three times consecutively.

+----+-----+
| Id | Num |
+----+-----+
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |
+----+-----+
For example, given the above Logs table, 1 is the only number that appears consecutively for at least three times.

+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```
思路，还是用join。一个join走天下。使用
```
select *
from 
	logs l1, 
	logs l2, 
	logs l3
where 
	l1.id=l2.id-1 
	and l2.id=l3.id-1 
	and l1.num=l2.num 
	and l2.num=l3.num;
```
语句之后的结果如下：
```
+----+-----+----+-----+----+-----+
| id | num | id | num | id | num |
+----+-----+----+-----+----+-----+
|  1 |   1 |  2 |   1 |  3 |   1 |
+----+-----+----+-----+----+-----+
```
将上面的语句稍微改变一下就得到该题的结果：
```
select distinct l1.num as `ConsecutiveNums` 
from 
	logs l1, 
	logs l2, 
	logs l3
where 
	l1.id=l2.id-1 
	and l2.id=l3.id-1 
	and l1.num=l2.num 
	and l2.num=l3.num;
```

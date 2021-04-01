# MySQL常用语法小结
---
# 一、基础
---
模式定义了数据如何存储、存储什么样的数据以及数据如何分解等信息，数据库和表都有模式。

主键的值不允许修改，也不允许复用（不能使用已经删除的主键值赋给新数据行的主键）。

SQL（Structured Query Language)，标准 SQL 由 ANSI 标准委员会管理，从而称为 ANSI SQL。各个 DBMS 都有自己的实现，如 PL/SQL、Transact-SQL 等。

SQL 语句不区分大小写，但是数据库表名、列名和值是否区分依赖于具体的 DBMS 以及配置。

## MySQL服务的启动、停止与卸载
在 Windows 命令提示符下运行:
```sql
启动: net start MySQL
停止: net stop MySQL
卸载: sc delete MySQL
```

## MySQL数据类型
MySQL有三大类数据类型, 分别为数字、日期\时间、字符串, 这三大类中又更细致的划分了许多子类型:

**数字类型**
```sql
整数: tinyint、smallint、mediumint、int、bigint
浮点数: float、double、real、decimal
```
**日期和时间**
```sql
 date、time、datetime、timestamp、year
```
**字符串类型**
```sql
字符串: char、varchar
文本: tinytext、text、mediumtext、longtext
二进制(可用来存储图片、音乐等): tinyblob、blob、mediumblob、longblob
```

## 登录
```sql
mysql -h 主机名 -u 用户名 -p
```
>- -h : 该命令用于指定客户端所要登录的MySQL主机名, 登录当前机器该参数可以省略;
>- -u : 所要登录的用户名;
>- -p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项

然后命令提示符会一直以 mysql> 加一个闪烁的光标等待命令的输入, 输入 exit 或 quit 退出登录。

## 注释

SQL 支持以下三种注释：
```sql
# 注释
SELECT *
FROM mytable; -- 注释
/* 注释1
   注释2 */
```

# 二、创建或导入数据库和删除数据库
## 创建数据库
使用 create database 语句可完成对数据库的创建, 创建命令的格式如下:
```sql
create database 数据库名 [其他选项];
```
例如我们需要创建一个名为 samp_db 的数据库并使用gbk字符, 在命令行下执行以下命令:
```sql
create database samp_db character set gbk;
```
创建一个数据库使用utf8字符，
```sql
create database db_name default charset utf8;
```
选择要用的数据库：
```sql
use samp_db;
```
MySQL语句以分号(;)作为语句的结束, 但use语句可以不加分号。

## 导入数据库
假如当前有一个完整的sql文件samp_db.sql，只需把它倒入数据库就行。首先还是create和use数据库(数据库名称不要求和sql文件同名)，然后使用
```sql
source D:\worksp\samp_db.sql;
```
就可以将数据库导入到数据库服务器中，但可能要花费一段时间。

## 删除数据库
```sql
drop database samp_db;
```

# 三、创建表
使用如下命令创建表：
```sql
create table 表名称(列声明);
```
例如：
```sql
create table students
(
	id int unsigned not null auto_increment primary key,
	name char(8) not null,
	sex char(4) not null,
	age tinyint unsigned not null,
	tel char(13) null default "-"
);
```
查询所有的数据库：
```sql
show databases;
```
查询某个数据库中的所有表：
```sql
show tables;
```
查询某个表中的所有列(比如查询表customers)：
```sql
show columns from customers;
```
查询某个表的详细信息：
```sql
describe customers;
```
# 四、修改表
alter table 语句用于创建后对表的修改, 基础用法如下:
## 添加列
基本形式: 
```sql
alter table 表名 add 列名 列数据类型 [after 插入位置];
```
例如：
在表的最后追加列 address: 
```sql
alter table students add address char(60);
```
在名为 age 的列后插入列 birthday: 
```sql
alter table students add birthday date after age;
```
## 修改列
基本形式: 
```sql
alter table 表名 change 列名称 列新名称 新数据类型;
```
示例:
将表 tel 列改名为 telphone: 
```sql
alter table students change tel telphone char(13) default "-";
```
将 name 列的数据类型改为 char(16): 
```sql
alter table students change name name char(16) not null;
```

## 删除列
基本形式: 
```sql
alter table 表名 drop 列名称;
```
示例:
删除 birthday 列: 
```sql
alter table students drop birthday;
```

## 重命名表
基本形式: 
```sql
alter table 表名 rename 新表名;
```
示例:
重命名 students 表为 workmates: 
```sql
alter table students rename workmates;
```

## 删除整张表
基本形式: 
```sql
drop table 表名;
```
示例: 删除 workmates 表: 
```sql
drop table workmates;
```




# 五、表内的增删改查
## 插入数据
insert 语句可以用来将一行或多行数据插到数据库表中, 使用的一般形式如下:
```sql
insert [into] 表名 [(列名1, 列名2, 列名3, ...)] values (值1, 值2, 值3, ...);
```
其中 [] 内的内容是可选的, 例如, 要给 samp_db 数据库中的 students 表插入一条记录, 执行语句:
```sql
insert into students values(NULL, "王刚", "男", 20, "13811371377");
```
有时我们只需要插入部分数据, 或者不按照列的顺序进行插入, 可以使用这样的形式进行插入:
```sql
insert into students (name, sex, age) values("孙丽华", "女", 21);
```

## 查询数据
### 基本用法
select 语句常用来根据一定的查询规则到数据库中获取数据, 其基本的用法为: 
```sql
select 列名称 from 表名称 [查询条件];
```
例如要查询 students 表中所有学生的名字和年龄, 输入语句 
```sql
select name, age from students;
```
也可以使用通配符 * 查询表中所有的内容, 语句: 
```sql
select * from students;
```

### 和insert或create配合
使用SELECT语句返回的列和值来填充INSERT语句的值。 
```sql
intert into table_1 select c1, c2, from table_2;
```
这样做的前提是两个表具有相同的表结构，通过复制一个表得到一个相同表结构的语句为：
```sql
create table tasks_bak like tasks;
```
类似的还有将一个表的内容插入到一个新表
```sql
create table newtable as select * from mytable;
```

### 按条件查询where
where 关键词用于指定查询条件, 用法形式为: 
```sql
select 列名称 from 表名称 where 条件;
```

where 子句不仅仅支持 "where 列名 = 值" 这种名等于值的查询形式, 对一般的比较运算的运算符都是支持的, 例如 `=、>、<、>=、<、!=` 以及一些扩展运算符 `is [not] null、in、like` 等等。 还可以对查询条件使用 `or` 和 `and` 进行组合查询。

示例:
查询年龄在21岁以上的所有人信息: 
```sql
select * from students where age > 21;
```
查询名字中带有 "王" 字的所有人信息: 
```sql
select * from students where name like "%王%";
```
查询id小于5且年龄大于20的所有人信息: 
```sql
select * from students where id<5 and age>20;
```

### distinct关键字
这个关键字用来在查询的结果中去重，主要有以下几种用法：
#### 作用于单列
```sql
select distinct [列名] from [表名];
```
例如
```sql
select distinct username from t_user;
```
假如username有重复的，只显示一次。

#### 作用于多列
```sql
select distinct [列名1], [列名2] from [表名];
```
例如：
```sql
select distinct username, password form t_user;
```
这种方法是根据两列分别去重的结果来输出的，根据去重之后剩余行数较多的列来输出。如下里例子：

表内容为：

|user_id|username|password|
|-|-|-|
|1|admin|123456|
|2|user|111111|
|3|user|222222|

如果根据username和password去重，结果为：

|username|password|
|-|-|
|admin|123456|
|user|111111|
|user|222222|

### limit关键字
限制返回的行数。可以有两个参数，第一个参数为起始行(第一条数据的起始行为0)；第二个参数为返回的总行数。即：
```sql
select * from [表名] limit offset,count
```
offset默认为0，表示从第一行开始查询，count表示要返回的数据行数
```sql
select * from [表名] limit 100,2
```
等同于
```sql
select * from [表名] limit 2 offset 100
```

```sql
select * from [表名] limit 3; # 从第一条数据开始，共返回3行
```

```sql
select * from [表名] limit 1, 2; # 从第二条数据开始，共返回2行
```


## 更新语句
update 语句可用来修改表中的数据, 基本的使用形式为:
```sql
update 表名称 set 列名称=新值 where 更新条件;
```
使用示例:
将id为5的手机号改为默认的"-": 
```sql
update students set tel=default where id=5;
```
将所有人的年龄增加1: 
```sql
update students set age=age+1;
```
将手机号为 13288097888 的姓名改为 "张伟鹏", 年龄改为 19: 
```sql
update students set name="张伟鹏", age=19 where tel="13288097888";
```
---
同样，update语句也可以和select语句结合使用。

---

## 删除语句
delete 语句用于删除表中的数据, 基本用法为:
```sql
delete from 表名称 where 删除条件;
```
使用示例:
删  除id为2的行: 
```sql
delete from students where id=2;
```
删除所有年龄小于21岁的数据: 
```sql
delete from students where age<20;
```
删除表中的所有数据: 
```sql
delete from students;
```

# 六、排序
>- ASC ：升序（默认）
>- DESC ：降序

```sql
select prod_id, prod_price, prod_name from products order by prod_price, prod_name; # 默认升序。先按照prod_price的升序排列，再按照prod_name的升序排列
```

可以按多个列进行排序，并且为每个列指定不同的排序方式：
```sql
select * from mytable order by col1 desc, col2 asc;
```

# 七、过滤
不进行过滤的数据非常大，导致通过网络传输了多余的数据，从而浪费了网络带宽。因此尽量使用 SQL 语句来过滤不必要的数据，而不是传输所有的数据到客户端中然后由客户端进行过滤。
```sql
select * from mytable where id is null;
```

下表显示了 WHERE 子句可用的操作符

|操作符	|说明|
|-|-|
|=|	等于|
|<|	小于|
|>|	大于|
|`<>` `!=`|	不等于|
|`<=`	|小于等于|
|`>=`	|大于等于|
|BETWEEN	|在两个值之间|
|IS NULL	|为 NULL 值|

应该注意到，NULL 与 0、空字符串都不同。

**AND** 和 **OR** 用于连接多个过滤条件。优先处理 AND，当一个过滤表达式涉及到多个 AND 和 OR 时，可以使用 () 来决定优先级，使得优先级关系更清晰。

**IN** 操作符用于匹配一组值，其后也可以接一个 SELECT 子句，从而匹配子查询得到的一组值。
```sql
select prod_name, prod_price from products where vend_id in (1002,1003) order by prod_name;
```

**NOT** 操作符用于否定一个条件。

# 八、通配符
通配符也是用在过滤语句中，但它只能用于文本字段。

>- % 匹配 >=0 个任意字符；
>- _ 匹配 ==1 个任意字符；
>- `[ ]` 可以匹配集合内的字符，例如 [ab] 将匹配字符 a 或者 b。用脱字符 ^ 可以对其进行否定，也就是不匹配集合内的字符。

使用 Like 来进行通配符匹配。
```sql
select * from mytable where col like '[^AB]%'; -- 不以 A 和 B 开头的任意文本
```

# 九、正则表达式


# 十、计算字段
"字段(field)"和列的概念相同，经常混用。

在数据库服务器上完成数据的转换和格式化的工作往往比客户端上快得多，并且转换和格式化后的数据量更少的话可以减少网络通信量。

计算字段通常需要使用 **AS** 来取别名，否则输出的时候字段名为计算表达式
```sql
select col1 * col2 as alias from mytable; # 结果为col1和col2相乘，名称为alias
```

**CONCAT()** 用于连接两个字段。许多数据库会使用空格把一个值填充为列宽，因此连接的结果会出现一些不必要的空格，使用 **TRIM()** 可以去除首尾空格。
```sql
select concat(trim(col1), '(', trim(col2), ')') as concat_col from mytable;
```

# 十一、函数
各个 DBMS 的函数都是不相同的，因此不可移植，以下主要是 MySQL 的函数。

## 汇总

|函 数	|说 明|
|-|-|
|AVG()|	返回某列的平均值|
|COUNT()|	返回某列的行数|
|MAX()|	返回某列的最大值|
|MIN()	|返回某列的最小值|
|SUM()|	返回某列值之和|

AVG() 会忽略 NULL 行。

使用 DISTINCT 可以让汇总函数值汇总不同的值。
```sql
SELECT AVG(DISTINCT col1) AS avg_col FROM mytable;
```

## 文本处理
|函数	|说明|
|-|-|
|LEFT()|	左边的字符|
|RIGHT()|	右边的字符|
|LOWER()|	转换为小写字符|
|UPPER()|	转换为大写字符|
|LTRIM()|	去除左边的空格|
|RTRIM()|	去除右边的空格|
|LENGTH()|	长度|
|SOUNDEX()|	转换为语音值|

其中， SOUNDEX() 可以将一个字符串转换为描述其语音表示的字母数字模式。

```sql
SELECT * FROM mytable WHERE SOUNDEX(col1) = SOUNDEX('apple') # 在col1列中查找所有发音类似于apple的关键字
```

## 日期和时间处理
>- 日期格式：YYYY-MM-DD
>- 时间格式：HH:MM:SS

|函 数|	说 明|
|-|-|
|AddDate()|	增加一个日期（天、周等）|
|AddTime()|	增加一个时间（时、分等）|
|CurDate()|	返回当前日期|
|CurTime()|	返回当前时间|
|Date()	|返回日期时间的日期部分
|DateDiff()	|计算两个日期之差
|Date_Add()|	高度灵活的日期运算函数|
|Date_Format()|	返回一个格式化的日期或时间串|
|Day()|	返回一个日期的天数部分|
|DayOfWeek()|	对于一个日期，返回对应的星期几|
|Hour()|	返回一个时间的小时部分|
|Minute()|	返回一个时间的分钟部分|
|Month()|	返回一个日期的月份部分|
|Now()|	返回当前日期和时间|
|Second()|	返回一个时间的秒部分|
|Time()|	返回一个日期时间的时间部分|
|Year()|	返回一个日期的年份部分|

```sql
SELECT NOW(); # 显示当前时间
```

```sql
select cust_id, order_num from orders where data(order_date) between '2015-09-01' and '2015-09-30';
```

## 数值处理
|函数|	说明|
|-|-|
|SIN()|	正弦|
|COS()|	余弦|
|TAN()|	正切|
|ABS()|	绝对值|
|SQRT()|	平方根|
|MOD()|	余数|
|EXP()|	指数|
|PI()|	圆周率|
|RAND()|随机数|

```sql
select abs(-12);
```

```sql
select sin(20);
```

# 十二、分组
分组就是把具有相同的数据值的行放在同一组中。比如，有两行数据中的username相同，那么group by username的时候，这两行会合成一行。

可以对同一分组数据使用汇总函数进行处理，例如求分组数据的平均值等。

指定的分组字段除了能按该字段进行分组，也会自动按该字段进行排序。
```sql
SELECT col, COUNT(*) AS num FROM mytable GROUP BY col;
```

GROUP BY 自动按分组字段进行排序，ORDER BY 也可以按汇总字段来进行排序。
```sql
SELECT col, COUNT(*) AS num FROM mytable GROUP BY col ORDER BY num;
```
WHERE 过滤行，HAVING 过滤分组，行过滤应当先于分组过滤。
```sql
SELECT col, COUNT(*) AS num FROM mytable WHERE col > 2 GROUP BY col HAVING num >= 2;
```
分组规定：

>- GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；
>- 除了汇总字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；
>- NULL 的行会单独分为一组；
>- 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。

# 十三、子查询
子查询中只能返回一个字段的数据。

可以将子查询的结果作为 WHRER 语句的过滤条件：
```sql
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
               FROM mytable2);
```
下面的语句可以检索出客户的订单数量，子查询语句会对第一个查询检索出的每个客户执行一次：
```sql
SELECT cust_name, (SELECT COUNT(*)
                   FROM Orders
                   WHERE Orders.cust_id = Customers.cust_id)
                   AS orders_num
FROM Customers
ORDER BY cust_name;
```


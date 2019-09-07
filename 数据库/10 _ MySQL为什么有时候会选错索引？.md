10 | MySQL为什么有时候会选错索引？

# 10 | MySQL为什么有时候会选错索引？
在`mysql`中，使用哪个索引默认是由优化器来选择的，但是，有时候优化器会选错索引，导致语句执行得很慢。下面先看一个例子。

首先建立一张表，语句如下：

```sql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`),
  KEY `b` (`b`)
) ENGINE=InnoDB；
```
然后，往表 t 中插入 10 万行记录，取值按整数递增，即：(1,1,1)，(2,2,2)，(3,3,3) 直到 (100000,100000,100000)。语句如下：

```sql
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000)do
    insert into t values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
```
之后，使用以下语句进行查询，是会用到索引`a`的，这里一般不会出问什么问题。

```sql
mysql> select * from t where a between 10000 and 20000;
```

接着，在表`t`中进行如下操作：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/1e5ba1c2934d3b2c0d96b210a27e1a1e.png">
</div>

首先Session A开启了一个事务。随后，session B 把数据都删除后，又调用了 idata 这个存储过程，插入了 10 万行数据。这时候，session B 的查询语句 select * from t where a between 10000 and 20000 就不会再选择索引 a 了。可以通过慢查询日志（slow log）来查看一下具体的执行情况。关于如何查看慢查询，可以参见附录。

```sql
set long_query_time=0;
select * from t where a between 10000 and 20000; /*Q1*/
select * from t force index(a) where a between 10000 and 20000;/*Q2*/
```

上述三句话含义如下：

- 第一句，是将慢查询日志的阈值设置为 0，表示这个线程接下来的语句都会被记录入慢查询日志中；
- 第二句，Q1 是 session B 原来的查询；
- 第三句，Q2 是加了 force index(a) 来和 session B 原来的查询语句执行情况对比。force index(a)的作用是强制使用索引a。

下图是三条SQL语句执行完成后的慢查询日志。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/7c58b9c71853b8bba1a8ad5e926de1f6.png">
</div>

可以看到，Q1 扫描了 10 万行，显然是走了全表扫描，执行时间是 40 毫秒。Q2 扫描了 10001 行，执行了 21 毫秒。也就是说，在没有使用 force index 的时候，MySQL 用错了索引，导致了更长的执行时间。

这个例子对应的是平常不断地删除历史数据和新增数据的场景。这时，MySQL 会选错索引。

## 优化器的逻辑
选择索引的任务是优化器来做的，优化器选择索引的标准包括扫描行数、是否使用临时表、是否排序等。因为上面的查询语句没有涉及到临时表和排序，所以选错索引的原因可能是判断扫描行数的时候出错了。

那么，扫描行数是怎么判断的呢?

MySQL 在真正开始执行语句之前，并不能精确地知道满足这个条件的记录有多少条，而只能根据统计信息来估算记录数。</p><p>这个统计信息就是索引的“区分度”。显然，一个索引上不同的值越多，这个索引的区分度就越好。而一个索引上不同的值的个数，我们称之为“基数”（cardinality）。也就是说，这个基数越大，索引的区分度越好。</p><p>可以使用 show index 方法，看到一个索引的基数。下图所示就是表 t 的 show index 的结果 。虽然这个表的每一行的三个字段值都是一样的，但是在统计信息中，这三个索引的基数值并不同，而且其实都不准确。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/16dbf8124ad529fec0066950446079d4.png">
</div>

MySQL得到索引的基数的方式是通过**采样统计**的方式。采样统计的时候，InnoDB 默认会选择 N 个数据页，统计这些页面上的不同值，得到一个平均值，然后乘以这个索引的页面数，就得到了这个索引的基数。</p><p>而数据表是会持续更新的，索引统计信息也不会固定不变。所以，当变更的数据行数超过 1/M 的时候，会自动触发重新做一次索引统计。</p><p>在 MySQL 中，有两种存储索引统计的方式，可以通过设置参数 innodb_stats_persistent 的值来选择：</p>
- 设置为 on 的时候，表示统计信息会持久化存储。这时，默认的 N 是 20，M 是 10。
- 设置为 off 的时候，表示统计信息只存储在内存中。这时，默认的 N 是 8，M 是 16。

由于是采样统计，所以不管 N 是 20 还是 8，这个基数都是很容易不准的。

但，这还不是全部。从上面的结果中看到，这次的索引统计值（cardinality 列）虽然不够精确，但大体上还是差不多的，选错索引一定还有别的原因。

其实索引统计只是一个输入，对于一个具体的语句来说，优化器还要判断，执行这个语句本身要扫描多少行。接下来再一起看看优化器预估的，这两个语句的扫描行数是多少。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/e2bc5f120858391d4accff05573e1289.png">
</div>

rows 这个字段表示的是预计扫描行数。其中，Q1 的结果还是符合预期的，rows 的值是 104620；但是 Q2 的 rows 值是 37116，偏差就大了。是这个偏差误导了优化器的判断。

那么，优化器为什么放着扫描 37000 行的执行计划不用，却选择了扫描行数是 100000 的执行计划呢？

这是因为，如果使用索引 a，每次从索引 a 上拿到一个值，都要回到主键索引上查出整行数据，这个代价优化器也要算进去的。而如果选择扫描 10 万行，是直接在主键索引上扫描的，没有额外的代价。优化器会估算这两个选择的代价，从结果看来，优化器认为直接扫描主键索引更快。当然，从执行时间看来，这个选择并不是最优的。

使用普通索引需要把回表的代价算进去，所以冤有头债有主，MySQL 选错索引，这件事儿还得归咎到没能准确地判断出扫描行数。

可以使用analyze table t命令重新统计索引信息，效果如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/209e9d3514688a3bcabbb75e54e1e49c.png">
</div>

所以在实践中如果发现 explain 的结果预估的 rows 值跟实际情况差距比较大，可以采用这个方法来处理。

下面看另一个语句：

```sql
mysql> select * from t where (a between 1 and 1000)  and (b between 50000 and 100000) order by b limit 1;
```

这个查询没有符合条件的记录，因此会返回空集合。下面分析一下应该选择a索引还是b索引。这两个索引的结构图如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/1d037f92063e800c3bfff3f4dbf1a2b9.png">
</div>

如果使用索引 a 进行查询，那么就是扫描索引 a 的前 1000 个值，然后取到对应的 id，再到主键索引上去查出每一行，然后根据字段 b 来过滤。显然这样需要扫描 1000 行。

如果使用索引 b 进行查询，那么就是扫描索引 b 的最后 50001 个值，与上面的执行过程相同，也是需要回到主键索引上取值再判断，所以需要扫描 50001 行。

所以如果使用索引 a 的话，执行速度明显会快很多。下面看一下实际情况。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/483bcb1ef3bb902844e80d9cbdd73ab8.png">
</div>

上面是使用explain分析的结果，可以看到返回结果中 key 字段显示，这次优化器选择了索引 b，而 rows 字段显示需要扫描的行数是 50198。从这个结果中，你可以得到两个结论：

- 扫描行数的估计值依然不准确
- 这个例子里 MySQL 又选错了索引

## 索引选择异常和处理
其实大多数时候优化器都能找到正确的索引，但偶尔会遇到选错索引的情况，解决方法有几个。

### 使用force index
对于上面的例子，使用force index的结果如下所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/9582401a6bed6cb8fd803c9555750b54.png">
</div>

可以看到，原本语句需要执行 2.23 秒，而当你使用 force index(a) 的时候，只用了 0.05 秒，比优化器的选择快了 40 多倍。

### 修改语句，引导 MySQL 使用正确的索引
在这个例子里，显然把“order by b limit 1” 改成 “order by b,a limit 1” ，语义的逻辑是相同的，改之后的效果：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/14cd598e52a2b72dd334a42603e5b894.png">
</div>

之前优化器选择使用索引 b，是因为它认为使用索引 b 可以避免排序（b 本身是索引，已经是有序的了，如果选择索引 b 的话，不需要再做排序，只需要遍历），所以即使扫描行数多，也判定为代价更小。</p><p>现在 order by b,a 这种写法，要求按照 b,a 排序，就意味着使用这两个索引都需要排序。因此，扫描行数成了影响决策的主要条件，于是此时优化器选了只需要扫描 1000 行的索引 a。</p><p>当然，这种修改并不是通用的优化手段，只是刚好在这个语句里面有 limit 1，因此如果有满足条件的记录， order by b limit 1 和 order by b,a limit 1 都会返回 b 是最小的那一行，逻辑上一致，才可以这么做。

还有一种改法
```sql
mysql> select * from  (select * from t where (a between 1 and 1000)  and (b between 50000 and 100000) order by b limit 100)alias limit 1;
```

执行效果如下图所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/14cd598e52a2b72dd334a42603e5b894.png">
</div>

用 limit 100 让优化器意识到，使用 b 索引代价是很高的(**为啥能起到这个作用？**)。

### 新建一个更合适的索引或删掉误用的索引
这也是一种解决方案。

## 附录：如何查看慢查询
和慢查询相关的变量主要由三个：

```sql
slow_query_log          # 默认是OFF,表示禁止慢查询
slow_query_log_file     # 慢查询存放的位置
long_query_time         # 只有大于long_query_time的查询才会被计做慢查询，默认是10s
log_output              # 表示日志存储的方式，默认值是'FILE'，表示存入文件中。log_output='TABLE'表示将日志存入数据库，这样日志信息就会被写入到mysql.slow_log表中
```
上述变量都可以通过`show variables like`命令来查看。

如果要使用慢查询功能，首先需要打开该功能，命令如下：

```sql
set global slow_query_log=1;
```
`log_output`命令不用变，表示将日志存入文件中，之后查看文件存放的位置：

```sql
show variables like "slow_query_log_file";
```
得到的结果如下：

```sql
+---------------------+---------------------------------------------------------------------------+
| Variable_name       | Value                                                                     |
+---------------------+---------------------------------------------------------------------------+
| slow_query_log_file | J:\ProfessionalSoftware\mysql-5.6.30-winx64\data\DESKTOP-IMQP1IF-slow.log |
+---------------------+---------------------------------------------------------------------------+
```

然后将long_query_time设置为自己希望的值，就可以进行查询操作了。为了查看慢查询日志，可以借助mysql自带的一个工具：mysqldumpslow。这个工具一般在mysql安装目录的bin文件夹中，windows上是以mysqldumpslow.pl文件存在的。这是一个perl程序，要运行它需要perl环境，具体的安装可以参考[这里](https://www.cnblogs.com/gyfluck/p/9969714.html)。

安装完之后，通过下面的命令就可以查看mysqldumpslow工具的可用参数:

```sql
perl mysqldumpslow.pl --help
```
可用参数如下所示。

```
--verbose    verbose
--debug      debug
--help       write this text to standard output

-v           verbose
-d           debug
-s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
            al: average lock time
            ar: average rows sent
            at: average query time
                c: count
                l: lock time
                r: rows sent
                t: query time
-r           reverse the sort order (largest last instead of first)
-t NUM       just show the top n queries
-a           don't abstract all numbers to N and strings to 'S'
-n NUM       abstract numbers with at least n digits within names
-g PATTERN   grep: only consider stmts that include this string
-h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
            default is '*', i.e. match all
-i NAME      name of server instance (if using mysql.server startup script)
-l           don't subtract lock time from total time
```

接着，使用下面的命令查看慢查询日志：

```sql
perl mysqldumpslow.pl -s r -t 10 J:\ProfessionalSoftware\mysql-5.6.30-winx64\data\DESKTOP-IMQP1IF-slow.log
```
如下是一条结果：

```sql
Count: 1  Time=0.01s (0s)  Lock=0.00s (0s)  Rows=10001.0 (10001), root[root]@localhost
  select * from t where a between N and N
```            
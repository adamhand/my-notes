# 34 | 到底可不可以使用join？
---

在实际生产中，使用`join`时总是会面临两个问题：

- 性能问题
- 使用哪个表做驱动表的问题

下面来分析这两个问题，还是使用例子来说明，建立两个表，以这两个表为例。
```
CREATE TABLE `t2` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`)
) ENGINE=InnoDB;

drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=1000)do
    insert into t2 values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();

create table t1 like t2;
insert into t1 (select * from t2 where id<=100)
```
这两个表都有一个主键索引 `id` 和一个索引 `a`，字段 `b` 上无索引。存储过程 `idata()` 往表 `t2` 里插入了 `1000` 行数据，在表 `t1` 里插入的是 `100` 行数据。

# Index Nested-Loop Join
先来看一个语句：
```
select * from t1 straight_join t2 on (t1.a=t2.a);
```

这条语句中使用`straight_join`来让`mysql`以固定的连接方式进行查询，让`t1`作为驱动表，`t2`
作为被驱动表。因为如果直接使用`join`，`MySQL` 优化器可能会选择表 `t1` 或 `t2` 作为驱动表，这样会影响该语句的执行过程。

这条语句的`explain`结果为：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_34_1.png">
</center>

可以看到，在这条语句里，被驱动表 `t2` 的字段 `a` 上有索引，`join` 过程用上了这个索引，因此这个语句的执行流程是这样的

- 从表 `t1` 中读入一行数据 `R`；
- 从数据行 `R` 中，取出 `a` 字段到表 `t2` 里去查找；
- 取出表 `t2` 中满足条件的行，跟 `R` 组成一行，作为结果集的一部分；
- 重复执行步骤 `1` 到 `3`，直到表 `t1` 的末尾循环结束。

这个过程是先遍历表 `t1`，然后根据从表 `t1` 中取出的每行数据中的 `a` 值，去表 `t2` 中查找满足条件的记录。在形式上，这个过程就跟嵌套查询类似，并且可以用上被驱动表的索引，所以我们称之为“`Index Nested-Loop Join`”，简称 `NLJ`。

它对应的流程图如下所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_34_2.jpg">
</center>

在这个流程里：

- 对驱动表 `t1` 做了全表扫描，这个过程需要扫描 `100` 行；
- 而对于每一行 `R`，根据 `a` 字段去表 `t2` 查找，走的是树搜索过程。由于前面构造的数据都是一一对应的，因此每次的搜索过程都只扫描一行，也是总共扫描 `100` 行；</p>
- 所以，整个执行流程，总扫描行数是 `200`。

现在，再来看一下文章开头的两个问题：

第一，`join`的性能问题。假设不使用 `join`，用单表查询的过程如下：

- 执行<code>select * from t1</code>，查出表 `t1` 的所有数据，这里有 `100` 行；
- 循环遍历这 100 行数据：
    - 从每一行 R 取出字段 a 的值 `$R.a`；
    - 执行<code>select * from t2 where a=$R.a</code>；
    - 把返回的结果和 R 构成结果集的一行。

可以看到，在这个查询过程，也是扫描了 200 行，但是总共执行了 101 条语句，比直接 join 多了 100 次交互。除此之外，客户端还要自己拼接 SQL 语句和结果。</p><p>显然，这么做还不如直接 join 好。

再来看看第二个问题：<strong>怎么选择驱动表？</strong></p><p>在这个 join 语句执行过程中，驱动表是走全表扫描，而被驱动表是走树搜索。</p><p>假设被驱动表的行数是 M。每次在被驱动表查一行数据，要先搜索索引 a，再搜索主键索引。每次搜索一棵树近似复杂度是以 2 为底的 M 的对数，记为 log<sub>2</sub>M，所以在被驱动表上查一行的时间复杂度是 2*log<sub>2</sub>M。</p><p>假设驱动表的行数是 N，执行过程就要扫描驱动表 N 行，然后对于每一行，到被驱动表上匹配一次。</p><p>因此整个执行过程，近似复杂度是 N + N*2*log<sub>2</sub>M。</p><p>显然，N 对扫描行数的影响更大，因此应该让小表来做驱动表。</p><blockquote>
<p>如果你没觉得这个影响有那么“显然”， 可以这么理解：N 扩大 1000 倍的话，扫描行数就会扩大 1000 倍；而 M 扩大 1000 倍，扫描行数扩大不到 10 倍。</p>
</blockquote><p>到这里小结一下，通过上面的分析我们得到了两个结论：</p><ol>

- 使用 join 语句，性能比强行拆成多个单表执行 SQL 语句的性能要好；
- 如果使用 join 语句的话，需要让小表做驱动表。

当然，这个结论的前提是“**可以使用被驱动表的索引**”。</p><p>接下来再看看被驱动表用不上索引的情况。

# Simple Nested-Loop Join
上面的sql语句如果改为下面这样的：
```
select * from t1 straight_join t2 on (t1.a=t2.b);
```

由于表 t2 的字段 b 上没有索引，因此再用图 2 的执行流程时，每次到 t2 去匹配的时候，就要做一次全表扫描。这样算来，这个 SQL 请求就要扫描表 t2 多达 100 次，总共扫描 100*1000=10 万行。这个算法太笨重了，

mysql没有采用这种算法，而是用了一种叫做“Block Nested-Loop Join”的算法，简称 BNL。

# Block Nested-Loop Join
使用这个算法时，被驱动表上没有索引可用时，算法的流程如下：

- 把表 t1 的数据读入线程内存 join_buffer 中，由于我们这个语句中写的是 select *，因此是把整个表 t1 放入了内存；
- 扫描表 t2，把表 t2 中的每一行取出来，跟 join_buffer 中的数据做对比，满足 join 条件的，作为结果集的一部分返回。

流程图如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_34_3.jpg">
</center>

explain结果如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_34_4.png">
</center>

可以看到，在这个过程中，对表 t1 和 t2 都做了一次全表扫描，因此总的扫描行数是 1100。由于 join_buffer 是以无序数组的方式组织的，因此对表 t2 中的每一行，都要做 100 次判断，总共需要在内存中做的判断次数是：100*1000=10 万次。

前面说，如果使用 Simple Nested-Loop Join 算法进行查询，扫描行数也是 10 万行。因此，从时间复杂度上来说，这两个算法是一样的。但是，Block Nested-Loop Join 算法的这 10 万次判断是**内存操作**，速度上会快很多，性能也更好。

接下来看一下在这种情况下，应该选择哪个表做驱动表。

假设小表的行数是 N，大表的行数是 M，那么在这个算法里：

- 两个表都做一次全表扫描，所以总的扫描行数是 M+N；
- 内存中的判断次数是 M*N。

可以看到，调换这两个算式中的 M 和 N 没差别，**因此这时候选择大表还是小表做驱动表，执行耗时是一样的**。

join_buffer 的大小是由参数 join_buffer_size 设定的，默认值是 256k。<strong>如果放不下表 t1 的所有数据话，策略很简单，就是分段放。</strong>把 join_buffer_size 改成 1200，再执行：
```
select * from t1 straight_join t2 on (t1.a=t2.b);
```
执行过程就变成了：

- 扫描表 t1，顺序读取数据行放入 join_buffer 中，放完第 88 行 join_buffer 满了，继续第 2 步；
- 扫描表 t2，把 t2 中的每一行取出来，跟 join_buffer 中的数据做对比，满足 join 条件的，作为结果集的一部分返回；</p>
- 清空 join_buffer；</p>
- 继续扫描表 t1，顺序读取最后的 12 行数据放入 join_buffer 中，继续执行第 2 步。</p>

执行流程图也就变成这样：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_34_5.jpg">
</center>

图中的步骤 4 和 5，表示清空 join_buffer 再复用。这个流程才体现出了这个算法名字中“**Block**”的由来，表示“**分块去 join**”。

可以看到，这时候由于表 t1 被分成了两次放入 join_buffer 中，导致表 t2 会被扫描两次。虽然分成两次放入 join_buffer，但是判断等值条件的次数还是不变的，依然是 `(88+12)*1000=10 万`次。

那么，在这种情况下驱动表应该怎么选？假设驱动表的数据行数是 N，需要分 K 段才能完成算法流程，被驱动表的数据行数是 M。**注意，这里的 K 不是常数，N 越大 K 就会越大，因此把 K 表示为λ*N，显然λ的取值范围是 (0,1)**。

所以，在这个算法的执行过程中：

- 扫描行数是 N+λ*N*M；
- 内存判断 N*M 次。

显然，内存判断次数是不受选择哪个表作为驱动表影响的。而考虑到扫描行数，在 M 和 N 大小确定的情况下，N 小一些，整个算式的结果会更小。所以结论是，**应该让小表当驱动表**。

当然，在 N+λ*N*M 这个式子里，λ才是影响扫描行数的关键因素，这个值越小越好。而N 固定的时候，join_buffer_size越大，λ越小。join_buffer_size 越大，一次可以放入的行越多，分成的段数也就越少，对被驱动表的全表扫描次数就越少。</p><p>这就是为什么如果 join 语句很慢，把 join_buffer_size 改大一些就有效果。

下面再来看一下文章开头的两个问题。**第一，能不能使用join**。

- **如果可以使用 Index Nested-Loop Join 算法**，也就是说可以用上被驱动表上的索引，**其实是没问题的**；
- **如果使用 Block Nested-Loop Join 算法**，扫描行数就会过多。尤其是在大表上的 join 操作，这样可能要扫描被驱动表很多次，会占用大量的系统资源。**所以这种 join 尽量不要用**。

所以在判断要不要使用 join 语句时，就是看 explain 结果里面**，Extra 字段里面有没有出现“Block Nested Loop”字样**。

**第二，如果要使用 join，应该选择大表做驱动表还是选择小表做驱动表**。

- 如果是 Index Nested-Loop Join 算法，应该选择小表做驱动表；
- 如果是 Block Nested-Loop Join 算法：
    - 在 join_buffer_size 足够大的时候，是一样的；
    - 在 join_buffer_size 不够大的时候（这种情况更常见），应该选择小表做驱动表。

所以，这个问题的结论就是，**总是应该使用小表做驱动表**。

## 什么是小表
先说结论**，在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表**。

比如下面两条语句：
```
select * from t1 straight_join t2 on (t1.b=t2.b) where t2.id<=50;
select * from t2 straight_join t1 on (t1.b=t2.b) where t2.id<=50;
```
这两条语句都没有用到索引，但是根据where语句筛选之后，t2表参与运算的只有50行，所以，如果使用第二条语句，join_buffer中只需要放入t2的前50行数据，而使用第一行数据，join_buffer中就需要放入t1的100行数据，所以，这里的t2就是“小表”。

再来看一个例子：
```
select t1.b,t2.* from  t1  straight_join t2 on (t1.b=t2.b) where t2.id<=100;
select t1.b,t2.* from  t2  straight_join t1 on (t1.b=t2.b) where t2.id<=100;
```
在这个例子中的两条语句中，t1和t2都只有100行数据参加运算，但是这两条语句在执行时放入join_buffer的数据个数还是不一样的：

- 表t1只查询字段b，所以如果把t1放入join_buffer中，只需要放入b字段；
- 表t2要查询全部数据，所以如果把t2放入join_buffer中，需要放入id、a和b三个字段。

# 小结

- 如果可以使用被驱动表的索引，join 语句还是有其优势的；
- 不能使用被驱动表的索引，只能使用 Block Nested-Loop Join 算法，这样的语句就尽量不要使用；
- 在使用 join 的时候，应该让小表做驱动表。

[【性能提升神器】STRAIGHT_JOIN](https://www.cnblogs.com/heyonggang/p/9462242.html)





# 16 | “order by”是怎么工作的？
---

先来看一个例子，假如一个市民表的定义如下：
```
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```
现在要查询城市是“杭州”的所有人名字，并且按照姓名排序返回前 1000 个人的姓名、年龄。可以使用如下语句：
```
select city,name,age from t where city='杭州' order by name limit 1000  ;
```
下面就分析一下这个语句是怎么执行的。

# 全字段排序
为避免全表扫描，肯定需要在 `city` 字段加上索引。使用`explaini`语句来观察上述语句的执行情况：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_1.png">
</center>

`Extra` 这个字段中的`“Using filesort”`表示的就是需要排序，`MySQL` 会给每个线程分配一块内存用于排序，称为 `sort_buffer`。

`city`索引的示意图如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_2.png">
</center>

可以看到，满足 `city='杭州’`条件的行，是从 `ID_X` 到 `ID_(X+N)` 的这些记录。

通常情况下，这个语句执行流程如下所示 ：

- 初始化 `sort_buffer`，确定放入 `name、city、age` 这三个字段；
- 从索引 `city` 找到第一个满足 `city='杭州’`条件的主键 `id`，也就是图中的 `ID_X`；
- 到主键 `id` 索引取出整行，取 `name、city、age` 三个字段的值，存入 `sort_buffer` 中；
- 从索引 `city` 取下一个记录的主键 id；
- 重复步骤 `3、4` 直到 `city` 的值不满足查询条件为止，对应的主键 `id` 也就是图中的 `ID_Y`；
- 对 `sort_buffer` 中的数据按照字段 `name` 做快速排序；
- 按照排序结果取前 `1000` 行返回给客户端。

上面这个排序过程zanqie称为全字段排序，执行流程的示意图如下所示
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_3.jpg">
</center>

`“按 name 排序”`这个动作，可能在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和参数 `sort_buffer_size`。

`sort_buffer_size`，就是 `MySQL` 为排序开辟的内存（`sort_buffer`）的大小。如果要排序的数据量小于 `sort_buffer_size`，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。可以用下面的方法来确定一个排序语句是否使用了临时文件。
```
/* 打开 optimizer_trace，只对本线程有效 */
SET optimizer_trace='enabled=on'; 

/* @a 保存 Innodb_rows_read 的初始值 */
select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 执行语句 */
select city, name,age from t where city='杭州' order by name limit 1000; 

/* 查看 OPTIMIZER_TRACE 输出 */
SELECT * FROM `information_schema`.`OPTIMIZER_TRACE`\G

/* @b 保存 Innodb_rows_read 的当前值 */
select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 计算 Innodb_rows_read 差值 */
select @b-@a;
```

这个方法是通过查看 `OPTIMIZER_TRACE` 的结果来确认的，可以从 `number_of_tmp_files` 中看到是否使用了临时文件。如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_4.png">
</center>

`number_of_tmp_files` 表示的是排序过程中使用的临时文件数。当内存放不下时，就需要使用外部排序，外部排序一般使用**归并排序**算法，所以可能要使用多个临时文件，这里就是`12`个。如果 `sort_buffer_size` 超过了需要排序的数据量的大小，`number_of_tmp_files` 就是 `0`，表示排序可以直接在内存中完成。否则就需要放在临时文件中排序。`sort_buffer_size` 越小，需要分成的份数越多，`number_of_tmp_files` 的值就越大。

上图中，`examined_rows=4000`表示参与排序的行数是 `4000` 行。`sort_mode` 里面的 `packed_additional_fields` 的意思是，排序过程对字符串做了“紧凑”处理。即使 `name` 字段的定义是 `varchar(16)`，在排序过程中还是要按照实际长度来分配空间的。最后一个查询语句 `select @b-@a` 的返回结果是 `4000`，表示整个执行过程只扫描了 `4000` 行。

# rowid 排序
在上面这个算法过程里面，只对原表的数据读了一遍，剩下的操作都是在 `sort_buffer` 和临时文件中执行的。但这个算法有一个问题，就是如果查询要返回的字段很多的话，那么 `sort_buffer` 里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。所以如果单行很大，这个方法效率不够好。

Mysql中的`max_length_for_sort_data`参数是专门控制用于排序的行数据的长度的一个参数。它的意思是，**如果单行的长度超过这个值，`MySQL` 就认为单行太大，要换一个算法**。

`city、name、age` 这三个字段的定义总长度是 `36`，可以通过下面的命令把 `max_length_for_sort_data` 设置为 `16`。
```
SET max_length_for_sort_data = 16;
```

这样更改后，`sort_buffer` 的字段只有要排序的列（即 `name` 字段）和主键 `id`。但是，排序的结果就因为少了 `city` 和 `age` 字段的值，不能直接返回了，整个执行流程就变成如下所示的样子：

- 初始化 `sort_buffer`，确定放入两个字段，即 `name` 和 `id`；
- 从索引 `city` 找到第一个满足 `city='杭州’`条件的主键 `id`，也就是图中的 `ID_X`；
- 到主键 `id` 索引取出整行，取 `name、id` 这两个字段，存入 `sort_buffer` 中；
- 从索引 `city` 取下一个记录的主键 `id`；
- 重复步骤 `3、4` 直到不满足 `city='杭州’`条件为止，也就是图中的 `ID_Y`；
- 对 `sort_buffer` 中的数据按照字段 `name` 进行排序；
- 遍历排序结果，取前 `1000` 行，并按照 `id` 的值回到原表中取出 `city、name 和 age` 三个字段返回给客户端。

这个执行流程的示意图如下，可以暂且把它称为 `rowid` 排序。
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_5.jpg">
</center>

这时，查看 `OPTIMIZER_TRACE`的结果，如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_6.png">
</center>

和全排序算法相比，有几个变化：

- `sort_mode` 变成了 `<ort_key, rowid>;`，表示参与排序的只有 `name` 和 `id` 这两个字段。
- `number_of_tmp_files` 变成 `10` 了，是因为这时候参与排序的行数虽然仍然是 `4000` 行，但是每一行都变小了，因此需要排序的总数据量就变小了，需要的临时文件也相应地变少了。

另外，`select @b-@a` 这个语句的值变成 `5000` 了。因为这时候除了排序过程外，在排序完成后，还要根据 `id` 去原表取值。由于语句是 `limit 1000`，因此会多读 `1000` 行。

# 全字段排序 VS rowid 排序
这两种排序算法的区别如下：

- 如果 `MySQL` 实在是担心排序内存太小，会影响排序效率，才会采用 `rowid` 排序算法，这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据。
- 如果 `MySQL` 认为内存足够大，会优先选择全字段排序，把需要的字段都放到 `sort_buffer` 中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。这也就体现了 MySQL 的一个设计思想：**如果内存够，就要多利用内存，尽量减少磁盘访问。**对于 `InnoDB` 表来说，`rowid` 排序会要求回表多造成磁盘读，因此不会被优先选择。

# 是否所有的order by都需要排序
`MySQL` 之所以需要生成临时表，并且在临时表上做排序操作，**其原因是原来的数据都是无序的**。对上面的例子来说，如果能够保证从 `city` 这个索引上取出来的行，天然就是按照 `name` 递增排序的话，就不用排序。

所以，可以在这个市民表上创建一个 `city` 和 `name` 的联合索引，对应的 `SQL` 语句是：
```
alter table t add index city_user(city, name);
```

这个索引的示意图如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_7.png">
</center>

这时整个查询过程的流程就变成了

- 从索引 (`city,name`) 找到第一个满足 `city='杭州’`条件的主键 `id`；
- 到主键 `id` 索引取出整行，取 `name、city、age` 三个字段的值，作为结果集的一部分直接返回；
- 从索引 (`city,name`) 取下一个记录主键 `id`；
- 重复步骤 `2、3`，直到查到第 `1000` 条记录，或者是不满足 `city='杭州’`条件时循环结束。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_8.jpg">
</center>

实际上，这个查询还可以使用覆盖索引进行优化。针对这个查询，可以创建一个 `city、name` 和 `age` 的联合索引，对应的 `SQL` 语句就是：
```
alter table t add index city_user_age(city, name, age);
```

这样整个查询语句的执行流程就变成了：

- 从索引 (`city,name,age`) 找到第一个满足 `city='杭州’`条件的记录，取出其中的 `city、name` 和 `age` 这三个字段的值，作为结果集的一部分直接返回；
- 从索引 (`city,name,age`) 取下一个记录，同样取出这三个字段的值，作为结果集的一部分直接返回；
- 重复执行步骤 `2`，直到查到第 `1000` 条记录，或者是不满足 `city='杭州’`条件时循环结束。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql_45_16_9.jpg">
</center>

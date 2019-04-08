# 12 | 为什么我的MySQL会“抖”一下？
---

一条`sql`语句正常执行的时候很快，但是有时候突然会变得特别慢，而且持续时间比较短，就好像“抖”了一下一样。

这种现象出现的原因很可能是因为在“刷脏”。

# 你的SQL语句为什么变“慢”了
MySQL 在更新数据的时候，都是将数据先从磁盘拉到 buffer pool 中，在buffer pool中修改完成后并将改变写入redo log，之后再写到磁盘中。

当数据在buffer pool中更改完成的这一刻，更新后的数据是“最新”的，因为此时磁盘中的数据还是更改前的“旧数据”，而我们都是将磁盘中已经持久化的数据作为“标准数据”，因此此时 buffer pool 中的“最新”数据也常人们被称为“脏数据（dirty data）”。

所以，当平时执行很快的更新操作，其实就是在写内存和日志，而 MySQL 偶尔“抖”一下的那个瞬间，可能就是在刷脏页（flush）。</p><p>那么，什么情况会引发数据库的 flush 过程呢？

- 第一种场景是 InnoDB 的 redo log 写满了。这时候系统会停止所有更新操作，把 checkpoint 往前推进，redo log 留出空间可以继续写。redo log的示意图如下，当把 checkpoint 位置从 CP 推进到 CP’，就需要将两个点之间的日志（浅绿色部分），对应的所有脏页都 flush 到磁盘上。之后，图中从 write pos 到 CP’之间就是可以再写入的 redo log 的区域。

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql45_12_1.jpg">
</center>

- 第二种情况是系统内存不足。当需要新的内存页，而内存不够用的时候，就要淘汰一些数据页，空出内存给别的数据页使用。如果淘汰的是“脏页”，就要先将脏页写到磁盘。

- 第三种情况是 MySQL 认为系统“空闲”的时候。
- 第四种情况是 MySQL 正常关闭的情况。这时候，MySQL 会把内存的脏页都 flush 到磁盘上，这样下次 MySQL 启动的时候，就可以直接从磁盘上读数据，启动速度会很快。

下面分析一下这四种情况对性能的影响。其中，第三种情况是属于 MySQL 空闲时的操作，这时系统没什么压力，而第四种场景是数据库本来就要关闭了。这两种情况下，你不会太关注“性能”问题。

第一种是“redo log 写满了，要 flush 脏页”，这种情况是 InnoDB 要**尽量避免**的。因为出现这种情况的时候，整个系统就不能再接受更新了，所有的更新都必须堵住。如果从监控上看，这时候更新数会跌为 0。

第二种是“内存不够用了，要先将脏页写到磁盘”，这种情况其实是常态。<strong>InnoDB 用缓冲池（buffer pool）管理内存，缓冲池中的内存页有三种状态：</strong>

- 第一种是，还没有使用的；
- 第二种是，使用了并且是干净页；
- 第三种是，使用了并且是脏页。

InnoDB 的策略是尽量使用内存，因此对于一个长时间运行的库来说，未被使用的页面很少。</p><p>而当要读入的数据页没有在内存的时候，就必须到缓冲池中申请一个数据页。这时候只能把最久不使用的数据页从内存中淘汰掉：如果要淘汰的是一个干净页，就直接释放出来复用；但如果是脏页呢，就必须将脏页先刷到磁盘，变成干净页后才能复用。

所以，刷脏页虽然是常态，但是出现以下这两种情况，都是会明显影响性能的：

- 一个查询要淘汰的脏页个数太多，会导致查询的响应时间明显变长；</p>
- 日志写满，更新全部堵住，写性能跌为 0，这种情况对敏感业务来说，是不能接受的。</p>

所以，InnoDB 需要有控制脏页比例的机制，来尽量避免上面的这两种情况。

# InnoDB 刷脏页的控制策略
首先是innodb_io_capacity 这个参数，它会告诉 InnoDB 主机的磁盘能力。这个值建议你设置成磁盘的 IOPS。磁盘的 IOPS 可以通过 fio 这个工具来测试，下面的语句是用来测试磁盘随机读写的命令(处理fio工具外，还有iometer和Orion可以测试I/O能力)：
```
fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest 
```

InnoDB 能够控制引擎按照“全力”的百分比来刷脏页。因为如果刷脏比较慢的情况下，主要有两个影响：

- 内存脏页太多
- redo log 写满

所以，InnoDB控制刷脏速率需要考虑的问题有两个：

- 脏页比例
- redo log 写盘速度

InnoDB 会根据这两个因素先单独算出两个数字。</p><p>参数 innodb_max_dirty_pages_pct 是脏页比例上限，默认值是 75%。InnoDB 会根据当前的**脏页比例（假设为 M）**，算出一个范围在 0 到 100 之间的数字，计算这个数字的伪代码类似这样：
```
F1(M)
{
  if M>=innodb_max_dirty_pages_pct then
      return 100;
  return 100*M/innodb_max_dirty_pages_pct;
}
```
InnoDB 每次写入的日志都有一个序号，当前写入的**序号跟 checkpoint 对应的序号之间的差值假设为 N**。InnoDB 会根据这个 N 算出一个范围在 0 到 100 之间的数字，这个计算公式可以记为 F2(N)。N 越大，算出来的值越大。

然后，**根据上述算得的 F1(M) 和 F2(N) 两个值，取其中较大的值记为 R，之后引擎就可以按照 innodb_io_capacity 定义的能力乘以 R% 来控制刷脏页的速度**。

上述的流程如下图所示，图中的 F1、F2 就是上面通过脏页比例和 redo log 写入速度算出来的两个值。：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mysql45_12_2.png">
</center>

所以，无论是查询语句在需要内存的时候可能要求淘汰一个脏页，还是由于刷脏页的逻辑会占用 IO 资源并可能影响到更新语句，都可能是造成MySQL“抖”了一下的原因。

要尽量避免这种情况，就要合理地设置 innodb_io_capacity 的值，并且<strong>平时要多关注脏页比例，不要让它经常接近 75%</strong>。

其中，脏页比例是通过 Innodb_buffer_pool_pages_dirty/Innodb_buffer_pool_pages_total 得到的，具体的命令参考下面的代码(这里存在一个问题，在mysql5.7执行下面代码时，会提示没有global_status这个表。但是show global status可以用)：
```
mysql> select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;
```

InnoDB中还有一个“连坐”机制：在准备刷一个脏页的时候，如果这个数据页旁边的数据页刚好是脏页，就会把这个“邻居”也带着一起刷掉；而且这个把“邻居”拖下水的逻辑还可以继续蔓延，也就是对于每个邻居数据页，如果跟它相邻的数据页也还是脏页的话，也会被放到一起刷。

在 InnoDB 中，innodb_flush_neighbors 参数就是用来控制这个行为的，值为 1 的时候会有上述的“连坐”机制，值为 0 时表示不找邻居，自己刷自己的。

找“邻居”这个优化在机械硬盘时代是很有意义的，可以减少很多随机 IO。机械硬盘的随机 IOPS 一般只有几百，相同的逻辑操作减少随机 IO 就意味着系统性能的大幅度提升。

而如果使用的是 SSD 这类 IOPS 比较高的设备的话，建议把 innodb_flush_neighbors 的值设置成 0。因为这时候 IOPS 往往不是瓶颈，而“只刷自己”，就能更快地执行完必要的刷脏页操作，减少 SQL 语句响应时间。

在 MySQL 8.0 中，innodb_flush_neighbors 参数的默认值已经是 0 了。

# 参考
[【MySQL】redo log --- 刷入磁盘过程](https://www.cnblogs.com/haohaozhang/p/10093182.html)


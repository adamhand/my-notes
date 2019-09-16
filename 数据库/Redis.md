# Redis
---

# 关系型数据库和非关系型数据库
## 关系型数据库
关系型数据库最典型的数据结构是表，由二维表及其之间的联系所组成的一个数据组织。当前主流的关系型数据库有Oracle、DB2、Microsoft SQL Server、Microsoft Access、MySQL等。

### 优点

- 易于维护：都是使用表结构，格式一致；
- 事务性： 保持数据的一致性；
- 复杂操作：支持SQL，可进行非常复杂的查询；

### 缺点
- 读写性能比较差，尤其是海量数据的高效率读写，因为关系型数据库将数据存储在硬盘中，硬盘I/O是一个很大的瓶颈；

## 非关系型数据库
菲关系型数据库，NoSQL(NoSQL = Not Only SQL )，意即"不仅仅是SQL"。常见的非关系型数据库有MongoDB、CouchDb、LevelDB和redis。

非关系型数据库严格上不是一种数据库，应该是一种数据结构化存储方法的集合，可以是文档或者键值对等。

### 优点

- 格式灵活：存储数据的格式可以是key,value形式、文档形式、图片形式等等，文档形式、图片形式等等，使用灵活，应用场景广泛，而关系型数据库则只支持基础类型。
- 速度快：nosql数据库将数据存储于缓存之中，速度快。
- 成本低：nosql数据库部署简单，基本都是开源软件。
- 可扩展性高：可扩展性同样也是因为基于键值对，数据之间没有耦合性，所以非常容易水平扩展。

### 缺点

- 不提供sql支持，学习和使用成本较高；
- 非关系数据库不提供强大的事务关系，没有保证数据的完整性和安全性

### 典型的NoSQL数据库
典型的NoSQL数据库有三种：**文档型**、**key-value型**和**列式数据库**。

**(1)文档型**
MongoDB、CouchDB属于这种类型，它们属于NoSQL数据库，但与键值存储相异。

主要有以下特点：

- 不用预先定义表结构。但是可以像定义了表结构一样使用，还省去了变更表结构的麻烦。
- 可以使用复杂的查询条件。

**(2)Key-Value型**
它的数据是以键值的形式存储的，虽然它的速度非常快，但基本上只能通过键的完全一致查询获取数据，根据数据的保存方式可以分为**临时性**、**永久性**和**两者兼具**三种。

- **临时性**：谓临时性就是数据有可能丢失，memcached把所有数据都保存在内存中，这样保存和读取的速度非常快，但是当memcached停止时，数据就不存在了。由于数据保存在内存中，所以无法操作超出内存容量的数据，旧数据会丢失。
- **永久性**：所谓永久性就是数据不会丢失，这里的键值存储是把数据保存在硬盘上，与临时性比起来，由于必然要发生对硬盘的IO操作，所以性能上还是有差距的，但数据不会丢失是它最大的优势。
- **两者兼备**：Redis属于这种类型。Redis有些特殊，临时性和永久性兼具。Redis首先把数据保存在内存中，在满足特定条件（默认是 15分钟一次以上，5分钟内10个以上，1分钟内10000个以上的键发生变更）的时候将数据写入到硬盘中，这样既确保了内存中数据的处理速度，又可以通过写入硬盘来保证数据的永久性，这种类型的数据库特别适合处理数组类型的数据。

**(3)面向列的数据库**
普通的关系型数据库都是以行为单位来存储数据的，擅长以行为单位的读入处理。相反，面向列的数据库是以列为单位来存储数据的，擅长以列为单位读入数据。

面向列的数据库扩展性强，更容易进行分布式扩展。

---
参考：
[关系型数据库与非关系型数据库的区别？](https://www.cnblogs.com/jesssey/p/7750911.html)</br>
[常见的关系型数据库和非关系型数据及其区别](https://blog.csdn.net/aaronthon/article/details/81714528)</br>
[关系型数据库和非关系数据库区别](https://blog.csdn.net/qq_31536117/article/details/78179646)</br>
[关系型和非关系型数据库的区别?](https://blog.csdn.net/longxingzhiwen/article/details/53896702)</br>

---

# Redis简述
Redis 是速度非常快的**非关系型**（NoSQL）**内存键值**数据库，可以存储键和五种不同类型的值之间的映射。

键的类型只能为字符串，值支持五种数据类型：**字符串**、**列表**、**集合**、**散列表**、**有序集合**。

Redis 支持很多特性，例如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。

# 数据类型
|数据类型|	可以存储的值|	操作|
|-|-|-|
|STRING|	字符串、整数或者浮点数|对整个字符串或者字符串的其中一部分执行操作对整数和浮点数执行自增或者自减操作|
|LIST|	列表|	从两端压入或者弹出元素；对单个或者多个元素进行修剪，只保留一个范围内的元素|
|SET	|无序集合|	添加、获取、移除单个元素；检查一个元素是否存在于集合中；计算交集、并集、差集;从集合里面随机获取元素|
|HASH|	包含键值对的无序散列表|添加、获取、移除单个键值对；获取所有键值对；检查某个键是否存在|
|ZSET|	有序集合|	添加、获取、删除元素；根据分值范围或者成员来获取元素；计算一个键的排名|

## STRING
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/6019b2db-bc3e-4408-b6d8-96025f4481d6.png" width="450">
</div >

```redis
> set hello world
OK
> get hello
"world"
> del hello
(integer) 1
> get hello
(nil)
```

## LIST
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/fb327611-7e2b-4f2f-9f5b-38592d408f07.png" width="450">
</div >

```redis
> rpush list-key item
(integer) 1
> rpush list-key item2
(integer) 2
> rpush list-key item
(integer) 3

> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"

> lindex list-key 1
"item2"

> lpop list-key
"item"

> lrange list-key 0 -1
1) "item2"
2) "item"
```

## SET
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/cd5fbcff-3f35-43a6-8ffa-082a93ce0f0e.png" width="450">
</div >

```redis
> sadd set-key item
(integer) 1
> sadd set-key item2
(integer) 1
> sadd set-key item3
(integer) 1
> sadd set-key item
(integer) 0

> smembers set-key
1) "item"
2) "item2"
3) "item3"

> sismember set-key item4
(integer) 0
> sismember set-key item
(integer) 1

> srem set-key item2
(integer) 1
> srem set-key item2
(integer) 0

> smembers set-key
1) "item"
2) "item3"
```

## HASH
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/7bd202a7-93d4-4f3a-a878-af68ae25539a.png" width="450">
</div >

```redis
> hset hash-key sub-key1 value1
(integer) 1
> hset hash-key sub-key2 value2
(integer) 1
> hset hash-key sub-key1 value1
(integer) 0

> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"

> hdel hash-key sub-key2
(integer) 1
> hdel hash-key sub-key2
(integer) 0

> hget hash-key sub-key1
"value1"

> hgetall hash-key
1) "sub-key1"
2) "value1"
```

## ZSET
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/1202b2d6-9469-4251-bd47-ca6034fb6116.png" width="450">
</div >

```redis
> zadd zset-key 728 member1
(integer) 1
> zadd zset-key 982 member0
(integer) 1
> zadd zset-key 982 member0
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"

> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"

> zrem zset-key member1
(integer) 1
> zrem zset-key member1
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```

# 底层数据结构
以上介绍了redis的五种数据类型，这五种数据类型由底层的六大数据结构实现，每一种数据类型都至少用到了一种数据结构。

## 简单动态字符串(simple dynamic string,SDS)
Redis 是用 C 语言写的，但是对于Redis的字符串，却不是 C 语言中的字符串（即以空字符’\0’结尾的字符数组），它是自己构建了一种名为 简单动态字符串（simple dynamic string,SDS）的抽象类型，并将 SDS 作为 Redis的默认字符串表示。

### SDS定义
```c
struct sdshdr{
     //记录buf数组中已使用字节的数量；等于 SDS 保存字符串的长度
     int len;
     //记录 buf 数组中未使用字节的数量
     int free;
     //字节数组，用于保存字符串
     char buf[];
}
```
用SDS保存字符串 “Redis”具体图示如下：
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/sds.PNG">
</div >

### 使用SDS的优势
① 常数复杂度获取字符串长度

由于 len 属性的存在，我们获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 O(1)。而对于 C 语言，获取字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。通过 strlen key 命令可以获取 key 的字符串长度。

② 杜绝缓冲区溢出

我们知道在 C 语言中使用 strcat  函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。而对于 SDS 数据类型，在进行字符修改的时候，会首先根据记录的 len 属性检查内存空间是否满足需求，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。

③ 减少修改字符串的内存重新分配次数

C语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。

而对于SDS，由于len属性和free属性的存在，对于修改字符串SDS实现了空间预分配和惰性空间释放两种策略：

- 空间预分配：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
- 惰性空间释放：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 free 属性将这些字节的数量记录下来，等待后续使用。（当然SDS也提供了相应的API，当我们有需要时，也可以手动释放这些未使用的空间。）

④ 二进制安全

因为C字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此C字符串无法正确存取；而所有 SDS 的API 都是以处理二进制的方式来处理 buf 里面的元素，并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束。

⑤ 兼容部分 C 字符串函数

虽然 SDS 是二进制安全的，但是一样遵从每个字符串都是以空字符串结尾的惯例，这样可以重用 C 语言库`<string.h>` 中的一部分函数。

## 双链表
链表是一种常用的数据结构，C 语言内部是没有内置这种数据结构的实现，所以Redis自己构建了链表的实现，实现了自己的双链表结构。

链表节点的定义：
```c
typedef  struct listNode{
       //前置节点
       struct listNode *prev;
       //后置节点
       struct listNode *next;
       //节点的值
       void *value;  
}listNode
```
链表定义：
```c
typedef struct list{
     //表头节点
     listNode *head;
     //表尾节点
     listNode *tail;
     //链表所包含的节点数量
     unsigned long len;
     //节点值复制函数
     void (*free) (void *ptr);
     //节点值释放函数
     void (*free) (void *ptr);
     //节点值对比函数
     int (*match) (void *ptr,void *key);
}list;
```
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/doublelist.PNG">
</div >

Redis链表特性：

① 双端：链表具有前置节点和后置节点的引用，获取这两个节点时间复杂度都为O(1)。
② 无环：表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL,对链表的访问都是以 NULL 结束。　　
③ 带链表长度计数器：通过 len 属性获取链表长度的时间复杂度为 O(1)。
④ 多态：链表节点使用 void* 指针来保存节点值，可以保存各种不同类型的值。

## 字典
字典又称为符号表（symble table）或者关联数组（associative array）、或映射（map），是一种用于保存键值对的抽象数据结构。字典中的每一个键 key 都是唯一的，通过 key 可以对值来进行查找或修改。C 语言中没有内置这种数据结构的实现，所以字典依然是 Redis自己构建的。

字典在Redis中应用十分广泛，比如Redis的数据库就是使用字典作为底层实现的，对数据库的增删改查也是建立在对字典的操作之上；另外，字典还是哈希键的底层实现之一。

Redis的字典使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点就保存了字典中的一个键值对。

哈希表结构定义：
```c
typedef struct dictht{
     //哈希表数组
     dictEntry **table;
     //哈希表大小
     unsigned long size;
     //哈希表大小掩码，用于计算索引值
     //总是等于 size-1
     unsigned long sizemask;
     //该哈希表已有节点的数量
     unsigned long used;
 
}dictht
```
哈希表是由数组 table 组成，table 中每个元素都是指向 dict.h/dictEntry 结构，每个dictEntry存储着一个键值对信息。dictEntry 结构定义如下：
```c
typedef struct dictEntry{
     //键
     void *key;
     //值
     union{
          void *val;
          uint64_tu64;
          int64_ts64;
     }v;
 
     //指向下一个哈希表节点，形成链表
     struct dictEntry *next;
}dictEntry
```
key 用来保存键，val 属性用来保存值，值可以是一个指针，也可以是uint64_t整数，也可以是int64_t整数。

该哈希表使用**链地址法**来解决哈希冲突，这里的next指针指向下一个哈希值相同的节点，形成一条链表。如下图所示：
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/dictht.PNG">
</div >

### 扩容和收缩
Redis 的字典 dict 中包含两个哈希表 dictht，这是为了方便进行 rehash 操作。**在扩容时**，根据原哈希表已使用的空间扩大一倍创建另一个哈希表，然后将原 dictht 上的键值对 rehash 到新建的 dictht 上面，完成之后释放空间并交换两个 dictht 的角色；**收缩时类似**，只不过时根据已使用空间缩小一倍创建一个新的哈希表。
```c
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

### 渐进式rehash
渐进式rehash是指扩容和收缩操作不是一次性、集中式完成的，而是分多次、渐进式完成的。如果保存在Redis中的键值对只有几个几十个，那么 rehash 操作可以瞬间完成，但是如果键值对有几百万，几千万甚至几亿，那么要一次性的进行 rehash，势必会造成Redis一段时间内不能进行别的操作，并且给服务器造成很大的负担。所以Redis采用渐进式 rehash,这样在进行渐进式rehash期间，字典的删除查找更新等操作可能会在两个哈希表上进行，第一个哈希表没有找到，就会去第二个哈希表上进行查找。但是进行 增加操作，一定是在新的哈希表上进行的。

渐进式 rehash 通过记录 dict 的 rehashidx 完成，它从 0 开始，然后每执行一次 rehash 都会递增。例如在一次 rehash 中，要把 dict[0] rehash 到 dict[1]，这一次会把 dict[0] 上 table[rehashidx] 的键值对 rehash 到 dict[1] 上，dict[0] 的 table[rehashidx] 指向 null，并令 rehashidx++。

在 rehash 期间，每次对字典执行添加、删除、查找或者更新操作时，都会执行一次渐进式 rehash。
```c
int dictRehash(dict *d, int n) {
    int empty_visits = n * 10; /* Max number of empty buckets to visit. */
    if (!dictIsRehashing(d)) return 0;

    while (n-- && d->ht[0].used != 0) {
        dictEntry *de, *nextde;

        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned long) d->rehashidx);
        while (d->ht[0].table[d->rehashidx] == NULL) {
            d->rehashidx++;
            if (--empty_visits == 0) return 1;
        }
        de = d->ht[0].table[d->rehashidx];
        /* Move all the keys in this bucket from the old to the new hash HT */
        while (de) {
            uint64_t h;

            nextde = de->next;
            /* Get the index in the new hash table */
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;
            d->ht[0].used--;
            d->ht[1].used++;
            de = nextde;
        }
        d->ht[0].table[d->rehashidx] = NULL;
        d->rehashidx++;
    }

    /* Check if we already rehashed the whole table... */
    if (d->ht[0].used == 0) {
        zfree(d->ht[0].table);
        d->ht[0] = d->ht[1];
        _dictReset(&d->ht[1]);
        d->rehashidx = -1;
        return 0;
    }

    /* More to rehash... */
    return 1;
}
```

## 跳跃表
是有序集合的底层实现之一。

跳跃表是基于多指针有序链表实现的，可以看成多个有序链表。具有如下性质：
① 由很多层结构组成；
② 每一层都是一个有序的链表，排列顺序为由高层到底层，都至少包含两个链表节点，分别是前面的head节点和后面的nil节点；
③ 最底层的链表包含了所有的元素；
④ 如果一个元素出现在某一层的链表中，那么在该层之下的链表也全都会出现（上一层的元素是当前层的元素的子集）；
⑤ 链表中的每个节点都包含两个指针，一个指向同一层的下一个链表节点，另一个指向下一层的同一个链表节点；
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jmplist_1.png">
</div >


Redis中跳跃表节点定义如下：
```c
typedef struct zskiplistNode {
     //层
     struct zskiplistLevel{
           //前进指针
           struct zskiplistNode *forward;
           //跨度
           unsigned int span;
     }level[];
 
     //后退指针
     struct zskiplistNode *backward;
     //分值
     double score;
     //成员对象
     robj *obj;
 
} zskiplistNode
```
多个跳跃表节点构成一个跳跃表：
```c
typedef struct zskiplist{
     //表头节点和表尾节点
     structz skiplistNode *header, *tail;
     //表中节点的数量
     unsigned long length;
     //表中层数最大的节点的层数
     int level;
 
}zskiplist;
```

① 搜索：从最高层的链表节点开始，如果比当前节点要大和比当前层的下一个节点要小，那么则往下找，也就是和当前层的下一层的节点的下一个节点进行比较，以此类推，一直找到最底层的最后一个节点，如果找到则返回，反之则返回空。
② 插入：插入操作有两种方法：

- **方法一：**使用抛硬币法确定插入的层数，如果是正面就累加，直到遇见反面为止，最后记录正面的次数作为插入的层数。当确定插入的层数k后，则需要将新元素插入到从底层到k层。**复杂度是O(logn)**。
- **方法二：**将要插入的节点和各层索引节点逐一比较，确定原链表的插入位置（O（logN））；然后把索引插入到原链表（O（1））；利用抛硬币的随机方式，决定新节点是否提升为上一级索引。结果为“正”则提升并继续抛硬币，结果为“负”则停止（O（logN））。总体上，跳跃表**插入操作的时间复杂度是O（logN）**，而这种数据结构所占空间是2N，**既空间复杂度是 O（N）**。

③ 删除：在各个层中找到包含指定值的节点，然后将节点从链表中删除即可，如果删除以后只剩下头尾两个节点，则删除这一层。删除的时间复杂度就是查询元素插入位置的时间复杂度，**所以是O(logn)**。

在上述跳跃表中寻找22的过程如下图所示：
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jumplist_2.png">
</div >

与红黑树等平衡树相比，跳跃表具有以下优点：

- 插入速度非常快速，因为不需要进行旋转等操作来维护平衡性；
- 更容易实现；
- 支持无锁操作。

## 整数集合
整数集合（intset）是Redis用于保存整数值的集合抽象数据类型，它可以保存类型为int16_t、int32_t 或者int64_t 的整数值，并且保证集合中不会出现重复元素。

定义如下：
```c
typedef struct intset{
     //编码方式
     uint32_t encoding;
     //集合包含的元素数量
     uint32_t length;
     //保存元素的数组
     int8_t contents[];
 
}intset;
```
整数集合的每个元素都是 contents 数组的一个数据项，它们按照从小到大的顺序排列，并且不包含任何重复项。

length 属性记录了 contents 数组的大小。

需要注意的是虽然 contents 数组声明为 int8_t 类型，但是实际上contents 数组并不保存任何 int8_t 类型的值，其真正类型有 encoding 来决定。

① 升级
当我们新增的元素类型比原集合元素类型的长度要大时，需要对整数集合进行升级，才能将新元素放入整数集合中。具体步骤：

- 根据新元素类型，扩展整数集合底层数组的大小，并为新元素分配空间。
- 将底层数组现有的所有元素都转成与新元素相同类型的元素，并将转换后的元素放到正确的位置，放置过程中，维持整个元素顺序都是有序的。
- 将新元素添加到整数集合中（保证有序）。

升级能极大地节省内存。

② 降级

整数集合不支持降级操作，一旦对数组进行了升级，编码就会一直保持升级后的状态

## 压缩列表
压缩列表（ziplist）是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

**压缩列表的原理：压缩列表并不是对数据利用某种算法进行压缩，而是将数据按照一定规则编码在一块连续的内存区域，目的是节省内存。**

# 缓存淘汰策略
Redis 可以为每个键设置过期时间，当键过期时，会自动删除该键。可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

Reids 具体有 6 种淘汰策略：
|策略|描述|
|-|-|
|volatile-lru|从已设置过期时间的数据集中挑选最近最少使用的数据淘汰|
|volatile-ttl|从已设置过期时间的数据集中挑选将要过期的数据淘汰|
|volatile-random|从已设置过期时间的数据集中任意选择数据淘汰|
|allkeys-lru|从所有数据集中挑选最近最少使用的数据淘汰|
|allkeys-random|从所有数据集中任意选择数据进行淘汰|
|noeviction|禁止驱逐数据|

# 持久化
持久化是将内存中的数据持久化到硬盘上，目的方式数据在断电后丢失。Redis提供两种持久化策略：RDB持久化和AOF持久化。**RDB持久化是将数据保存到硬盘，AOF持久化是将每次执行的写命令保存到硬盘**。Redis默认开启RDB，关闭AOF。

## RDB持久化
将某个时间点的所有数据保存到硬盘上的.rdb文件中，适合全量复制，但是实时性不高，可能会丢数据，而且如果数据量很大，保存的时间会很长。

RDB持久化的触发分为两种：**手动触发**与**Redis定时触发**。

**手动触发可以使用**：

- save：会阻塞当前Redis服务器，直到持久化完成，线上应该禁止使用。
- bgsave：该触发方式会fork一个子进程，由子进程负责持久化过程，因此阻塞只会发生在fork子进程的时候。

**自动触发的场景主要有**：

- 根据 save m n 配置规则自动触发；
- 从节点全量复制时，主节点发送rdb文件给从节点完成复制操作，主节点会触发 bgsave；
- 执行 debug reload 时；
- 执行 shutdown时，如果没有开启aof，也会触发。

其中通过 save m n 配置的一个例子是，在redis.conf文件中进行如下配置：

```conf
# 时间策略
save 900 1
save 300 10
save 60 10000

# 文件名称
dbfilename dump.rdb

# 文件保存路径
dir /home/work/app/redis/data/

# 如果持久化出错，主进程是否停止写入
stop-writes-on-bgsave-error yes

# 是否压缩
rdbcompression yes

# 导入时是否检查
rdbchecksum yes
```

上述配置的含义如下：

- save 900 1 表示900s内如果有1条是写入命令，就触发产生一次快照(调用bgsave方法)
- save 300 10 表示300s内有10条写入，就产生快照

save 60 10000含义相同，上述三个命令只要满足一个就会进行备份。

- stop-writes-on-bgsave-error yes表示当备份进程出错时，主进程就停止接受新的写入操作，是为了保护持久化的数据一致性问题
- rdbcompression yes表示开启压缩功能，开启后可能会消耗更多的CPU资源，但是可以节省磁盘资源。一般选择不开启，因为CPU资源相对来说更珍贵

如果想要禁用RDB配置，只需要在save的最后一行写上：save ""。

save m n的操作其实是一种Redis定时任务，是由Redis的定时任务机制实现的。定时任务执行的频率可以在配置文件中通过 hz 10 来设置（这个配置表示1s内执行10次，也就是每100ms触发一次定时任务）。配置文件的注释中说，该值最大能够设置为500，但是不建议超过100，因为值越大说明执行频率越频繁越高，这会带来CPU的更多消耗，从而影响主进程读写性能。

## AOF持久化
将写命令添加到 AOF 文件（Append Only File）的末尾，支持命令级和秒级持久化，但是随着服务器写请求的增多，AOF 文件会越来越大，恢复时间慢。

AOF的整个流程大体来看可以分为两步，一步是**命令的实时写入**，第二步**是对aof文件的重写**。

将命令写入AOF文件之前会先写入缓冲区中，步骤是：`命令写入->追加到aof_buf ->同步到aof磁盘`，这是为了减少磁盘IO。

aof重写是为了减少aof文件的大小，可以手动触发也可以自动触发。**手动触发**是通过`bgrewriteaof`，**自动触发**方式可以参见下面的配置。重写的过程可以使用下面的流程图来表示：

<div align="center">
<img src="2347795269-5b70e0fd162b4_articlex.png">
</div>

需要注意以下几点：

- 在重写期间，由于主进程依然在响应命令，为了保证最终备份的完整性；因此它依然会写入旧的AOF file中，如果重写失败，能够保证数据不丢失。
- 为了把重写期间响应的写入信息也写入到新的文件中，因此也会为子进程保留一个buf，防止新写的file丢失数据。
- **重写是直接把当前内存的数据生成对应命令，并不需要读取老的AOF文件进行分析、命令合并。**

AOF模式的配置方式如下：

```conf
# 是否开启aof
appendonly yes

# 文件名称
appendfilename "appendonly.aof"

# 同步方式
appendfsync everysec

# aof重写期间是否同步
no-appendfsync-on-rewrite no

# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof时如果有错如何处理
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes
```
appendfsync everysec 它其实有三种模式:

always：把每个写命令都立即同步到aof，很慢，但是很安全
everysec：每秒同步一次，是折中方案。一般情况下都采用此配置，这样可以兼顾速度与安全，最多损失1s的数据。
no：redis不处理交给OS来处理，非常快，但是也最不安全

aof-load-truncated yes 如果该配置启用，在加载时发现aof尾部不正确是，会向客户端写入一个log，但是会继续执行，如果设置为 no ，发现错误就会停止，必须修复后才能重新加载。

## 恢复数据
Redis异常重启之后需要恢复数据时，会先检查AOF文件是否存在，因为AOF保存的数据更完整。如果不存在就尝试加载RDB。

# 事务
Redis中和事务相关的命令有四个： MULTI 、 DISCARD 、 EXEC 和 WATCH，它们的基本功能如下：

- MULTI：开启一个事务，接下来的命令会放入一个队列中
- DISCARD：终止当前事务并丢弃，也即是清空队列中的命令。该命令必须应用在MULTI命令中
- EXEC：和MULTI配合使用，提交事务并触发执行
- WATCH：因为Redis不支持回滚事务，所以当多个事务操作同一个数据时可能会出现并发问题，WATCH是一种乐观锁的实现方式，用于监视某个变量是不是被更改过。必须在MULTI之前执行

下面较为详细地看一下各个命令。

## MULTI
对于一个非事务的命令，Redis服务器接收到客户端发过来的命令时，会直接执行，并返回执行结果。而使用MULTI开启一个事务之后，服务器在收到来自客户端的命令时， 不会立即执行命令， 而是将这些命令全部放进一个事务队列里， 然后返回 QUEUED ， 表示命令已入队，如下所示：

```redis
redis> multi
OK
redis> set msg "hello world"
QUEUED
redis> get msg
QUEUED
redis> exec
1) OK
2) "hello world"
```
由于Redis中的事务不能嵌套，所以再MULTI命令中再发送一次MULTI命令会返回一个错误，但不会造成事务失败，也不会修改事务队列中已有的数据，如下所示。

```redis
redis> multi
OK
redis> set name Zhangsan
QUEUED
redis> multi
(error) ERR MULTI calls can not be nested
redis> exec
1) OK
redis> get name
"Zhangsan"
```

## EXEC
该命令会提交事务并触发命令的执行，按照FIFO的顺序执行队列中的命令。

## DISCARD
终止当前事务并清空队列。如下面的例子所示，使用MULTI开启一个事务之后，执行set capital_of_China Beijing之后执行DISCARD命令，之后使用get capital_of_China命令得到的结果为nil。

```redis
redis> multi
OK
redis> set capital_of_China Beijing
QUEUED
redis> discard
OK
redis> get capital_of_China
(nil)
```

## WATCH
由于Redis不支持事务回滚，所以当多个事务对同一个变量进行操作的时候，会出现并发问题。如下所示。

首先打开一个客户端，之后开启一个事务，然后将money变量设置为10，但不提交事务。
```redis
redis> multi
OK
redis> set money 10
QUEUED
```
之后打开另一个客户端，开启一个事务，将money变量设置为50，之后提交事务并进行一次查询。
```redis
redis> multi
OK
redis> set money 50
QUEUED
redis> exec
1) OK
redis> get money
"50"
```
可以看到，此时money的值为50。然后在第一个客户端进行提交，然后在两个客户端查询money。
```redis
redis> exec
1) OK
redis> get money
"10"
```
```redis
redis> get money
"10"
```
可以看到，两个客户端的结果都变为了10。为了解决这个问题，可以使用WATCH命令。WATCH用于监控某个key是否发生了改变，如果在一个事务中，某个key在设置参数之后，在执行exec之前，其他客户端修改了该key，则该事务将返回nil。

还是上面的例子，打开一个客户端，执行以下命令：
```redis
redis> flushall
OK
redis> watch money
OK
redis> multi
OK
redis> set money 10
QUEUED
```
在另一个客户端中将money改为50
```redis
redis> multi
OK
redis> set money 50
QUEUED
redis> exec
1) OK
```
之后在第一个客户端中提交事务，会返回nil，同时查询money的值，为50。
```redis
redis> exec
(nil)
redis> get money
"50"
```

需要注意的是：

- WATCH监控的元素在当前事务提交之前发生变化（另一个事务执行了EXEC），则无论当前事务中有多少命令，全部失败
- WATCH监控的元素在当前事务提交之前，被放入到另外一个事务的队列中，但并未执行EXEC，则当前事务可正常提交

### WATCH命令的实现
每个Redis数据库保存着一个watched_keys字典，这个字典的键是某个被WATCH命令监视的数据库键，而字典的值是一个链表，链表记录了所有监视相应数据库键的客户端。如下图所示。

<div align="center">
<img src="18464438-7358a18d9263b767.png">
</div>

所有对数据库进行修改命令，如SET、LPUSH、SADD、ZREM、DEL等，在执行后都会对watched_keys字典进行检查，查看被修改的数据库键是否是被客户端所监视的键，如果有的话，客户端REDIS_DIRTY_CAS标识将会被打开，表示该客户端的事务安全性已经被破坏。

当服务器接收到一个客户端发来的EXEC命令，服务器会根据这个客户端是否打开了REDIS_DIRTY_CAS标识来决定是否执行事务。

# 事件
Redis是一个事件驱动的服务器，主要有两类事件：

- 文件事件(file event)：Redis服务器通过套接字与客户端或其他Redis服务器进行连接，文件事件就是服务器对套接字操作的抽象。
- 时间事件(time event)：redis服务器中的一些操作（比如serverCron函数）需要在给定的时间点执行，而时间事件就是对这类定时操作的抽象。

## 文件事件
redis基于Reactor模式开发了自己的网络事件处理器，即文件事件处理器(file event handler)：
- 文件事件处理器以单线程方式运行。通过使用I/O多路复用程序来监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
- 当被监听的套接字准备好执行连接应答(accept)，读取(read)，写入(write)，关闭(close)等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

文件事件处理器的四个组成部分，分别是套接字，I/O多路复用程序，文件事件分派器（dispatch），以及事件处理器。如下图所示。

<div algin="center">
<img src="file event handler.PNG">
</div>

程序会在编译时自动选择系统中性能最高的I/O多路复用函数库来作为redis的I/O多路复用程序的底层实现，包括 Solaries 10 中的 evport、Linux 中的 epoll 和 macOS/FreeBSD 中的 kqueue。如果当前编译环境没有上述函数，就会选择 select 作为备选方案

## 时间事件
redis的时间事件分为以下两类：(目前版本的redis只使用周期性事件，而没有使用定时事件。)

- 定时事件：让一段程序在指定的时间之后执行一次。比如说，让程序X在当前时间的30毫秒之后执行一次。
- 周期性事件：让一段程序每隔指定时间就执行一次。比如说，让程序Y每隔30毫秒就执行一次。

服务器将所有时间事件都放在一个无序链表中，每当时间事件执行器运行时，它就遍历整个链表，查找所有已到达的时间事件，并调用相应的事件处理器。正常模式下的redis服务器只使用serverCron一个时间事件。

# 复制


# 参考
---
[Redis设计与实现-黄建宏](http://redisbook.com/)</br>
[Redis详解（四）------redis的底层数据结构](https://www.cnblogs.com/ysocean/p/9080942.html#_label6)</br>
[Redis详解（五）------redis的五大数据类型实现原理](https://www.cnblogs.com/ysocean/p/9102811.html)</br>
[Redis详解（三）------redis的五大数据类型详细用法](https://www.cnblogs.com/ysocean/p/9080940.html#_label4)</br>
[漫画算法：什么是跳跃表？](http://blog.jobbole.com/111731/)</br>
[跳跃表-原理及Java实现](http://www.cnblogs.com/acfox/p/3688607.html)</br>
[跳表SkipList的原理和实现](https://imtinx.iteye.com/blog/1291165)</br>
[一起看懂Redis两种持久化方式的原理](https://segmentfault.com/a/1190000015983518?utm_source=tag-newest)</br>
[深入学习Redis（2）：持久化](https://www.cnblogs.com/williamjie/p/9230557.html)
[Redis事务](https://www.cnblogs.com/qq931399960/p/10556699.html)</br>
[Redis事务](https://www.jianshu.com/p/479398a8e82c)</br>
[]()</br>
---


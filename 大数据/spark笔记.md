spark笔记

<!-- TOC -->

- [概念](#%E6%A6%82%E5%BF%B5)
- [spark运作方式](#spark%E8%BF%90%E4%BD%9C%E6%96%B9%E5%BC%8F)
- [spark安装](#spark%E5%AE%89%E8%A3%85)
- [RDD编程](#rdd%E7%BC%96%E7%A8%8B)
    - [构建开发环境](#%E6%9E%84%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)
    - [RDD介绍](#rdd%E4%BB%8B%E7%BB%8D)
    - [创建RDD](#%E5%88%9B%E5%BB%BArdd)
    - [RDD操作](#rdd%E6%93%8D%E4%BD%9C)
        - [转化操作](#%E8%BD%AC%E5%8C%96%E6%93%8D%E4%BD%9C)
        - [行动操作](#%E8%A1%8C%E5%8A%A8%E6%93%8D%E4%BD%9C)
    - [向Spark传递函数](#%E5%90%91spark%E4%BC%A0%E9%80%92%E5%87%BD%E6%95%B0)
    - [专有RDD](#%E4%B8%93%E6%9C%89rdd)
    - [持久化](#%E6%8C%81%E4%B9%85%E5%8C%96)
- [键值对操作](#%E9%94%AE%E5%80%BC%E5%AF%B9%E6%93%8D%E4%BD%9C)
    - [创建Pair RDD](#%E5%88%9B%E5%BB%BApair-rdd)
    - [Pair RDD的转化操作](#pair-rdd%E7%9A%84%E8%BD%AC%E5%8C%96%E6%93%8D%E4%BD%9C)
        - [聚合操作](#%E8%81%9A%E5%90%88%E6%93%8D%E4%BD%9C)
    - [Pair RDD的行动操作](#pair-rdd%E7%9A%84%E8%A1%8C%E5%8A%A8%E6%93%8D%E4%BD%9C)
    - [数据分区](#%E6%95%B0%E6%8D%AE%E5%88%86%E5%8C%BA)
        - [与数据分区相关的操作](#%E4%B8%8E%E6%95%B0%E6%8D%AE%E5%88%86%E5%8C%BA%E7%9B%B8%E5%85%B3%E7%9A%84%E6%93%8D%E4%BD%9C)
- [数据读取与保存](#%E6%95%B0%E6%8D%AE%E8%AF%BB%E5%8F%96%E4%B8%8E%E4%BF%9D%E5%AD%98)
    - [文件格式](#%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F)
        - [文本格式](#%E6%96%87%E6%9C%AC%E6%A0%BC%E5%BC%8F)
        - [JSON](#json)
        - [CSV和TSV](#csv%E5%92%8Ctsv)
        - [sequenceFile](#sequencefile)
    - [文件系统](#%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
        - [本地/“常规”文件系统](#%E6%9C%AC%E5%9C%B0%E5%B8%B8%E8%A7%84%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
        - [Amazon S3](#amazon-s3)
        - [HDFS](#hdfs)
    - [Spark SQL中的结构化数据](#spark-sql%E4%B8%AD%E7%9A%84%E7%BB%93%E6%9E%84%E5%8C%96%E6%95%B0%E6%8D%AE)
- [Spark编程进阶](#spark%E7%BC%96%E7%A8%8B%E8%BF%9B%E9%98%B6)
    - [累加器](#%E7%B4%AF%E5%8A%A0%E5%99%A8)
- [spark sql](#spark-sql)
    - [连接Spark sql](#%E8%BF%9E%E6%8E%A5spark-sql)
    - [在应用中使用Saprk sql](#%E5%9C%A8%E5%BA%94%E7%94%A8%E4%B8%AD%E4%BD%BF%E7%94%A8saprk-sql)
        - [初始化Spark sql](#%E5%88%9D%E5%A7%8B%E5%8C%96spark-sql)
        - [基本查询示例](#%E5%9F%BA%E6%9C%AC%E6%9F%A5%E8%AF%A2%E7%A4%BA%E4%BE%8B)
        - [SchemaRDD](#schemardd)
    - [读取和存储数据](#%E8%AF%BB%E5%8F%96%E5%92%8C%E5%AD%98%E5%82%A8%E6%95%B0%E6%8D%AE)
        - [apache hive](#apache-hive)
        - [json](#json)
            - [将RDD转化为SchemaRDD](#%E5%B0%86rdd%E8%BD%AC%E5%8C%96%E4%B8%BAschemardd)

<!-- /TOC -->

## 概念
`Spark`是一个用来实现快速而通用的集群计算(分布式计算)的平台。

`spark`组件如下图所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/spark1.PNG">
</div>

各个部分的作用如下：

- `Spark Core`:`Spark Core` 实现了 `Spark` 的基本功能，包含任务调度、内存管理、错误恢复、与存储系统交互等模块。`Spark Core` 中还包含了对**弹性分布式数据集** `distributed dataset`，简称 `RDD`）的 `API` 定义。`RDD` 表示分布在多个计算节点上可以并行操作的元素集合，是`Spark` 主要的编程抽象，是 `Spark` 对分布式数据和计算的基本抽象。`Spark Core` 提供了创建和操作这些集合的多个 `API`。
- `Spark SQL`:`Spark SQL` 是 `Spark` 用来操作结构化数据的程序包。通过 `Spark SQL`，可以使用 `SQL`或者 `Apache Hive` 版本的 `SQL` 方言（`HQL`）来查询数据。
- `Spark Streaming`:`Spark Streaming` 是 `Spark` 提供的对实时数据进行流式计算的组件。
- `MLlib`:`MLlib`是`spark`提供的机器学习（`ML`）功能的程序库。`MLlib` 提供了很多种机器学习算法，包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。
- `GraphX`:`GraphX` 是用来操作图（比如社交网络的朋友关系图）的程序库，可以进行并行的图计算。
- 集群管理器:`Spark` 支持在各种集群管理器（`cluster manager`）上运行，包括 `Hadoop YARN、Apache Mesos`，以及 `Spark` 自带的一个简易调度器:独立调度器。

## spark运作方式
每个 `Spark` 应用都由一个**驱动器程序**（`driver program`）来发起集群上的各种并行操作。驱动器程序包含应用的 `main` 函数，并且定义了集群上的分布式数据集，还对这些分布式数据集应用了相关操作。

驱动器程序通过一个 **`SparkContext`** 对象来访问 `Spark`。这个对象代表**对计算集群的一个连接**。

有了 `SparkContext`，就可以用它来创建 `RDD`，并且对`RDD`执行某些操作。

要执行这些操作，驱动器程序一般要管理多个**执行器**（`executor`）节点。

最后，使用用来传递函数的 `API`，可以将对应操作运行在集群上。

示意图如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/spark2.PNG">
</div>

## spark安装
略

## RDD编程

### 构建开发环境
在`IDEA`中新建一个`maven`工程，并填写`spark`依赖：

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.3.4</version>
</dependency>
```

### RDD介绍
`Spark `中的 `RDD` 就是一个不可变的分布式对象集合。每个 `RDD` 都被分为多个**分区**，这些分区运行在集群中的不同节点上。

创建 `RDD`的方法有两种：**读取一个外部数据集**，或**在驱动器程序里分发驱动器程序中的对象集合（比如 `list` 和 `set`）**。

创建出来后，`RDD` 支持两种类型的操作： **转化操作**（`transformation`） 和**行动操作（`action`）**。转化操作会由一个 `RDD` 生成一个新的 `RDD`。行动操作会对 `RDD` 计算出一个结果，并把结果返回到驱动器程序中，或把结果存储到外部存储系统（如 `HDFS）`中。（类似`java stream`中的*中间操作*和*终端操作*）。

“行动操作”其实是一种“惰性计算”的方式，这在大数据领域很有道理，这样就会避免很多中间结果的存储，避免了存储空间的浪费。

默认情况下，`Spark` 的 `RDD` 会在每次对它们进行行动操作时**重新计算**。如果想在多个行动操作中重用同一个 `RDD`，可以使用 `RDD.persist()` 让 `Spark` 把这个 `RDD` 缓存下来。

### 创建RDD

- 对集合进行并行化的方式创建RDD

```java
// sc即SparkContext对象
JavaRDD<String> lines = sc.parallelize(Arrays.asList("pandas", "i like pandas"));
```

- 是从外部存储中读取数据来创建 RDD

```java
JavaRDD<String> lines = sc.textFile("/path/to/README.md");
```

### RDD操作
#### 转化操作
`RDD `的转化操作是返回新 `RDD` 的操作。且转化出来的 `RDD` 是惰性求值的。

```java
JavaRDD<String> inputRDD = sc.textFile("log.txt"); 
JavaRDD<String> errorsRDD = inputRDD.filter( 
  new Function<String, Boolean>() { 
    public Boolean call(String x) { return x.contains("error"); } 
  } 
});
```

`filter()` 操作不会改变已有的 `inputRDD` 中的数据，该操作会返回一个全新的 `RDD`，`inputRDD` 在后面的程序中还可以继续使用。

通过转化操作，从已有的 `RDD` 中派生出新的 `RDD`，`Spark` 会使用**谱系图**（`lineage graph`）来记录这些不同 `RDD` 之间的依赖关系。`Spark` 需要用这些信息来按需计算每个 `RDD`，也可以依靠谱系图在持久化的 `RDD` 丢失部分数据时恢复所丢失的数据。如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/spark3.PNG">
</div>

常见的转化操作如下：

对一个数据为`{1, 2, 3, 3}`的`RDD`进行基本的`RDD`转化操作:

|函数名| 目的 |示例| 结果|
|-|-|-|-|
|map() |将函数应用于 RDD 中的每个元素，将返回值构成新的 RDD|rdd.map(x => x + 1)| {2, 3, 4, 4}|
|flatMap() |将函数应用于 RDD 中的每个元素，将返回的迭代器的所有内容构成新的 RDD。通常用来切分单词|rdd.flatMap(x => x.to(3))| {1, 2, 3, 2, 3, 3, 3}|
|filter()| 返回一个由通过传给 filter()的函数的元素组成的 RDD|rdd.filter(x => x != 1)| {2, 3, 3}|
|distinct()|去重| rdd.distinct()| {1, 2, 3}|
|sample(withReplacement, fraction, [seed])|对 RDD 采样，以及是否替换| rdd.sample(false, 0.5)| 非确定的|

对数据分别为{1, 2, 3}和{3, 4, 5}的RDD进行针对两个RDD的转化操作:

|函数名 |目的 |示例| 结果|
|-|-|-|-|
|union()| 生成一个包含两个 RDD 中所有元素的 RDD,如果输入的 RDD 中有重复数据，结果中会包含这些重复数据|rdd.union(other)| {1, 2, 3, 3, 4, 5}|
|intersection()| 求两个 RDD 共同的元素的 RDD |rdd.intersection(other)| {3}|
|subtract()| 移除一个 RDD 中的内容（例如移除训练数据）|rdd.subtract(other)| {1, 2}|
|cartesian()| 与另一个 RDD 的笛卡儿积| rdd.cartesian(other)| {(1,  3),  (1,  4),  ... (3, 5)}|


#### 行动操作
行动操作会把最终求得的结果返回到驱动器程序，或者写入外部存储系统中。

```java
System.out.println("Input had " + badLinesRDD.count() + " concerning lines") 
System.out.println("Here are 10 examples:") 
for (String line: badLinesRDD.take(10)) { 
  System.out.println(line); 
}
```

上例使用 `take()` 获取了 `RDD` 中的少量元素。然后在本地遍历这些元素，并在驱动器端打印出来。`RDD` 还有一个 `collect()` 函数，可以用来获取**整个** `RDD` 中的数据。因此，`collect()` 最好不要用在大规模数据集上。

常见的行动操作如下：

对一个数据为`{1, 2, 3, 3}`的RDD进行基本的`RDD`行动操作:

|函数名| 目的| 示例| 结果|
|-|-|-|-|
|collect()| 返回 RDD 中的所有元素| rdd.collect() |{1, 2, 3, 3}|
|count() |RDD 中的元素个数| rdd.count() |4|
|countByValue()|各元素在 RDD 中出现的次数| rdd.countByValue() |{(1, 1), (2, 1), (3, 2)}|
|take(num)| 从 RDD 中返回 num 个元素| rdd.take(2) {1, 2}|
|top(num)| 从 RDD 中返回最前面的 num个元素|rdd.top(2) |{3, 3}|
|takeOrdered(num)(ordering)|从 RDD 中按照提供的顺序返回最前面的 num 个元素|rdd.takeOrdered(2)(myOrdering) |{3, 3}|
|takeSample(withReplacement, num, [seed])|从 RDD 中返回任意一些元素| rdd.takeSample(false, 1) |非确定的|
|reduce(func) |并行整合RDD中所有数据（例如 sum）|rdd.reduce((x, y) => x + y) |9|
|fold(zero)(func)| 和 reduce() 一 样， 但 是 需 要提供初始值|rdd.fold(0)((x, y) => x + y) |9|
|aggregate(zeroValue)(seqOp, combOp)|和 reduce() 相 似， 但 是 通 常返回不同类型的函数|rdd.aggregate((0, 0))((x, y) => (x._1 + y, x._2 + 1), (x, y) 
(x._1 + y._1, x._2 + y._2)) |(9,4)|
|foreach(func) |对 RDD 中的每个元素使用给定的函数|rdd.foreach(func) |无|

其中，`aggregate()` 函数允许返回值类型与所操作的 `RDD` 类型不同。使用 `aggregate()` 时，需要提供期待返回的类型的初始值。然后通过第一个函数把 `RDD` 中的元素合并起来放入累加器。然后，需要提供第二个函数来将累加器两两合并。

使用`aggregate()` 函数实现计算`RDD`平均值的方法如下：

```java
class AvgCount implements Serializable { 
  public AvgCount(int total, int num) { 
    this.total = total; 
    this.num = num; 
  } 
  public int total; 
  public int num; 
   public double avg() { 
    return total / (double) num; 
  } 
} 
Function2<AvgCount, Integer, AvgCount> addAndCount = 
  new Function2<AvgCount, Integer, AvgCount>() { 
    public AvgCount call(AvgCount a, Integer x) { 
      a.total += x; 
      a.num += 1; 
      return a; 
  } 
}; 
Function2<AvgCount, AvgCount, AvgCount> combine = 
  new Function2<AvgCount, AvgCount, AvgCount>() { 
  public AvgCount call(AvgCount a, AvgCount b) { 
    a.total += b.total; 
    a.num += b.num; 
    return a; 
  } 
}; 
AvgCount initial = new AvgCount(0, 0); 
AvgCount result = rdd.aggregate(initial, addAndCount, combine); 
System.out.println(result.avg());
```

### 向Spark传递函数
`Spark` 的大部分转化操作和一部分行动操作，都需要依赖用户传递的函数来计算。在 `Java` 中，函数需要作为实现了 `Spark` 的 `org.apache.spark.api.java.function` 包中的任一函数接口的对象来传递。最基本的函数式接口如下：

|函数名 |实现的方法 |用途|
|-|-|-|
|Function<T, R>| R call(T) |接收一个输入值并返回一个输出值，用于类似 map() 和filter() 等操作中|
|Function2<T1, T2, R>| R call(T1, T2) |接收两个输入值并返回一个输出值，用于类似 aggregate()和 fold() 等操作中|
|FlatMapFunction<T, R> |Iterable<R> call(T)| 接收一个输入值并返回任意个输出，用于类似 flatMap()这样的操作中|

例子如下：

```java
// 匿名类
RDD<String> errors = lines.filter(new Function<String, Boolean>() { 
  public Boolean call(String x) { return x.contains("error"); } 
});

// lamda表达式
RDD<String> errors = lines.filter(s -> s.contains("error"));
```

### 专有RDD
有些函数只能用于特定类型的 `RDD`，比如 `mean()` 和 `variance()` 只能用在数值 `RDD` 上，而 `join()` 只能用在键值对 `RDD` 上。在 `Java` 中，这些函数都没有定义在标准的 `RDD`类中，所以要访问这些附加功能，必须要确保获得了正确的专用 `RDD` 类。

`Java` 中有两个专门的类 `JavaDoubleRDD`和 `JavaPairRDD`，来处理特殊类型的 `RDD`。

要构建出这些特殊类型的 `RDD`，需要使用特殊版本的类来替代一般使用的 `Function` 类。如果要从 `T` 类型的 `RDD` 创建出一个 `DoubleRDD`，应当在映射操作中使用 `<T>`来替代 `<T, Double>`。同时，当需要一个 `DoubleRDD` 时，应当调用 `mapToDouble()` 来替代`map()`。如下：

```java
JavaDoubleRDD result = rdd.mapToDouble( 
    new DoubleFunction<Integer>() { 
        public double call(Integer x) { 
        return (double) x * x; 
        } 
    }); 
    System.out.println(result.mean());
```

Java中针对专门类型的函数接口如下：

|函数名 |等价函数 |用途|
|-|-|-|
|DoubleFlatMapFunction<T>| Function<T, Iterable<Double>> |用于 flatMapToDouble，以生成 DoubleRDD|
|DoubleFunction<T> |Function<T, Double>| 用于 mapToDouble，以生成DoubleRDD|
|PairFlatMapFunction<T, K, V> |Function<T, Iterable<Tuple2<K, V>>> |用于 flatMapToPair，以生成 PairRDD<K, V>|
|PairFunction<T, K, V>| Function<T, Tuple2<K, V>> |用 于 mapToPair， 以生成PairRDD<K, V>|

### 持久化
为了避免多次计算同一个 `RDD`，可以让 `Spark` 对数据进行持久化。默认情况下 `persist()` 会把数据以序列化的形式缓存在 `JVM` 的堆空间中。

`org.apache.spark.storage.StorageLevel和pyspark.StorageLevel`中的持久化级别如下；如有必要，可以通过在存储级别的末尾加上“`_2`”来把持久化数据存为两份。

|级别|使用的空间|CPU时间|是否在内存中|是否在磁盘上|备注|
|-|-|-|-|-|-|
|MEMORY_ONLY| 高| 低 |是| 否  |
|MEMORY_ONLY_SER |低 |高| 是 |否  |
|MEMORY_AND_DISK |高| 中等| 部分| 部分 |如果数据在内存中放不下，则溢写到磁盘上|
|MEMORY_AND_DISK_SER| 低 |高 |部分| 部分 |如果数据在内存中放不下，则溢写到磁盘上。在内存中存放序列化后的数据|
|DISK_ONLY |低| 高| 否 |是 |

如果要缓存的数据太多，内存中放不下，`Spark` 会自动利用最近最少使用（`LRU`）的缓存策略把最老的分区从内存中移除。 调用`unpersist()`方法可以手动把持久化的 `RDD` 从缓存中移除。

## 键值对操作
`Spark` 为包含键值对类型的 `RDD` 提供了一些专有的操作。这些 `RDD` 被称为 `pair RDD`。

### 创建Pair RDD
`Java` 没有自带的二元组类型，因此 `Spark` 的 `Java API` 让用户使用 `scala.Tuple2` 类来创建二元组。通过 `new  Tuple2(elem1,  elem2)` 来创建一个新的二元组，并且可以通过 `._1() 和 ._2()` 方法访问其中的元素。

`Java` 用户还需要调用专门的 `Spark` 函数来创建 `pair RDD`。例如，要使用 `mapToPair()` 函数来代替基础版的 `map()` 函数，如下。

```java
PairFunction<String, String, String> keyData = 
  new PairFunction<String, String, String>() { 
  public Tuple2<String, String> call(String x) { 
    return new Tuple2(x.split(" ")[0], x); 
  } 
}; 
JavaPairRDD<String, String> pairs = lines.mapToPair(keyData);
```
`Java` 从内存数据集创建 `pair RDD`的话，需要使用 `SparkContext.parallelizePairs()`。

### Pair RDD的转化操作
常用的`Pair RDD`操作如下（以键值对集合{(1, 2), (3, 4), (3, 6)}为例）：

|函数名 |目的 |示例 |结果|
|-|-|-|-|
|reduceByKey(func) |合并具有相同键的值| rdd.reduceByKey((x, y) => x + y) |{(1, 2), (3, 10)}|
|groupByKey()| 对具有相同键的值进行分组 |rdd.groupByKey()| {(1, [2]), (3, [4, 6])}|
|combineByKey( createCombiner,mergeValue, mergeCombiners,partitioner)|使用不同的返回类型合并具有相同键的值|
|mapValues(func)| 对 pair RDD 中的每个值应用一个函数而不改变键|rdd.mapValues(x => x+1) |{(1, 3), (3, 5), (3, 7)}|
|flatMapValues(func)| 对 pair RDD 中的每个值应用一个返回迭代器的函数，然后对返回的每个元素都生成一个对应原键的键值对记录。通常用于符号化||
|keys() |返回一个仅包含键的 RDD| rdd.keys() |{1, 3, 3}|
|values() |返回一个仅包含值的 |RDD rdd.values()| {2, 4, 6}|
|sortByKey() |返回一个根据键排序的| RDD rdd.sortByKey() |{(1, 2), (3, 4), (3, 6)}|

针对两个pair RDD的转化操作（`rdd = {(1, 2), (3, 4), (3, 6)}other = {(3, 9)}`）

|函数名| 目的| 示例| 结果|
|-|-|-|-|
|subtractByKey |删掉 RDD 中键与 other RDD 中的键相同的元素|rdd.subtractByKey(other) |{(1, 2)}|
|join |对两个 RDD 进行内连接 |rdd.join(other) |{(3, (4, 9)), (3, (6, 9))}|
|rightOuterJoin |对两个 RDD 进行连接操作，确保第二个 RDD 的键必须存在（右外连接）|rdd.rightOuterJoin(other) |{(3,(Some(4),9)), (3,(Some(6),9))}|
|leftOuterJoin |对两个 RDD 进行连接操作，确保第一个 RDD 的键必须存在（左外连接）|rdd.leftOuterJoin(other) |{(1,(2,None)), (3, (4,Some(9))), (3,(6,Some(9)))}|
|cogroup |将两个 RDD 中拥有相同键的数据分组到一起|rdd.cogroup(other)| {(1,([2],[])), (3, ([4, 6],[9]))}|

#### 聚合操作
对于基础 `RDD` 上的 `fold()、combine()、reduce()` 等行动操作，`pair RDD` 上则有相应的针对键的转化操作。

- `reduceByKey()`:它接收一个函数，并使用该函数对值进行合并。`reduceByKey()` 会为数据集中的每个键进行并行的归约操作，每个归约操作会将键相同的值合并起来。它会返回一个由各键和对应键归约出来的结果值组成的新的 `RDD`。
- `foldByKey()`:它使用一个与 `RDD` 和合并函数中的数据类型相同的零值作为初始值。与 `fold()` 一样，`foldByKey()` 操作所使用的合并函数对零值与另一个元素进行合并，结果仍为该元素。
- `combineByKey()`:`combineByKey()` 可以让用户返回与输入数据的类型不同的返回值。 由于`combineByKey()` 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就和之前的某个元素的键相同。
    - 如果这是一个新的元素，`combineByKey()` 会使用一个叫作 `createCombiner()` 的函数来创建那个键对应的累加器的初始值。需要注意的是，这一过程会在每个分区中第一次出现各个键时发生，而不是在整个 `RDD` 中第一次出现一个键时发生。
    - 如果这是一个在处理当前分区之前已经遇到的键，它会使用 `mergeValue()` 方法将该键的累加器对应的当前值与这个新的值进行合并。
    - 由于每个分区都是独立处理的，因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器，就需要使用用户提供的 `mergeCombiners()` 方法将各
个分区的结果进行合并。

在 `Java` 中使用 `combineByKey()` 求每个键对应的平均值的例子如下：

```java
public static class AvgCount implements Serializable { 
  public AvgCount(int total, int num) {   total_ = total;  num_ = num; } 
  public int total_; 
  public int num_; 
  public float avg() {   returntotal_/(float)num_; } 
} 
 
Function<Integer, AvgCount> createAcc = new Function<Integer, AvgCount>() { 
  public AvgCount call(Integer x) {
      return new AvgCount(x, 1); 
  } 
}; 
Function2<AvgCount, Integer, AvgCount> addAndCount = 
  new Function2<AvgCount, Integer, AvgCount>() { 
  public AvgCount call(AvgCount a, Integer x) { 
    a.total_ += x; 
    a.num_ += 1; 
    return a; 
  } 
}; 
Function2<AvgCount, AvgCount, AvgCount> combine = 
  new Function2<AvgCount, AvgCount, AvgCount>() { 
  public AvgCount call(AvgCount a, AvgCount b) { 
    a.total_ += b.total_;  
    a.num_ += b.num_;  
    return a; 
  } 
}; 
AvgCount initial = new AvgCount(0,0); 
JavaPairRDD<String, AvgCount> avgCounts = 
  nums.combineByKey(createAcc, addAndCount, combine); 
Map<String, AvgCount> countMap = avgCounts.collectAsMap(); 
for (Entry<String, AvgCount> entry : countMap.entrySet()) { 
  System.out.println(entry.getKey() + ":" + entry.getValue().avg()); 
}
```

分区处理过程的示意图如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/spark4.PNG">
</div>

每个 `RDD` 都有固定数目的分区，分区数决定了在 `RDD` 上执行操作时的并行度。若指定并行度，`spark`会使用默认值。上面提到的大多数操作都可以接受第二个参数，用来指定分组结果或聚合结果的`RDD` 的分区数。例如：

```java
// 自定义并行度为10
sc.parallelize(data).reduceByKey(lambda x, y: x + y, 10)
```

`Spark` 提供了 `repartition()` 函数和`coalesce()`函数（是前者的优化）把数据通过网络进行混洗，并创建出新的分区集合，但是代价十分巨大。 使用`rdd.partitions.size()`可以查看分区的大小。

### Pair RDD的行动操作
Pair RDD的行动操作如下（以键值对集合`{(1, 2), (3, 4), (3, 6)}`为例）：

|函数 |描述 |示例 |结果|
|-|-|-|-|
|countByKey() |对每个键对应的元素分别计数| rdd.countByKey() |{(1, 1), (3, 2)}|
|collectAsMap() |将结果以映射表的形式返回，以便查询| rdd.collectAsMap()| Map{(1, 2), (3, 4), (3, 6)}|
|lookup(key)|返回给定键对应的所有值 |rdd.lookup(3) |[4, 6]|

### 数据分区
分布式系统中的数据会分散分散存储在不同的机器上，在计算式会将数据通过网络传输聚集到一起，，因此控制数据分布以获得最少的网络传输可以极大地提升整体性能，减少**数据混洗**(即`shuffle`,让数据在分布式节点之间重新分布以使得某些数据被放在同一分区里)。

`Spark` 中所有的键值对 `RDD` 都可以进行分区。常见的分区方式有**哈希分区**和**范围分区**。前者是根据键的哈希值进行分区，后者会根据键的范围进行分区。

当不使用分区对两个数据集进行`join`时，每一次`join`都会把两个数据集中所有键的哈希值都求出来，将该哈希值相同的记录通过网络传到同一台机器上，然后在那台机器上对所有键相同的记录进行连接。如下所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/spark5.PNG">
</div>

为了避免这种情况，可以在`join开`始之前对`userData`表使用`partitionBy()`转化操作，将这张表转为哈希分区。将`userData`分区后，`spark`会自动利用这一点。当调用`userData.join(events)`时，`spark`就只会对`events`数据进行混洗操作，将`events`中特定的`userID`的记录发送到`userData`的对应分区所在的那台机器上，这样可以大大减少网络传输以及计算量。如下所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/spark6.PNG">
</div>

使用`partitionBy`的例子如下：

```java
JavaRDD<String> input = jsc.parallelize(Arrays.asList("1", "2", "3"));
        JavaPairRDD<String, Integer> counts = input
                .mapToPair(x -> new Tuple2<>(x, 1))
                .reduceByKey((x, y) -> (x + y));
// 分区并持久化，如果想使用范围分区，只许将HashPartitioner替换为RangePartitioner
JavaPairRDD<String, Integer> partitionedCount = counts.partitionBy(new HashPartitioner(10)).persist(StorageLevel.DISK_ONLY());
// 获得分区信息，Partitione对象描述额 RDD 中各个键分别属于哪个分区
Optional<Partitioner> opt = partitionedCount.partitioner();
if (opt.isPresent()) {
    Partitioner partitioner = opt.get();
}
```

要注意的是，**`partitionBy()`是一个转化操作。因此它的返回值总是一个新的`RDD`，但它不会改变原来的`RDD`。所以一定要对`partitionBy`的结果进行持久化。不然每一次`jion`它都会重新分区。**

除`HashPartitioner`和`RangePartitioner`之外，还可以自定义分区方式，需要继承`Partitioner`类并实现以下几个方法。

- `numPartitions`: `Int`,返回创建出来的分区数。
- `getPartition(key: Any)`: `Int`,返回给定键的分区编号（`0` 到 `numPartitions-1`）。
- `equals()`：`Java` 判断相等性的标准方法。这个方法的实现非常重要，`Spark` 需要用这个方法来检查自定义的分区器对象是否和其他分区器实例相同，这样 `Spark` `才可以判断两个RDD` 的分区方式是否相同。
- `hashCode()`：与`equals()`实现同时实现，保证两个对象相同的时候，`hashcode`是相同的。


#### 与数据分区相关的操作
下面的操作都会触发根据数据的键产生混洗的操作：

- `cogroup()`
- `groupWith()`
- `join()`
- `leftOuterJoin()`
- `rightOuterJoin()`
- `groupByKey()`
- `reduceByKey()`
- `combineByKey()`
- `lookup()`

对于像 `reduceByKey()` 这样只作用于单个 `RDD` 的操作，运行在未分区的 `RDD` 上的时候会导致每个键的所有对应值都在每台机器上进行本地计算，只需要把本地最终归约出的结果值从各工作节点传回主节点，所以原本的网络开销就不算大，即使使用分区，也不会对性能有太大提高。

而对于诸如 `cogroup()` 和`join()` 这样的二元操作，预先进行数据分区会导致其中至少一个 `RDD`（使用已知分区器的那个 `RDD`）不发生数据混洗，对这种操作，使用分区会对性能有比较大提高。

许多其他 `Spark` 操作会自动为结果 `RDD` 设定已知的分区方式信息如下：

- `cogroup()`
- `groupWith()`
- `join()`
- `leftOuterJoin()`
- `rightOuterJoin()`
- `groupByKey()`
- `reduceByKey()`
- `combineByKey()`
- `partitionBy()`
- `sort()`
- `mapValues()`（如果父 `RDD` 有分区方式的话）
- `flatMapValues()`（如果父 `RDD` 有分区方式的话）
- `filter()`（如果父 `RDD` 有分区方式的话）

对于二元操作，输出数据的分区方式取决于父 `RDD` 的分区方式。默认情况下，结果会采用哈希分区，分区的数量和操作的并行度一样。不过，如果其中的一个父 `RDD` 已经设置过分区方式，那么结果就会采用那种分区方式；如果两个父 `RDD` 都设置过分区方式，结果 `RDD` 会采用第一个父 `RDD` 的分区方式。

## 数据读取与保存
常见的`spark`数据源有三类：

- 文件格式与文件系统：文件的存储来源可以是本地文件系统或分布式文件系统（比如 `NFS、HDFS、Amazon S3` 等）,格式多样，包括文本文件、`JSON、SequenceFile`，
以及 `protocol buffer`等
- `Spark SQL`中的结构化数据源：比如 JSON 和 Apache Hive 在内的结构化数据源
- 数据库与键值存储：可以使用Spark 自带的库和一些第三方库来连接 Cassandra、HBase、Elasticsearch 以及 JDBC 源

### 文件格式
|格式名称|结构化|备注|
|-|-|-|
|文本文件 |否 |普通的文本文件，每行一条记录|
|JSON| 半结构化 |常见的基于文本的格式，半结构化；大多数库都要求每行一条记录|
|CSV| 是 |非常常见的基于文本的格式，通常在电子表格应用中使用|
|SequenceFiles |是| 一种用于键值对数据的常见 Hadoop 文件格式|
|Protocol buffers |是| 一种快速、节约空间的跨语言格式|
|对象文件| 是| 用来将 Spark 作业中的数据存储下来以让共享的代码读取。改变类的时候它会失效，因为它依赖于 Java 序列化|

#### 文本格式
读取文本格式的时候，输入的每一行都会成为 RDD 的一个元素。如果文件较小，也可以使用wholeTextFiles()方法将多个完整的文本文件一次性读取为一个 pair RDD，其中键是文件名，值是文件内容。

- 读取

```java
JavaRDD<String> input = sc.textFile("file:///home/holden/repos/spark/README.md");
```

- 保存
 使用saveAsTextFile() 方法保存文件，传入一个路径，但是不能控制数据的哪一部分输出到哪个文件中。

```java
input.saveAsTextFile("file:///home/holden/repos/spark/");
```

#### JSON
读取 JSON 数据的最简单的方式是将数据作为文本文件读取，然后使用 JSON 解析器来对 RDD 中的值进行映射操作。使用Jackson读取json的例子如下(使用 mapPartitions() 来重用解析器)：

```java
class ParseJson implements FlatMapFunction<Iterator<String>, Person> { 
  public Iterable<Person> call(Iterator<String> lines) throws Exception { 
    ArrayList<Person> people = new ArrayList<Person>(); 
    ObjectMapper mapper = new ObjectMapper(); 
    while (lines.hasNext()) { 
      String line = lines.next(); 
      try { 
        people.add(mapper.readValue(line, Person.class)); 
      } catch (Exception e) { 
        // 跳过失败的数据 
      } 
    } 
    return people; 
  } 
} 
JavaRDD<String> input = sc.textFile("file.json"); 
JavaRDD<Person> result = input.mapPartitions(new ParseJson());
```

写入json的方式如下：

```java
class WriteJson implements FlatMapFunction<Iterator<Person>, String> { 
  public Iterable<String> call(Iterator<Person> people) throws Exception { 
    ArrayList<String> text = new ArrayList<String>(); 
    ObjectMapper mapper = new ObjectMapper(); 
    while (people.hasNext()) { 
      Person person = people.next(); 
      text.add(mapper.writeValueAsString(person)); 
    } 
    return text; 
  } 
} 

// 选出喜爱熊猫的人
JavaRDD<Person> result = input.mapPartitions(new ParseJson()).filter( 
  new LikesPandas()); 
JavaRDD<String> formatted = result.mapPartitions(new WriteJson()); 
formatted.saveAsTextFile(outfile);
```

#### CSV和TSV
CSV(Comma-separated Values)即逗号分隔值，TSV(Tab-separated values)即制表符分隔值，CSV文件每行都有固定数目的字段，字段间用逗号隔开，在制表符分隔值文件中用制表符隔开。

读取 CSV/TSV 数据需要先把文件当作普通文本文件来读取数据，再对数据进行处理。如果 CSV 的所有数据字段均没有包含换行符，可以使用 textFile() 读取并解析数据；如果在字段中嵌有换行符，就需要完整读入每个文件，然后解析各段。

使用 opencsv 库读取CSV文件的例子如下:

```java
// 一行行读取
import au.com.bytecode.opencsv.CSVReader; 
import Java.io.StringReader; 
... 
public static class ParseLine implements Function<String, String[]> { 
  public String[] call(String line) throws Exception { 
    CSVReader reader = new CSVReader(new StringReader(line)); 
    return reader.readNext(); 
  } 
} 
JavaRDD<String> csvFile1 = sc.textFile(inputFile); 
JavaPairRDD<String[]> csvData = csvFile1.map(new ParseLine());

// 读取整个文件
public static class ParseLine   implements FlatMapFunction<Tuple2<String, String>, String[]> { 
  public Iterable<String[]> call(Tuple2<String, String> file) throws Exception { 
    CSVReader reader = new CSVReader(new StringReader(file._2())); 
    return reader.readAll(); 
  } 
} 
JavaPairRDD<String, String> csvData = sc.wholeTextFiles(inputFile); 
JavaRDD<String[]> keyedRDD = csvData.flatMap(new ParseLine());
```

保存CSV文件的时候可以重用输出编码器来加速，由于y在csv中不需要输出字段名，因此为了保持输出一致，需要创建一种映射关系（做法是写一个函数，将各个字段转化为一定顺序的数组）。写入CSV文件的例子如下：

```java
csvData.map(x->{
             StringWriter stringWriter = new StringWriter();
             CSVWriter csvWriter = new CSVWriter(stringWriter);
             csvWriter.writeNext(x);
             csvWriter.close();
             return stringWriter.toString();
        }).saveAsTextFile("file;///sparkRS/csv");
```

#### sequenceFile
SequeceFile是Hadoop API提供的一种二进制文件支持。这种二进制文件直接将`<key, value>对`序列化到文件中。

由于 Hadoop 使用了一套自定义的序列化框架，因此 SequenceFile 是由实现 Hadoop 的 Writable接口的元素组成。常见的数据类型和它对应的Writeable类如下所示：

|Scala类型| Java类型 |Hadoop Writable类|
|-|-|-|
|Int |Integer| IntWritable 或 VIntWritable 2|
|Long| Long |LongWritable 或 VLongWritable 2|
|Float |Float |FloatWritable|
|Double| Double| DoubleWritable|
|Boolean| Boolean| BooleanWritable|
|Array[Byte]| byte[]| BytesWritable|
|String| String| Text|
|Array[T]| T[]| ArrayWritable<TW>|
|List[T] |List<T> |ArrayWritable<TW>|
|Map[A, B] |Map<A, B>| MapWritable<AW, BW>|

读取sequenceFile比较简单，在Java中只需要使用 sequenceFile(path, keyClass, valueClass, minPartitions)方法。比如，要从一个SequenceFile 中读取人员以及他们所见过的熊猫数目，按照上面基本类和Writable类的对应关系，keyClass 是 Text，而 valueClass 则是 IntWritable 或 VIntWritable(变长类型)。例子如下：

```java
public static class ConvertToNativeTypes implements 
  PairFunction<Tuple2<Text, IntWritable>, String, Integer> { 
  public Tuple2<String, Integer> call(Tuple2<Text, IntWritable> record) { 
    return new Tuple2(record._1.toString(), record._2.get()); 
  } 
} 
 
JavaPairRDD<Text, IntWritable> input = sc.sequenceFile(fileName, Text.class, 
  IntWritable.class); 
JavaPairRDD<String, Integer> result = input.mapToPair( 
  new ConvertToNativeTypes());
```

Java API中没有直接保存sequenceFile的方法，需要保存自定义Hadoop格式。

```java
public static class ConvertToWritableTypes implements 
  PairFunction<Tuple2<String, Integer>, Text, IntWritable> { 
  public Tuple2<Text, IntWritable> call(Tuple2<String, Integer> record) { 
    return new Tuple2(new Text(record._1), new IntWritable(record._2)); 
  } 
} 
 
JavaPairRDD<String, Integer> rdd = sc.parallelizePairs(input); 
JavaPairRDD<Text, IntWritable> result = rdd.mapToPair(new ConvertToWritableTypes()); 
result.saveAsHadoopFile(fileName, Text.class, IntWritable.class, 
  SequenceFileOutputFormat.class);
```

### 文件系统
#### 本地/“常规”文件系统
Spark 支持从本地文件系统中读取文件，不过它要求文件在集群中所有节点的相同路径下都可以找到。如果文件还没有放在集群中的所有节点上，可以在驱动器程序中从本地读取该文件而无
需使用整个集群，然后再调用 parallelize 将内容分发给工作节点。不过这种方式可能会比较慢。

#### Amazon S3
..

#### HDFS
..

### Spark SQL中的结构化数据

## Spark编程进阶
Spark提供了两种类型的共享变量：累加器（accumulator）与广播变量（broadcast variable）。累加器用来对信息进行聚合，而广播变量用来高效分发较大的对象。

### 累加器
累加器主要用于多个节点对一个变量进行共享性的操作，提供了将工作节点中的值聚合到驱动器程序中的简单语法。Accumulator只提供了累加的功能，只能累加，不能减少累加器只能在Driver端构建，并只能从Driver端读取结果，在Task端只能进行累加。

## spark sql
Spark sql提供了三种功能：

- Spark sql可以从各种结构化的数据源(例如JSON、Hive、Parquet等)中读取数据
- Spark sql不仅支持在Spark 程序内使用SQL 语句进行数据查询，也支持从类似商业智能软件Tableau 这样的外部工具中通过标准数据库连接器（JDBC/ODBC）连接SparkSQL 进行查询
- 当在Spark 程序内使用Spark SQL 时，Spark SQL 支持SQL 与常规的Python/Java/Scala代码高度整合

为了实现这些功能，Spark SQL 提供了一种特殊的RDD，叫作SchemaRDD。SchemaRDD是存放Row 对象的RDD，每个Row 对象代表一行记录。SchemaRDD 还包含记录的结构信息（即数据字段）。SchemaRDD 看起来和普通的RDD 很像，但是在内部，SchemaRDD 可以利用结构信息更加高效地存储数据。此外，SchemaRDD 还支持RDD 上所没有的一些新操作，比如运行SQL 查询。SchemaRDD 可以从外部数据源创建，也可以从查询结果或普通RDD 中创建。

> 注：RDD、DataFrame(SchemaRDD)、Dataset这三者都是spark提供的分布式数据集。它们出现的版本分别为：RDD (Spark1.0) —> SchemaRDD(Spark1.3后改名为Dataframe) —> Dataset(Spark1.6)。

### 连接Spark sql
Apache Hive 是Hadoop 上的SQL 引擎，Spark SQL 编译时可以包含Hive 支持，也可以不包含。如果要在Spark SQL 中包含Hive 的库，并不需要事先安装Hive。

连接带有Hive 支持的Spark SQL 的Maven依赖如下：

```mavan
groupId = org.apache.spark
artifactId = spark-hive_2.10
version = 1.2.0
```

如果不需要引入hive依赖，则可以使用spark-sql_2.10 来代替spark-hive_2.10。

当使用Spark SQL 进行编程时，根据是否使用Hive 支持，有两个不同的入口。推荐使用的入口是HiveContext，它可以提供HiveQL 以及其他依赖于Hive 的功能的支持。更为基础的SQLContext 则支持Spark SQL 功能的一个子集，子集中去掉了需要依赖于Hive 的功能。

最后，若要把Spark SQL 连接到一个部署好的Hive 上，你必须把hive-site.xml 复制到Spark 的配置文件目录中（$SPARK_HOME/conf）。

即使没有部署好Hive，Spark SQL 也可以运行。在这种情况下，Spark SQL 会在当前的工作目录中创建出自己的Hive 元数据仓库，叫作metastore_db。此外，如果此时尝试使用HiveQL 中的CREATE TABLE（并非CREATE EXTERNAL TABLE）语句来创建表，这些表会被放在默认的文件系统中的/user/hive/warehouse 目录中（如果classpath 中有配好的hdfs-site.xml，默认的文件系统就是HDFS，否则就是本地文件系统）。


### 在应用中使用Saprk sql
要在应用中使用Spark sql，需要基于已有的SparkContext 创建出一个HiveContext（或者SQLContext）。

#### 初始化Spark sql
在Java中初始化Spark sql的语句如下：

```java
JavaSparkContext ctx = new JavaSparkContext(...);
SQLContext sqlCtx = new HiveContext(ctx);
```

#### 基本查询示例
下面的例子显示了从JSON文件中读取一部分数据，将这部分数据注册为一张临时表并赋予该表一个名字，之后从该表中查询数据。

```java
SchemaRDD input = hiveCtx.jsonFile(inputFile);
// 注册输入的SchemaRDD
input.registerTempTable("tweets");
// 依据retweetCount（转发计数）选出推文
SchemaRDD topTweets = hiveCtx.sql("SELECT text, retweetCount FROM
tweets ORDER BY retweetCount LIMIT 10");
```

#### SchemaRDD
SchemaRDD是一个由Row对象组成的RDD，Row对象只是对基本数据类型(如整型和字符串型的封装)，它的本质是一个定长的字段数组，在Java/Scala中有标准的getter方法获取Row中各个字段的值。

SchemaRDD支持通过registerTempTable()方法注册为临时表，这样就可以通过HiveContext.sql或SQLContext.sql方法对其进行查询。这种临时表是HiveContext或者SQLContext中的临时变量，当应用退出时，这些临时变量就消失了。

SchemaRDD中可以存储的数据类型和基本编程语言的对应关系如下表所示：

|Spark SQL/HiveQL类型|Scala类型|Java类型|Python|
|-|-|-|-|
|TINYINT |Byte |Byte/byte| int/long ( 在-128 到127 之间)|
|SMALLINT |Short| Short/short| int/long ( 在-32768 到32767之间)|
|INT |Int |Int/int| int 或long|
|BIGINT |Long |Long/long| long|
|FLOAT| Float |Float /float|float|
|DOUBLE |Double |Double/double| float|
|DECIMAL |Scala.math.BigDecimal| java.math.BigDecimal| decimal.Decimal|
|STRING |String |String |string|
|BINARY |Array[Byte] |byte[] |bytearray|
|BOOLEAN| Boolean |Boolean/boolean| bool|
|TIMESTAMP| java.sql.TimeStamp| java.sql.TimeStamp| datetime.datetime|
|ARRAY<DATA_TYPE> |Seq |List| list、tuple 或array|
|MAP<KEY_TYPE, VAL_TYPE>| Map |Map| dict|
|STRUCT<COL1:COL1_TYPE, ...>| Row| Row |Row|

### 读取和存储数据
#### apache hive
在Java中读取hive中数据的例子如下：

```java
HiveContext hiveCtx = new HiveContext(sc);
SchemaRDD rows = hiveCtx.sql("SELECT key, value FROM mytable");
JavaRDD<Integer> keys = rdd.toJavaRDD().map(new Function<Row, Integer>() {
public Integer call(Row row) { return row.getInt(0); }
});
```

#### json
在Java中读取json的方法是使用jsonFile()方法，如下：

```java
SchemaRDD input = hiveCtx.jsonFile(jsonFile);
```

##### 将RDD转化为SchemaRDD
在Java中可以使用applySchema()方法将RDD转化为SchemaRDD，但是这个RDD 中的数据类型带有公有的getter 和setter 方法，并且可以被序列化。例子如下：

```java
class HappyPerson implements Serializable {
    private String name;
    private String favouriteBeverage;
    public HappyPerson() {}
    public HappyPerson(String n, String b) {
        name = n; favouriteBeverage = b;
    }
    public String getName() { return name; }
    public void setName(String n) { name = n; }
    public String getFavouriteBeverage() { return favouriteBeverage; }
    public void setFavouriteBeverage(String b) { favouriteBeverage = b; }
};
...
ArrayList<HappyPerson> peopleList = new ArrayList<HappyPerson>();
peopleList.add(new HappyPerson("holden", "coffee"));
JavaRDD<HappyPerson> happyPeopleRDD = sc.parallelize(peopleList);
SchemaRDD happyPeopleSchemaRDD = hiveCtx.applySchema(happyPeopleRDD,
HappyPerson.class);
happyPeopleSchemaRDD.registerTempTable("happy_people");
```

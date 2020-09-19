# `Java8`实战笔记

<!-- TOC -->

- [Java8实战笔记](#java8%E5%AE%9E%E6%88%98%E7%AC%94%E8%AE%B0)
    - [行为参数化](#%E8%A1%8C%E4%B8%BA%E5%8F%82%E6%95%B0%E5%8C%96)
        - [行为参数化的例子：筛选苹果](#%E8%A1%8C%E4%B8%BA%E5%8F%82%E6%95%B0%E5%8C%96%E7%9A%84%E4%BE%8B%E5%AD%90%E7%AD%9B%E9%80%89%E8%8B%B9%E6%9E%9C)
            - [第一次尝试：筛选绿苹果](#%E7%AC%AC%E4%B8%80%E6%AC%A1%E5%B0%9D%E8%AF%95%E7%AD%9B%E9%80%89%E7%BB%BF%E8%8B%B9%E6%9E%9C)
            - [第二次尝试：把颜色作为参数](#%E7%AC%AC%E4%BA%8C%E6%AC%A1%E5%B0%9D%E8%AF%95%E6%8A%8A%E9%A2%9C%E8%89%B2%E4%BD%9C%E4%B8%BA%E5%8F%82%E6%95%B0)
            - [第三次尝试：对想到的每个属性做筛选](#%E7%AC%AC%E4%B8%89%E6%AC%A1%E5%B0%9D%E8%AF%95%E5%AF%B9%E6%83%B3%E5%88%B0%E7%9A%84%E6%AF%8F%E4%B8%AA%E5%B1%9E%E6%80%A7%E5%81%9A%E7%AD%9B%E9%80%89)
            - [第四次尝试：根据抽象条件筛选](#%E7%AC%AC%E5%9B%9B%E6%AC%A1%E5%B0%9D%E8%AF%95%E6%A0%B9%E6%8D%AE%E6%8A%BD%E8%B1%A1%E6%9D%A1%E4%BB%B6%E7%AD%9B%E9%80%89)
            - [第五次尝试：使用匿名类](#%E7%AC%AC%E4%BA%94%E6%AC%A1%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8%E5%8C%BF%E5%90%8D%E7%B1%BB)
            - [第六次尝试：使用 Lambda 表达式](#%E7%AC%AC%E5%85%AD%E6%AC%A1%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8-lambda-%E8%A1%A8%E8%BE%BE%E5%BC%8F)
            - [第七次尝试：将List类型抽象化](#%E7%AC%AC%E4%B8%83%E6%AC%A1%E5%B0%9D%E8%AF%95%E5%B0%86list%E7%B1%BB%E5%9E%8B%E6%8A%BD%E8%B1%A1%E5%8C%96)
    - [lambda表达式](#lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F)
        - [lambda表达式的基本语法](#lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95)
        - [函数式接口](#%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3)
            - [Predicate接口使用示例](#predicate%E6%8E%A5%E5%8F%A3%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)
            - [Function接口使用示例](#function%E6%8E%A5%E5%8F%A3%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)
            - [Consumer接口使用示例](#consumer%E6%8E%A5%E5%8F%A3%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)
            - [Supplier接口使用示例](#supplier%E6%8E%A5%E5%8F%A3%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)
            - [原始类型特化接口](#%E5%8E%9F%E5%A7%8B%E7%B1%BB%E5%9E%8B%E7%89%B9%E5%8C%96%E6%8E%A5%E5%8F%A3)
        - [类型检查和类型推断](#%E7%B1%BB%E5%9E%8B%E6%A3%80%E6%9F%A5%E5%92%8C%E7%B1%BB%E5%9E%8B%E6%8E%A8%E6%96%AD)
        - [方法引用](#%E6%96%B9%E6%B3%95%E5%BC%95%E7%94%A8)
            - [构造函数引用](#%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0%E5%BC%95%E7%94%A8)
                - [无参构造函数](#%E6%97%A0%E5%8F%82%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
                - [一个参数的构造函数](#%E4%B8%80%E4%B8%AA%E5%8F%82%E6%95%B0%E7%9A%84%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
                - [两个参数的构造函数](#%E4%B8%A4%E4%B8%AA%E5%8F%82%E6%95%B0%E7%9A%84%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)
        - [复合lambda表达式](#%E5%A4%8D%E5%90%88lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F)
    - [流](#%E6%B5%81)
        - [流的特性](#%E6%B5%81%E7%9A%84%E7%89%B9%E6%80%A7)
        - [流与集合](#%E6%B5%81%E4%B8%8E%E9%9B%86%E5%90%88)
            - [内部迭代和外部迭代](#%E5%86%85%E9%83%A8%E8%BF%AD%E4%BB%A3%E5%92%8C%E5%A4%96%E9%83%A8%E8%BF%AD%E4%BB%A3)
        - [流操作](#%E6%B5%81%E6%93%8D%E4%BD%9C)
        - [Stream API](#stream-api)
            - [筛选和切片](#%E7%AD%9B%E9%80%89%E5%92%8C%E5%88%87%E7%89%87)
                - [用谓词进行筛选](#%E7%94%A8%E8%B0%93%E8%AF%8D%E8%BF%9B%E8%A1%8C%E7%AD%9B%E9%80%89)
                - [筛选各异的元素](#%E7%AD%9B%E9%80%89%E5%90%84%E5%BC%82%E7%9A%84%E5%85%83%E7%B4%A0)
                - [截短流](#%E6%88%AA%E7%9F%AD%E6%B5%81)
                - [跳过元素](#%E8%B7%B3%E8%BF%87%E5%85%83%E7%B4%A0)
            - [映射](#%E6%98%A0%E5%B0%84)
                - [对流中每一个元素应用函数](#%E5%AF%B9%E6%B5%81%E4%B8%AD%E6%AF%8F%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0%E5%BA%94%E7%94%A8%E5%87%BD%E6%95%B0)
                - [流的扁平化](#%E6%B5%81%E7%9A%84%E6%89%81%E5%B9%B3%E5%8C%96)
            - [查找和匹配](#%E6%9F%A5%E6%89%BE%E5%92%8C%E5%8C%B9%E9%85%8D)
                - [检查谓词是否至少匹配一个元素](#%E6%A3%80%E6%9F%A5%E8%B0%93%E8%AF%8D%E6%98%AF%E5%90%A6%E8%87%B3%E5%B0%91%E5%8C%B9%E9%85%8D%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0)
                - [检查谓词是否匹配所有元素](#%E6%A3%80%E6%9F%A5%E8%B0%93%E8%AF%8D%E6%98%AF%E5%90%A6%E5%8C%B9%E9%85%8D%E6%89%80%E6%9C%89%E5%85%83%E7%B4%A0)
                - [查找元素](#%E6%9F%A5%E6%89%BE%E5%85%83%E7%B4%A0)
                - [查找第一个元素](#%E6%9F%A5%E6%89%BE%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0)
            - [归约reduce](#%E5%BD%92%E7%BA%A6reduce)
                - [有状态和无状态](#%E6%9C%89%E7%8A%B6%E6%80%81%E5%92%8C%E6%97%A0%E7%8A%B6%E6%80%81)
            - [数值流](#%E6%95%B0%E5%80%BC%E6%B5%81)
                - [原始类型特化流](#%E5%8E%9F%E5%A7%8B%E7%B1%BB%E5%9E%8B%E7%89%B9%E5%8C%96%E6%B5%81)
                    - [映射到数值流](#%E6%98%A0%E5%B0%84%E5%88%B0%E6%95%B0%E5%80%BC%E6%B5%81)
                    - [转换回对象流](#%E8%BD%AC%E6%8D%A2%E5%9B%9E%E5%AF%B9%E8%B1%A1%E6%B5%81)
                    - [默认值](#%E9%BB%98%E8%AE%A4%E5%80%BC)
                - [数值范围](#%E6%95%B0%E5%80%BC%E8%8C%83%E5%9B%B4)
            - [构件流](#%E6%9E%84%E4%BB%B6%E6%B5%81)
                - [由值创建流](#%E7%94%B1%E5%80%BC%E5%88%9B%E5%BB%BA%E6%B5%81)
                - [由数组创建流](#%E7%94%B1%E6%95%B0%E7%BB%84%E5%88%9B%E5%BB%BA%E6%B5%81)
                - [由文件生成流](#%E7%94%B1%E6%96%87%E4%BB%B6%E7%94%9F%E6%88%90%E6%B5%81)
                - [由函数生成流：创建无限流](#%E7%94%B1%E5%87%BD%E6%95%B0%E7%94%9F%E6%88%90%E6%B5%81%E5%88%9B%E5%BB%BA%E6%97%A0%E9%99%90%E6%B5%81)
                    - [iterate](#iterate)
                    - [generate](#generate)
    - [用流收集数据](#%E7%94%A8%E6%B5%81%E6%94%B6%E9%9B%86%E6%95%B0%E6%8D%AE)
        - [预定义收集器](#%E9%A2%84%E5%AE%9A%E4%B9%89%E6%94%B6%E9%9B%86%E5%99%A8)
        - [规约和汇总](#%E8%A7%84%E7%BA%A6%E5%92%8C%E6%B1%87%E6%80%BB)
            - [counting收集器](#counting%E6%94%B6%E9%9B%86%E5%99%A8)
            - [查找流的最大值和最小值](#%E6%9F%A5%E6%89%BE%E6%B5%81%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC%E5%92%8C%E6%9C%80%E5%B0%8F%E5%80%BC)
            - [汇总](#%E6%B1%87%E6%80%BB)
            - [连接字符串](#%E8%BF%9E%E6%8E%A5%E5%AD%97%E7%AC%A6%E4%B8%B2)
        - [分组](#%E5%88%86%E7%BB%84)
            - [多级分组](#%E5%A4%9A%E7%BA%A7%E5%88%86%E7%BB%84)
            - [按子组收集数据](#%E6%8C%89%E5%AD%90%E7%BB%84%E6%94%B6%E9%9B%86%E6%95%B0%E6%8D%AE)
                - [把收集器的结果转换为另一种类型](#%E6%8A%8A%E6%94%B6%E9%9B%86%E5%99%A8%E7%9A%84%E7%BB%93%E6%9E%9C%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%8F%A6%E4%B8%80%E7%A7%8D%E7%B1%BB%E5%9E%8B)
                - [与groupingBy联合使用的其他收集器的例子](#%E4%B8%8Egroupingby%E8%81%94%E5%90%88%E4%BD%BF%E7%94%A8%E7%9A%84%E5%85%B6%E4%BB%96%E6%94%B6%E9%9B%86%E5%99%A8%E7%9A%84%E4%BE%8B%E5%AD%90)
            - [分区](#%E5%88%86%E5%8C%BA)
                - [多级分区](#%E5%A4%9A%E7%BA%A7%E5%88%86%E5%8C%BA)
                - [一个例子：将数字按照质数和非质数进行分类](#%E4%B8%80%E4%B8%AA%E4%BE%8B%E5%AD%90%E5%B0%86%E6%95%B0%E5%AD%97%E6%8C%89%E7%85%A7%E8%B4%A8%E6%95%B0%E5%92%8C%E9%9D%9E%E8%B4%A8%E6%95%B0%E8%BF%9B%E8%A1%8C%E5%88%86%E7%B1%BB)
                - [自定义收集器](#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%94%B6%E9%9B%86%E5%99%A8)
                - [建立新的结果容器：supplier方法](#%E5%BB%BA%E7%AB%8B%E6%96%B0%E7%9A%84%E7%BB%93%E6%9E%9C%E5%AE%B9%E5%99%A8supplier%E6%96%B9%E6%B3%95)
                - [将元素添加到结果容器：accumulator方法](#%E5%B0%86%E5%85%83%E7%B4%A0%E6%B7%BB%E5%8A%A0%E5%88%B0%E7%BB%93%E6%9E%9C%E5%AE%B9%E5%99%A8accumulator%E6%96%B9%E6%B3%95)
                - [对结果容器应用最终转换：finisher方法](#%E5%AF%B9%E7%BB%93%E6%9E%9C%E5%AE%B9%E5%99%A8%E5%BA%94%E7%94%A8%E6%9C%80%E7%BB%88%E8%BD%AC%E6%8D%A2finisher%E6%96%B9%E6%B3%95)
                - [合并两个结果容器：combiner方法](#%E5%90%88%E5%B9%B6%E4%B8%A4%E4%B8%AA%E7%BB%93%E6%9E%9C%E5%AE%B9%E5%99%A8combiner%E6%96%B9%E6%B3%95)
                - [characteristics方法](#characteristics%E6%96%B9%E6%B3%95)
    - [并行数据处理与性能](#%E5%B9%B6%E8%A1%8C%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B8%8E%E6%80%A7%E8%83%BD)
        - [串行流和并行流的转换](#%E4%B8%B2%E8%A1%8C%E6%B5%81%E5%92%8C%E5%B9%B6%E8%A1%8C%E6%B5%81%E7%9A%84%E8%BD%AC%E6%8D%A2)
        - [正确使用并行流](#%E6%AD%A3%E7%A1%AE%E4%BD%BF%E7%94%A8%E5%B9%B6%E8%A1%8C%E6%B5%81)

<!-- /TOC -->

## 行为参数化
“行为参数化”的含义，从字面上来看就是：把行为当做参数传递给方法，与之对应的是“值参数化”。这里的“行为”其实就是一段代码。比如说，需要写两个只有几行代码不同的方法，现在只需要把不同的那部分代码作为参数传递进去就行了。一言以蔽之，“行为参数化”就是让方法接受多种行为（或策略）作为参数，并在内部使用，来完成不同的行为

行为参数化和设计模式中的“策略模式”很像，该模式的思想是这样的：定义一个函数式接口(定义一个抽象方法的接口)，根据不同的需要来实现这个接口，然后把不同实现类的实例当做参数加入到方法中。

行为参数化可以很好的应对不断变更的需求，使代码更优雅。

本文用到的代码在[java8](https://github.com/adamhand/java8)。

### 行为参数化的例子：筛选苹果

#### 第一次尝试：筛选绿苹果
有个应用程序是帮助农民了解自己的库存的，这位农民一开始想有一个查找库存中所有绿色苹果的功能，根据需求可以写出如下方案：

```java
private static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ("green".equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```

#### 第二次尝试：把颜色作为参数

之后农民扩展需求，还想找出红色的、黄色的、浅绿色的苹果，这时，想到一个好的办法，将参数抽象化，即将`color`作为参数进行传递，如下：

```java
private static List<Apple> filterApplesByColor(List<Apple> inventory, String color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (color.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```
然而，农民朋友又有了新需求：区分轻的苹果和重的苹果，重的苹果一般大于`150`克，有了改变颜色的教训，这次想到了后面可能会改变重量，于是写下了如下解决方案：

```java
private static List<Apple> filterHeavyApples(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
```
想法不多，但是`filterHeavyApples()`方法大部分代码和`filterApplesByColor()`重复了，显然不是一个好的解决方法。

#### 第三次尝试：对想到的每个属性做筛选
将颜色和重量都作为参数，并且设置`flag`来区分需要筛选哪个属性。如下。

```java
public static List<Apple> filterApples(List<Apple> inventory, String color, int weight, boolean flag){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if((flag && apple.getColor.equals(color)) ||
            (!flag && apple.getWeight() > weight)){
            result.add(apple);
        }
    }
    return result;
}
```
使用起来如下：

```java
List<Apple> greenApples = filterApples(inventory, "green", 0, ture);
List<Apple> heavyApples = filterApples(inventory, "", 150, false);
```
这个方法也不完美，不仅使用时参数比较乱，而且当新加入筛选条件时还是得改源代码。

这时，就需要“行为参数化”上场了，首先对选择标准建模：现在考虑的是苹果，需要根据`Apple`的某些属性（比如它是绿色的吗？重量超过`150`克吗？）来返回一个boolean值，称为**谓词**（即一个返回`boolean`值得函数）。

#### 第四次尝试：根据抽象条件筛选
首先定义一个接口对选择标准建模：

```java
public interface ApplePredicate{
    boolean test (Apple apple);
}
```

接着使用`ApplePredicate`的多个实现代表不同的选择标准:

```java
// 筛选重苹果
public class AppleHeavyWeightPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

// 筛选绿苹果
public class AppleGreenColorPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}
```
若还需要找出所有重量超过`150`克的红苹果，只需要创建一个类实现`ApplePredicate`就行了。如下。

```java
public class AppleRedHeavyPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150 &&
                "red".equals(apple.getColor());
    }
}
```

利用`ApplePredicate`改过之后，`filter`方法看起来就是这样的：

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate predicate) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (predicate.test(apple)) {
            result.add(apple);
        }
    }

    return result;
}
```

到目前为止，行为参数化的功能已经基本实现，但是还有不足之处。若某个`predicate`只用一次，就没必要新建一个类来实现`ApplePredicate`接口了，而是可以使用匿名类。

#### 第五次尝试：使用匿名类
使用匿名类的例子如下：

```java
List<Apple> redApples = filterApples(inventory, new Applepredicate(){
    public boolean test(Apple apple){
        return "red".equals(apple.getColor());
    }
});
```
然而，对于精益求精的人来说，这还不够好，因为匿名类写起来很麻烦，且有时候不易理解。

#### 第六次尝试：使用 Lambda 表达式
上面的匿名类的例子可以使用`lambda`表达式写为如下的形式：

```java
List<Apple> result = 
    filterApples(inventory, (Apple apple) -> "red".equals(apple.getColor()));
```

#### 第七次尝试：将List类型抽象化
在通往抽象的道路上，还可以更近一步。目前`filterApples`方法还只适用于`Apple`。还可以将`List`类型抽象化，从而超越你眼前要处理的问题：

```java
public interface Predicate<T>{
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> result = new ArrayList<>();
    for(T e:list){
        if(p.test(e)){
            result.add(e);
        }
    }
    return result;
}
```
现在可以把`filter`方法用在香蕉、桔子、`Integer`或者是`String`的列表上了。这里有一些使用`Lambda`表达式的例子：

```java
List<Apple> redApples = 
    filter(inventory, (Apple apple) -> "red".equals(apple.getColor()));
List<Integer> evenNumbers = 
    filter(numbers, (Integer i) -> i % 2 == 0);
```

## lambda表达式
`lambda`表达式可以看成是对匿名类的一种简写，比如，在`java8`之前，使用比较器的匿名类写法时可以这样写：

```java
Arrays.sort(nums, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1 - o2;
    }
});
```
这种写法比较臃肿，实际之需要一个`compare`方法，却需要传递一个`Comparator`对象进去。那么，能不能之传递一个方法进去呢？`lambda`表达式的出现使这个功能称为可能。`lambda`表达式可以看成是一种**匿名函数**：它没有名称，但有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。并且，**`lambda`表达式可以作为参数传递给方法或存储在变量中**。

上面的`sort`代码用`lambda`表达式写出来就是：

```java
Arrays.sort(nums, (Integer o1, Integer o2) -> o1.compareTo(o2));
```

### lambda表达式的基本语法
`lambda`表达式的基本语法是：

```java
(parameters) -> expression 

(parameters) -> { statements; }
```
包括：参数列表、箭头(`->`)和`lambda`主体三部分。当只有一条`expression`时，`{}`和`return`可以省略，但是有多条时，不能省略。

### 函数式接口
`lambda`表达式当使用必须需要“函数式接口”，即**只有一个抽象方法的接口**。成为函数式接口可以使用注解`@FunctionalInterface`。上面的`Comparator<T>`接口，还有`Runnable`接口和`Callable<V>`接口都是函数式接口。

`java8`开始，接口可以拥有默认方法(即在类没有对方法进行实现时，其主体为方法提供默认实现的方法)。但是，**哪怕有很多默认方法，只要接口只定义了一个抽象方法，它就仍然是一个函数式接口。**

有了函数式接口，便可以使用`lambda`表达式直接以内联的形式为函数式接口的抽象，并把整个表达式(或者说函数式接口的一个具体的实现实例)作为函数式接口的实例。

函数式接口的抽象方法的签名称为函数描述符。所以为了应用不同的`Lambda`表达式，需要一套能够描述常见函数描述符的函数式接口。`java8`中有几个常用的函数式接口：`Predicate 、 Consumer、Function`和`Supplier`。它们分别表述如下：

|函数式接口|参数类型|返回类型|用途|
|-|-|-|-|
|Predicate < T>断定型接口|	T|	boolean	|确定类型为T的对象是否满足某约束,并返回Boolean值。包含方法boolean test(T t)|
|Function < T,R>函数型接口|	T	|R	|如果需要定义一个Lambda，将输入对象的信息映射到输出，就可以使用这个接口。包含方法apply(T t)|
|Consumer < T>消费型接口|	T	|void||如果需要访问类型 T 的对象，并对其执行某些操作，就可以使用这个接口。包含方法accept(T t)|
|Supplier < T>供给型接口|	无|	T|	返回类型为T的对象,包含方法:T get()|

示例如下。

#### Predicate接口使用示例
判断输入的字符串是否为空：

```java
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T s :  list) {
        if (p.test(s)) {
            result.add(s);
        }
    }
    return result;
}

Predicate<String> noneEmptyStringPredicate = (String s) -> (!s.isEmpty());
List<String> noneEmpty = PredicateDemo.filter(Arrays.asList("a", "b", "c"),
        noneEmptyStringPredicate);
```

#### Function接口使用示例
输入字符串，输出字符串的长度。即将字符串类型映射为整型。

```java
    public static <T, R>List<R> map(List<T> list, Function<T, R> f) {
        List<R> result = new ArrayList<>();
        for (T t : list) {
            result.add(f.apply(t));
        }
        return result;
    }
            List<Integer> l = map(
                Arrays.asList("lambdas", "in", "action"),
                (String s) -> s.length()
        );
```
上面的代码中的`lambda`表达式其实还可以简化，如果在编译器中加入了诸如阿里的代码检查工具，对`(String s) -> s.length()`会提示进行修改：`Can be replaced with method reference less`，即可以使用“函数引用”来简化，可以将这句改为`String::length`。“函数引用”在下面会进行记录。

#### Consumer接口使用示例
对输入的数组进行遍历。

```java
public static <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accept(t);
    }
}

forEach(
        Arrays.asList(1, 2, 3, 4, 5),
        (Integer i) -> System.out.println(i)
);
```

#### Supplier接口使用示例
由名字可以看出，这个接口的作用是作为一个“提供者”，可`Consumer`接口的作用相反，它可以提供一个对象。比如：

```java
private static String getString(Supplier<String> stringSupplier) {
    return stringSupplier.get();
}

System.out.println(
        getString(() -> "hello" + " world")
);
```

#### 原始类型特化接口
由于`java8`中提供的上述函数式接口用到了泛型，只能绑定到到引用类型，不能绑定到原始类型(比如`int`、`double`等)。所以下面的语句会涉及到自动拆箱和装箱功能：

```java
List<Integer> list = new ArrayList<>(); 
for (int i = 300; i < 400; i++){ 
    list.add(i); 
} 
```

会导致性能受到影响。因此，`java8`提供了原始类型特定化接口，即针对不同的原始类型，都有与之对应的函数式接口。如下：

|函数式接口|函数描述符|原始类型特化|
|-|-|-|
|Predicate<T>|  T->boolean | IntPredicate,LongPredicate, DoublePredicate |
|Consumer<T> | T->void  |IntConsumer,LongConsumer, DoubleConsumer |
|Function<T,R>|  T->R  |IntFunction<R>, IntToDoubleFunction, IntToLongFunction, LongFunction<R>, LongToDoubleFunction, LongToIntFunction, DoubleFunction<R>, ToIntFunction<T>, ToDoubleFunction<T>, ToLongFunction<T> |
|Supplier<T>|   ()->T  | BooleanSupplier,IntSupplier, LongSupplier,DoubleSupplier |
|UnaryOperator<T> | T->T|  IntUnaryOperator,LongUnaryOperator, DoubleUnaryOperator|
|BinaryOperator<T>| (T,T)->T | IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator |
|BiPredicate<L,R> | (L,R)->boolean||
|BiConsumer<T,U> | (T,U)->void | ObjIntConsumer<T>, ObjLongConsumer<T>,ObjDoubleConsumer<T>| 
|BiFunction<T,U,R>| (T,U)->R | ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>|

一个例子如下：

```java
// 避免了装箱
IntPredicate evenNumbers = (int i) -> i % 2 == 0; 
evenNumbers.test(1000); 

// 存在装箱
Predicate<Integer> oddNumbers = (Integer i) -> i % 2 == 1;    
oddNumbers.test(1000); 
```

### 类型检查和类型推断
在将`lambda`表达式和函数式接口进行匹配的时候，会将表达式的签名(参数类型，返回值等)和接口的函数描述符进行匹配。`Java`编译器会从上下文(目标类型)推断出用什么函数式接
口来配合`Lambda`表达式，也可以推断出适合`Lambda`的签名。因此，可以在`Lambda`语法中省去标注参数类型。如下：

```java
// 无类型推断
Comparator<Apple> c = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); 

// 有类型推断
Comparator<Apple> c = 
    (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); 
```

### 方法引用
方法引用可以看成是`lambda`表达式的一种快捷写法。它的基本思想是，如果一个Lambda代表的只是“直接调用这个方法”，那最好还是用名称来调用它，而不是去描述如何调用它。方法引用的一般格式是：`目标引用::方法名称`。例如`Apple::getWeight`就是引用了`Apple`类中定义的方法`getWeight`。

方法应用主要有三类：

- 类::静态方法。如`Integer::parseInt`
- 类::实例方法。如`String::length`
- 对象::实例方法。假设有一个局部变量`expensiveTransaction`用于存放`Transaction`类型的对象，它支持实例方法`getValue`，则可以写为`expensiveTransaction::expensiveTransaction`

#### 构造函数引用
对于一个现有构造函数，可以利用它的名称和关键字`new`来创建它的一个引用：`ClassName::new`。例如，对于有一个`Apple`类如下：

```java
public class Apple {
    private int weight;   // 重量
    private String color; // 颜色

    public Apple() {}

    public Apple(int weight) {
        this.weight = weight;
    }

    public Apple(int weight, String color) {
        this.weight = weight;
        this.color = color;
    }
}
```

它包括三个构造函数：一个无参函数，一个含有一个参数，一个含有两个参数。

##### 无参构造函数
对于无参构造函数，可以使用`Supplier`接口与之匹配，生成一个`Apple`对象。如下：

```java
Supplier<Apple> supplier = Apple::new;
Apple apple =  supplier.get();
```
等同于：

```java
Supplier<Apple> supplier = () -> new Apple();
Apple apple = supplier.get();
```

##### 一个参数的构造函数
对于只含有一个参数的构造函数，可以使用`Function`接口与之匹配。如下：

```java
Function<Integer, Apple> function = Apple::new;
Apple apple = function.apply(150);
```
等价于：

```java
Function<Integer, Apple> function = (weight) -> new Apple(weight);
Apple apple = function.apply(150);
```
下面的代码中，根据传入的存放着整数的`List`来为每个新建的苹果设定重量：

```java
public static List<Apple> map(List<Integer> list, Function<Integer, Apple> f) {
    List<Apple> result = new ArrayList<>();
    for (Integer w : list) {
        result.add(f.apply(w));
    }
    return result;
}

List<Integer> weights = Arrays.asList(100, 200, 150, 230);
List<Apple> apples = map(weights, Apple::new);
```

##### 两个参数的构造函数
可以使用`BiFunction`来匹配这样的构造函数，如下：

```java
BiFunction<Integer, String, Apple> biFunction = Apple::new;
Apple apple = biFunction.apply(100, "green");
```

下面的代码，可以根据传入的`String`类型的水果种类名和`Integer`类型的水果重量来得到不同种类、颜色和重量的水果。

```java
static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
static {
    map.put("apple", Apple::new);
    map.put("orange", Orange::new);
}

public static Fruit giveMeFruit(String fruit, Integer weight) {
    return map.get(fruit.toLowerCase())
            .apply(weight);
}
```

### 复合lambda表达式
[待续]

## 流
允许以声明性方式处理数据集合(说明想要完成什么，而不是说明如何去做)。可以把它们看成遍历数据集的高级迭代器。此外，流还可以透明地并行处理。

假如目前有一个返回低热量菜肴的名称的需求，按照之前的写法，大概是这样的：

```java
List<Dish> lowerCaloricDishes = new ArrayList<>();
for (Dish d : menu) {
    if (d.getCalories() < 400) {
        lowerCaloricDishes.add(d);
    }
}
Collections.sort(lowerCaloricDishes, new Comparator<Dish>() {
    @Override
    public int compare(Dish o1, Dish o2) {
        return Integer.compare(o1.getCalories(), o2.getCalories());
    }
});
List<String> lowerCaloricDishesName = new ArrayList<>();
for (Dish d : lowerCaloricDishes) {
    lowerCaloricDishesName.add(d.getName());
}
```
而使用流的写法如下：

```java
List<String> lowerCaloricDishesName = menu
        .stream()
        .filter(d -> d.getCalories() < 400)
        .sorted(Comparator.comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(Collectors.toList());
```
可以看到，代码的可读性更好了，因为是以声明的方式进行编写的，同时，可以将每一个声明进行链接。如果想利用多核架构并行执行这段代码，只需要把`menu.stream()`改成`menu.parallelStream()`。

上述代码用到了例子如下：

```java
public class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return this.name;
    }

    public enum Type { MEAT, FISH, OTHER }
}

List<Dish> menu = Arrays.asList(
        new Dish("pork", false, 800, Dish.Type.MEAT),
        new Dish("beef", false, 700, Dish.Type.MEAT),
        new Dish("chicken", false, 400, Dish.Type.MEAT),
        new Dish("french fries", true, 530, Dish.Type.OTHER),
        new Dish("rice", true, 350, Dish.Type.OTHER),
        new Dish("season fruit", true, 120, Dish.Type.OTHER),
        new Dish("pizza", true, 550, Dish.Type.OTHER),
        new Dish("prawns", false, 300, Dish.Type.FISH),
        new Dish("salmon", false, 450, Dish.Type.FISH) );
```

### 流的特性
流的一个简短的定义是“从支持数据处理操作的源生成的元素序列”。这个定义中包含几个关键词：

- 元素序列：流提供了一个接口，可以访问特定元素类型的一组有序值
- 源：流会使用一个提供数据的源，如集合、数组或输入/输出资源
- 数据处理操作——流的数据处理功能支持类似于数据库的操作，以及函数式编程语言中的常用操作，如`filter、map、reduce、find、match、sort`等。流操作可以顺序执行，也可并行执行

此外，流操作有两个重要的特点:

-  流水线：很多流操作本身会返回一个流，这样多个操作就可以链接起来，形成一个大的流水线
- 内部迭代：与使用迭代器显式迭代的集合不同，流的迭代操作是在背后进行的

### 流与集合
流与集合的差异，在于**什么时候进行计算**。集合中的每个元素都得先算出来才能添加到集合中，它更偏重于存储，是静态的概念。而只有在消费者要求的时候才会计算值，更偏重于对数据的计算和处理。**和迭代器类似，流只能遍历一次**。

#### 内部迭代和外部迭代
使用 `Collection` 接口需要用户去做迭代(比如用 `for-each`)，这称为外部迭代。相反，`Streams`库使用内部迭代——它帮你把迭代做了，还把得到的流值存在了某个地方，只要给出
一个函数说要干什么就可以了。

内部迭代时，项目可以透明地并行处理，或者用更优化的顺序进行处理。**`Streams`库的内部迭代可以自动选择一种适合你硬件的数据表示和并行实现**。而使用外部迭代时，需要使用者自己管理所有的并行问题。

### 流操作
流操作基本可以分为两类：

- 中间操作：例如`filter`、`map`和`limit`可以连成一条流水线。流的流水线背后的理念类似于**构建器模式**
- 终端操作：例如`collect`触发流水线执行并关闭它。除非流水线上触发一个终端操作，否则中间操作不会执行任何处理。终端操作会从流的流水线生成结果。其结果是任何不是流的值，比如`List、Integer`，甚至`void`

由此总结，流的使用一般包括三件事：

- 一个数据源（如集合）来执行一个查询
- 一个中间操作链，形成一条流的流水线
- 一个终端操作，执行流水线，并能生成结果

### Stream API
`Stream API`提供的常见的中间操作如下：

|操作  |类型  |返回类型  |操作参数 | 函数描述符|
|-|-|-|-|-|
|filter | 中间|  Stream<T> | Predicate<T> | T -> boolean|
|map | 中间  |Stream<R> | Function<T, R> | T -> R |
|limit|  中间 | Stream<T> |||    
|sorted |中间  |Stream<T>  |Comparator<T> | (T, T) -> int |
|distinct|  中间  |Stream<T> ||

常见的终端操作如下：

|操作|类型|目的|
|-|-|-|
|forEach | 终端 | 消费流中的每个元素并对其应用 Lambda。这一操作返回 void  |
|count | 终端 | 返回流中元素的个数。这一操作返回 long  |
|collect | 终端|  把流归约成一个集合，比如 List 、 Map 甚至是 Integer|

可以使用这些`API`实现如筛选、切片、映射、查找、匹配和归约等操作。除此之外，还有一些其他流，比如数值流、文件流、数组流和无限流。

#### 筛选和切片

##### 用谓词进行筛选
流支持使用谓词进行筛选。比如`filter`方法，会接受一个谓词，返回一个包含所有符合谓词的元素的流。例如，选出所有的素菜可以使用下面的写法：

```java
List<Dish> vegetarianMenu = menu.stream() 
                                .filter(Dish::isVegetarian) 
                                .collect(toList());
```

##### 筛选各异的元素
`distinct`方法会返回一个元素各异(根据流所生成元素的`hashCode`和`equals`方法实现)的流。以下代码会筛选出列表中所有的偶数，并确保没有重复。

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4); 
numbers.stream()  
       .filter(i -> i % 2 == 0) 
       .distinct() 
       .forEach(System.out::println);
```

##### 截短流
`limit(n)`方法会返回一个不超过给定长度的流。下面的代码是选出热量超过`300`卡路里的头三道菜:

```java
List<Dish> dishes = menu.stream() 
                        .filter(d -> d.getCalories() > 300) 
                        .limit(3) 
                        .collect(toList());
```
因为有了截短流，所以`filter`方法只会选出符合谓词的头三个元素，然后就返回，而不是将所有符合谓词的元素都选择出来。


##### 跳过元素
`skip(n)`方法，返回一个扔掉了前`n`个元素的流。如果流中元素不足`n`个，则返回一个空流。下面的代码将跳过超过`300`卡路里的头两道菜，并返回剩下的。

```java
List<Dish> dishes = menu.stream() 
                        .filter(d -> d.getCalories() > 300) 
                        .skip(2) 
                        .collect(toList()); 
```

#### 映射
`Stream API`提供了从某些对象中选择信息的方法：`map`和`flatmap`。

##### 对流中每一个元素应用函数
`map`方法会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素。下面的代码是找出每道菜的名称有多长。

```java
List<Integer> dishNameLengths = menu.stream() 
                                    .map(Dish::getName) 
                                    .map(String::length) 
                                    .collect(toList()); 
```

##### 流的扁平化
`flatMap`实现了一种叫做“扁平流”的流，即把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。例子如下。

```java
List<String> words = Arrays.asList("Java 8", "Lambdas", "In", "Action");
List<String> uniqueCharacters = 
    words.stream() 
         // 将每个单词转换为由其字母构成的数组
         .map(w -> w.split(""))
         // 将各个生成流扁平化为单个流
         .flatMap(Arrays::stream)  
         .distinct() 
         .collect(Collectors.toList()); 
```
上述代码的作用是对于一张单词表，返回一张列表，列出里面各不相同的字符。

下面的例子是，给定两个数字列表，返回所有总和能被`3`整除的数对。例如，给定列表`[1, 2, 3]`和列表`[3, 4]`，应该返回`[(2, 4), (3, 3)]`。

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3); 
List<Integer> numbers2 = Arrays.asList(3, 4); 
List<int[]> pairs = 
    numbers1.stream() 
            .flatMap(i -> 
                       numbers2.stream() 
                               .filter(j -> (i + j) % 3 == 0) 
                               .map(j -> new int[]{i, j}) 
                    ) 
            .collect(toList()); 
```

#### 查找和匹配
`Stream API`通过`allMatch、anyMatch、noneMatch、findFirst`和`findAny`方法查看数据集中的某些元素是否匹配一个给定的属性。

##### 检查谓词是否至少匹配一个元素
`anyMatch`方法可以回答“流中是否有一个元素能匹配给定的谓词”。比如，可以用它来看看菜单里面是否有素食可选择：
```java
if(menu.stream().anyMatch(Dish::isVegetarian)){ 
    System.out.println("The menu is (somewhat) vegetarian friendly!!"); 
}
```
`anyMatch`方法返回一个``boolean`，因此是一个终端操作。

##### 检查谓词是否匹配所有元素
`allMatch`和`noneMatch`用于查看流中的元素是否都能/不能匹配给定的谓词。

```java
boolean isHealthy = menu.stream() 
                        .allMatch(d -> d.getCalories() < 1000);

boolean isHealthy = menu.stream() 
                        .noneMatch(d -> d.getCalories() >= 1000);
```
`anyMatch、allMatch`和`noneMatch`这三个操作都用到了**短路**，这就是`Java`中`&&`和`||`运算符短路在流中的版本。

##### 查找元素
`findAny`方法将返回当前流中的任意元素。下面的代码可以找到一道素食菜肴。

```java
Optional<Dish> dish = 
    menu.stream() 
        .filter(Dish::isVegetarian) 
        .findAny(); 
```

##### 查找第一个元素
`findFirst`方法返回流中的第一个元素，与`findAny`类似，只不过后者返回任意，且在使用并行流时限制较少。下面的代码能找出第一个平方
能被`3`整除的数：

```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5); 
Optional<Integer> firstSquareDivisibleByThree = 
    someNumbers.stream() 
               .map(x -> x * x) 
               .filter(x -> x % 3 == 0) 
               .findFirst(); // 9
```

#### 归约(reduce)
如果需要将流中的元素反复进行结合，得到一个值，这个过程叫做归约(`reduce`)，或者折叠(`flod`)。联想到`map-reduce`的本质是分治，归约的过程其实就是归并。

下面的代码是对数组列表中的元素求和：

```java
int[] numbers = {1, 2, 3};
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
// 使用方法引用简写上述代码
int sum = numbers.stream().reduce(0, Integer::sum);
```

`reduce`接受两个参数： 

- 一个初始值，这里是`0`
- 一个`BinaryOperator<T>`来将两个元素结合起来产生一个新值，这里我们用的是`lambda (a, b) -> a + b`

`reduce`还有一个重载的变体，它不接受初始值，但是会返回一个`Optional`对象：

```java
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));
```

因为流中没有任何元素的情况下，`reduce`操作无法返回和。

下面的例子用来返回最大值：

```java
Optional<Integer> max = numbers.stream().reduce((x, y) -> x < y ? x : y); 
Optional<Integer> max = numbers.stream().reduce(Integer::max); 
```

下面的方法用`map`和`reduce`方法数一数流中有多少个菜(`map-reduce`模式很容易并行化):

```java
int count = menu.stream() 
                // 将每个菜映射为1
                .map(d -> 1) 
                .reduce(0, (a, b) -> a + b); 
```

##### 有状态和无状态
流中有状态和无状态的定义是：

- 有状态操作：在操作过程中需要内部状态来进行中间存储的，比如`reduce、sum、max`等操作
    - 有界状态：存储需要的中间状态是有限的
    - 无解状态：存储需要的中间状态是无限的
- 无状态操作：在操作过程中不无要中间存储的操作，比如`map`或`filter`等操作会从输入流中获取每一个元素，并在输出流中得到`0`或`1`个结果

#### 数值流
在处理数据流是有可能会含有拆装箱的操作，造成性能的损失，比如下面的例子：

```java
int calories = menu.stream() 
                   .map(Dish::getCalories) 
                   .reduce(0, Integer::sum);
```
在计算时，每个`Integer`都会被拆箱成`int`型。为了避免这种损失，引入了原始类型流特化，专门支持处理数值流的方法。

##### 原始类型特化流
`IntStream 、 DoubleStream` 和`LongStream` ，分别将流中的元素特化为 `int 、 long 和 double` ，从而避免了暗含的装箱成本。每个接口都带来了进行常用数值归约的新方法，比如对数值流求和的 `sum` ，找到最大元素的 `max` 。

###### 映射到数值流 
将流转换为特化版本的常用方法是 `mapToInt 、 mapToDouble` 和 `mapToLong` 。这些方法和 `map` 方法的工作方式一样，只是它们返回的是一个特化流，而不是 `Stream<T>`。

可以像下面这样用 `mapToInt` 对 `menu` 中的卡路里求和：
```java
int calories = menu.stream() 
                   .mapToInt(Dish::getCalories)  
                   .sum(); 
```

###### 转换回对象流
可以使用`boxed()`方法将数值流转化回对象流：

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); 
// 将数值流转化为Stream
Stream<Integer> stream = intStream.boxed();
```

###### 默认值
对于三种原始流特化，也分别有一个 `Optional` 原始类型特化版本： `OptionalInt 、 OptionalDouble` 和 `OptionalLong` 。 

例如，要找到 `IntStream` 中的最大元素，可以调用 `max` 方法，它会返回一个 `OptionalInt` ：
```java
OptionalInt maxCalories = menu.stream() 
                              .mapToInt(Dish::getCalories) 
                              .max(); 
```
现在，如果没有最大值的话，就可以显式处理 `OptionalInt` 去定义一个默认值了： 
```java
int max = maxCalories.orElse(1);
```

##### 数值范围
`IntStream` 和 `LongStream` 的中有静态方法，可以生成数值范围：`range` 和 `rangeClosed` 。这两个方法都是第一个参数接受起始值，第二个参数接受结束值。但 `range` 是不包含结束值的，而 `rangeClosed` 则包含结束值。

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)  
                                 .filter(n -> n % 2 == 0);
```

#### 构件流

##### 由值创建流
静态方法 `Stream.of` ，通过显式值创建一个流。它可以接受任意数量的参数。

```java
Stream<String> stream = Stream.of("Java 8 ", "Lambdas ", "In ", "Action");
```

如下所示得到一个空流： 
```java
Stream<String> emptyStream = Stream.empty();
```

##### 由数组创建流 
静态方法 `Arrays.stream` 从数组创建一个流。它接受一个数组作为参数：
```java
int[] numbers = {2, 3, 5, 7, 11, 13}; 
int sum = Arrays.stream(numbers).sum(); 
```

##### 由文件生成流
`Files.lines`会返回一个由指定文件中的各行构成的字符串流。下面的方法查看一个文件中有多少各不相同的词：
```java
long uniqueWords = 0; 
try(Stream<String> lines = 
          Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){ 
uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))    
                   .distinct() 
                   .count(); 
}  
catch(IOException e){ 
            
} 
```

##### 由函数生成流：创建无限流
`Stream.iterate` 和 `Stream.generate`可以创建所谓的无限流：不像从固定集合创建的流那样有固定大小的流。由 `iterate` 和 `generate` 产生的流会用给定的函数按需创建值，因此可以无穷无尽地计算下去。

###### iterate
`iterate` 方法接受一个初始值，还有一个依次应用在每个产生的新值上的`Lambda`(`UnaryOperator<t>` 类型)。使用该方法产生斐波那契元组(`(0,  1), (1, 1), (1, 2), (2, 3), (3, 5), (5, 8), (8, 13), (13, 21) … `)的代码如下：
```java
Stream.iterate(new int[]{0, 1}, 
               t -> new int[]{t[1], t[0]+t[1]}) 
      .limit(20) 
      .forEach(t -> System.out.println("(" + t[0] + "," + t[1] +")")); 
```

###### generate
`generate` 不是依次对每个新生成的值应用函数的。它接受一个 `Supplier<T>` 类型的`Lambda`提供新的值。下面代码生成五个`0`到`1`之间的随机双精度数的流。

```java
Stream.generate(Math::random) 
      .limit(5) 
      .forEach(System.out::println);
```

## 用流收集数据
终端操作Collect其实是一个规约操作，它像reduceBy一样可以接受不同的做法(定义新的Collector接口)作为参数，将流中的元素累计成一个汇总结果。

一般来说， Collector 会对元素应用一个转换函数（很多时候是不体现任何效果的恒等转换，例如toList ），并将结果累积在一个数据结构中，从而产生这一过程的最终输出。Collector 接口中方法的实现决定了如何对流执行归约操作。

### 预定义收集器
预定义收集器是指可以从Collectors类中提供的工厂方法(例如groupingBy)。它们主要提供三种功能：

-  将流元素归约和汇总为一个值 
- 元素分组
- 元素分区。分区是分组的特殊情况，即使用谓词（返回一个布尔值的单参数函数）作为分组函数

### 规约和汇总

#### counting收集器
继续按照前面菜肴的例子，可以利用counting 工厂方法返回的收集器，数一数菜单里有多少种菜：

```java
long howManyDishes = menu.stream().collect(Collectors.counting()); 
```

这还可以写得更为直接： 
```java
long howManyDishes = menu.stream().count(); 
```

#### 查找流的最大值和最小值
Collectors.maxBy 和Collectors.minBy ，来计算流中的最大或最小值。这两个收集器接收一个 Comparator 参数来比较流中的元素。例如查找菜肴中热量最高的菜，可以使用如下代码:

```java
Comparator<Dish> dishCaloriesComparator = 
    Comparator.comparingInt(Dish::getCalories); 
 
Optional<Dish> mostCalorieDish = 
    menu.stream() 
        .collect(maxBy(dishCaloriesComparator));
```

#### 汇总
Collectors.summingInt 。它可接受一个把对象映射为求和所需 int 的函数，并返回一个收集器；例如，求出菜单列表中的总热量的代码如下：

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
Collectors.summingLong 和 Collectors.summingDouble 方法的作用完全一样，可以用于求和字段为 long 或 double 的情况。

Collectors.averagingInt ，连同对应的 averagingLong 和averagingDouble 可以计算数值的平均数：

```java
double avgCalories = 
    menu.stream().collect(averagingInt(Dish::getCalories));
```

summarizingInt 工厂方法可以数出菜单中元素的个数，并得到菜肴热量总和、平均值、最大值和最小值： 

```java
IntSummaryStatistics menuStatistics = 
        menu.stream().collect(summarizingInt(Dish::getCalories));
```

这个收集器会把所有这些信息收集到一个叫作 IntSummaryStatistics 的类里，它提供了方便的取值（getter）方法来访问结果。

同样，相应的 summarizingLong 和 summarizingDouble 工厂方法有相关的 LongSummary-Statistics 和DoubleSummaryStatistics 类型，适用于收集的属性是原始类型 long 或double 的情况。

#### 连接字符串
joining 工厂方法返回的收集器会把对流中每一个对象应用 toString 方法得到的所有字符串连接成一个字符串。例如把菜肴中所有菜肴的名字连起来的操作为：

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining()); 
```
该方法还可以接受一个分隔符作为字符串之间的分界，如下：

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

### 分组
用 Collectors.groupingBy 工厂方法返回的收集器可以对流中的元素进行分组，例如，对菜单中的菜按照类型(type)进行分类的代码如下：

```java
Map<Dish.Type, List<Dish>> dishesByType = 
                      menu.stream().collect(groupingBy(Dish::getType));
```
分组操作的结果是一个 Map ，把分组函数返回的值作为映射的键，把流中所有具有这个分类值的项目的列表作为对应的映射值。

但是，分类函数不一定像方法引用那样可用，因为想用以分类的条件可能比简单的属性访问器要复杂。例如，可能想把热量不到400卡路里的菜划分为“低热量”（diet），热量400到700卡路里的菜划为“普通”（normal），高于700卡路里的划为“高热量”（fat）。由于 Dish 类的作者没有把这个操作写成一个方法，无法使用方法引用，但可以把这个逻辑写成Lambda表达式:

```java
public enum CaloricLevel { DIET, NORMAL, FAT } 
 
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect( 
        groupingBy(dish -> { 
               if (dish.getCalories() <= 400) return CaloricLevel.DIET; 
               else if (dish.getCalories() <= 700) return 
    CaloricLevel.NORMAL; 
        else return CaloricLevel.FAT; 
         } )); 
```

#### 多级分组
如果想要按照多个标准进行分类，就需要使用使用多级分组。要实现多级分组，需要使用一个由双参数版本的 Collectors.groupingBy 工厂方法创建的收集器，它除了普通的分类函数之外，还可以接受 collector 类型的第二个参数。那么要进行二级分组的话，可以把一个内层roupingBy传递给外层 groupingBy ，并定义一个为流中项目分类的二级标准。例如，按照菜肴种类和卡路里进行分类的代码如下：

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
menu.stream().collect(
    // 一级分类函数
    groupingBy(Dish::getType,  
        // 二级分类函数
        groupingBy(dish -> {  
        if (dish.getCalories() <= 400) {
            return CaloricLevel.DIET; 
        } else if (dish.getCalories() <= 700) {
            return CaloricLevel.NORMAL; 
        }  else return CaloricLevel.FAT; 
        } ) 
    ) 
);
```

二级分组的结果是一个两级 Map。的外层 Map 的键就是第一级分类函数生成的值，而这个 Map 的值又是一个 Map ，键是二级分类函数生成的值，最后，第二级 map 的值是流中元素构
成的 List，是分别应用第一级和第二级分类函数所得到的对应第一级和第二级键的值。

#### 按子组收集数据
传递给第一个 groupingBy 的第二个收集器可以是任何类型，而不一定是另一个 groupingBy 。例如，要数一数菜单中每类菜有多少个，可以传递 counting 收集器作为
groupingBy 收集器的第二个参数：

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect( 
                    groupingBy(Dish::getType, counting()));
```

##### 把收集器的结果转换为另一种类型
下面的例子先按照菜肴的类型进行分类，之后选取每个种类中卡路里最高的菜肴：

```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = 
    menu.stream() 
        .collect(groupingBy(Dish::getType, 
                            maxBy(comparingInt(Dish::getCalories))));
```

这个收集器的值如下：

```java
{FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]} 
```

如果想要把结果中的Optional类型去掉，就需要将结果转化为另一种类型，可以使用Collectors.collectingAndThen 工厂方法返回的收集器。

```java
Map<Dish.Type, Dish> mostCaloricByType = 
    menu.stream() 
        .collect(groupingBy(Dish::getType,    
                 collectingAndThen( 
                     maxBy(comparingInt(Dish::getCalories)),  
                 Optional::get)));  
```

这个工厂方法接受两个参数——要转换的收集器以及转换函数，并返回另一个收集器。

Optional.get()，这个操作放在这里是安全的，因为 reducing收集器永远都不会返回 Optional.empty() 。其结果是下面的 Map：

```
{FISH=salmon, OTHER=pizza, MEAT=pork}。
```

##### 与groupingBy联合使用的其他收集器的例子 
一般来说，通过 groupingBy 工厂方法的第二个参数传递的收集器将会对分到同一组中的所有流元素执行进一步归约操作。除此之外，常常和 groupingBy 联合使用的另一个收集器是 mapping 方法生成的。这个方法接受两个参数：一个函数对流中的元素做变换，另一个则将变换的结果对象收集起来。其目的是在累加之前对每个输入元素应用一个映射函数，这样就可以让接受特定类型元素的收集器适应不同类型的对象。

比如，对于每种类型的 Dish，菜单中都有哪些 CaloricLevel的实现方式如下：

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = 
menu.stream().collect( 
   groupingBy(Dish::getType, mapping( 
   dish -> {
       if (dish.getCalories() <= 400) {
           return CaloricLevel.DIET; 
       } else if (dish.getCalories() <= 700) {
           return CaloricLevel.NORMAL; 
       } else {
           return CaloricLevel.FAT; 
           }, 
   toSet() )));
```
结果为：

```
{OTHER=[DIET, NORMAL], MEAT=[DIET, NORMAL, FAT], FISH=[DIET, NORMAL]} 
```

但是，在这个例子中，对于返回的 Set 是什么类型并没有任何保证。但通过使用 toCollection ，就可以有更多的控制。例如，可以给它传递一个构造函数引用来要求 HashSet:

```java
Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = 
menu.stream().collect( 
    groupingBy(Dish::getType, mapping( 
    dish -> {
        if (dish.getCaloeies() <= 400) {
            return CaloricLevel.DIET;
        } else if (dish.getCalories() <= 700) {
            return CaloricLevel.NORMAL; 
        } else {
            return CaloricLevel.FAT; 
            }, 
    toCollection(HashSet::new) ))); 
```

#### 分区
分区是分组的特殊情况，它由一个谓词（返回一个布尔值的函数）作为分类函数，这儿函数叫做**分区函数**。分区得到的分组Map的键类型是Boolean，于是它最多可以分为两组——true是一组，false是一组。下面的代码可以将菜单中的素食和非素食分开：

```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));
```

##### 多级分区
同样的，partitioningBy方法第二个参数也可以接收一个收集器，从而实现多级分区。下面的方法会产生一个二级Map：

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType =
menu.stream().collect(partitioningBy(Dish::isVegetarian,groupingBy(Dish::getType)));
```

结果为：
```
{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},true={OTHER=[french fries, rice, season fruit, pizza]}}
```

##### 一个例子：将数字按照质数和非质数进行分类
下面的代码接受一个参数n，并将前n个自然数分为质数和非质数，如下：

```java
    public boolean isPrime(int candidate) {
        // 仅需要测试小于等于待测数平方根的因子
        int candidateRoot = (int)Math.sqrt(candidate);
        // 产生一个自然数范围，从2开始；如果待测数不能被流中任意数字正常则返回true
        return IntStream.rangeClosed(2, candidateRoot).noneMatch(i -> i % 2 == 0);
    }

    public Map<Boolean, List<Integer>> partitionPrimes(int n) {
        // boxed()方法的作用是将数值流转换为流
        return IntStream.rangeClosed(2, n)
                .boxed()
                .collect(Collectors.partitioningBy(candidate -> isPrime(candidate)));
    }
```

如果传入10，则返回结果为：
```
{false=[4, 5, 6, 7, 8, 9, 10], true=[2, 3]}
```

##### 自定义收集器
Collector接口的声明如下：
```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```

- T是流中要收集的项目的泛型。
- A是累加器的类型，累加器是在收集过程中用于累积部分结果的对象。
- R是收集操作得到的对象（通常但并不一定是集合）的类型。

也可以说，T是供应源 A是累加器 R是组合器。

其中，前四个方法都会返回一个被collect方法调用的函数，而第五个方法返回一个提示列表，告诉collect方法在执行规约操作的时候可以应用那些优化，比如并行度等。

##### 建立新的结果容器：supplier方法
在调用该方法时会创建一个空的累加器实例，供数据收集过程使用。比如下面的写法可以返回一个空的List：

```java
public Supplier<List<T>> supplier() {
    return ArrayList::new;
}
```

##### 将元素添加到结果容器：accumulator方法
accumulator方法会返回执行归约操作的函数。当遍历到流中第n个元素时，这个函数执行时会有两个参数：保存归约结果的累加器（已收集了流中的前 n1 个项目），还有第n个元素本身。该函数将返回void，因为累加器是原位更新。

将元素添加至当前列表的代码如下：

```java
public BiConsumer<List<T>, T> accumulator() {
    return List::add;
}
```

##### 对结果容器应用最终转换：finisher方法
在遍历完流后，finisher方法必须返回在累积过程的最后要调用的一个函数，以便将累加器对象转换为整个集合操作的最终结果。例如：

```java
public Function<List<T>, List<T>> finisher() {
    return Function.identity();
}
```

##### 合并两个结果容器：combiner方法
combiner方法会返回一个供归约操作使用的函数，它定义了对流的各个子部分进行并行处理时，各个子部分归约所得的累加器要如何合并。例如：

```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1; 
    }
}
```

##### characteristics方法
characteristics会返回一个不可变的Characteristics集合，它定义了收集器的行为——尤其是关于流是否可以并行归约，以及可以使用哪些优化的提示。Characteristics是一个包含三个项目的枚举。

- UNORDERED——归约结果不受流中项目的遍历和累积顺序的影响。
- CONCURRENT——accumulator函数可以从多个线程同时调用，且该收集器可以并行归约流。如果收集器没有标为UNORDERED，那它仅在用于无序数据源时才可以并行归约。
- IDENTITY_FINISH——这表明完成器方法返回的函数是一个恒等函数，可以跳过。这种情况下，累加器对象将会直接用作归约过程的最终结果。这也意味着，将累加器A不加检查地转换为结果R是安全的。

## 并行数据处理与性能
并行流就是一个把内容分成多个数据块，并用不同的线程分别处理每个数据块的流。

### 串行流和并行流的转换
可以调用parallel方法将串行流转变成并行流：

```java
public static long parallelSum(long n) {
return Stream.iterate(1L, i -> i + 1)
    .limit(n)
    .parallel()
    .reduce(0L, Long::sum);
}
```
对并行流调用sequential方法就可以把它变成顺序流，sequential方法和parallel方法可以结合使用，可以更细化地控制在遍历流时哪些操作要并行执行，哪些要顺序执行。但最后一次parallel或sequential调用会影响整个流水线。在下面的例中，流水线会并行执行，因为最后调用的是它。

```java
stream.parallel()
    .filter(...)
    .sequential()
    .map(...)
    .parallel()
    .reduce();
```

并行流内部使用了默认的ForkJoinPool，它默认的线程数量是机器的处理器数量， 这个值是由Runtime.getRuntime().available-Processors()得到的。可以通过系统属性java.util.concurrent.ForkJoinPool.common.parallelism来改变线程池大小，如下所示：

```
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","12");
```
这是一个全局设置，因此它将影响代码中所有的并行流，如果没有充足的理由，最好不要修改。

### 正确使用并行流
错用并行流而产生错误的首要原因，就是使用的算法改变了某些共享状态。

```java
public class Accumulator { 
    public long total = 0; 
    public void add(long value) { total += value; } 
}

public static long sideEffectParallelSum(long n) { 
    Accumulator accumulator = new Accumulator(); 
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add); 
    return accumulator.total; 

}
```

这是因为total是一个共享变量，多个线程在同时访问累加器，执行total += value，而这个操作不是一个原子操作，会产生并发问题。
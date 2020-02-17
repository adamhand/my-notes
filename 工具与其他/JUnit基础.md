# `JUnit`基础

`JUnit`是一个`Java`测试框架，继承自`xUnit`。

## 常用的注解
`JUnit`中常用的注解有：

- `@Test`：将一个普通方法修饰成一个测试方法。该方法可以设置参数，常见的参数有：
    - `@Test(expcted=xxx.class)`：假如测试方法发生`xxx`这个异常类的异常，方法会继续执行而不会被异常打断
    - `@Test(timeout=xxms)`：在`xx`毫秒之后会打断该测试方法的执行
- `@BeforClass`：修饰`static`方法，在类加载时被执行
- `@AfterClass`：同上，在所有方法运行结束后执行
- `@Befor`：在每一个测试方法执行前都会执行一次
- `@After`：在每一个测试方法执行后都会执行一次
- `@Ignore`：修饰的方法会被忽略运行
- `@RunWith`：用来更改测试运行器

`@Test(timeout=xxms)`的演示如下。一个“计算器”类有一个“除法”方法，如下：

```java
public class CalculateService {
    public int div(int a, int b) {
        return a / b;
    }
}
```

测试该方法的代码如下：

```java
@Test(expected = ArithmeticException.class)
public void testDiv() {
    assertEquals(1, new CalculateService().div(1, 0));
}
```
调用`div`方法时穿进去的除数为`0`，是错误的，如果不加`expected = ArithmeticException.class`，运行会出错，加上之后程序不会报错。

## 测试套件的使用
假如测试方法分散在多个测试类中，想一键执行它们，可以使用测试套件`Suite`。比如，有三个测试类`TestClass0 、TestClass1和TestClass2`，如下：

```java
public class TestClass0 {
    @Test
    public void test() {
        System.out.println("test class 0");
    }
}

public class TestClass1 {
    @Test
    public void test() {
        System.out.println("test class 1");
    }
}

public class TestClass2 {
    @Test
    public void test() {
        System.out.println("test class 2");
    }
}
```
在`TestClassAll`类中一键执行三个测试类中的测试方法，需要在`TestClassAll`类中添加注解，如下：

```java
@RunWith(Suite.class)
@Suite.SuiteClasses({ TestClass0.class, TestClass1.class, TestClass2.class })
public class TestClassAll {
}
```
这样，运行`TestClassAll`类就可以一键执行其余三个测试类中的测试方法。

## 参数化测试
在运行一个测试方法的时候，有时候可能需要使用多组参数对其进行测试。如果每次更换参数都需要手动运行代码，就增加了复杂度。`Parameterized`的测试运行期允许使用者使用不同参数多次运行同一个测试。

使用`Parameterized`需要注意以下几个规则：

- `RunWith`中使用(`value = Parameterized.class`)
- 需有一个**公共静态**方法，该方法的返回值必须是**Collection(T[])**，且不能有参数，该方法需要被`@Parameters`修饰
- 若需要的话，声明变量来存放预期值和输入参数
- 为测试类声明一个带参数的构造函数对变量进行注入(或者使用`@Parameter`对变量进行注解式的注入)

例子如下：

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import static org.junit.Assert.*;

import java.util.Arrays;
import java.util.Collection;

@RunWith(Parameterized.class)
public class ParameterTest {
    int expected = 0;
    int input0 = 0;
    int input1 = 0;

    @Parameterized.Parameters
    public static Collection<Object[]> method() {
        return Arrays.asList(new Object[][] {
            {1, 2, 1},
            {2, 4, 2}
        });
    }

    public ParameterTest(int expected, int input0, int input1) {
        this.expected = expected;
        this.input0 = input0;
        this.input1 = input1;
    }

    @Test
    public void testDiv() {
        assertEquals(expected, new CalculateService().div(input0, input1));
    }
}
```

也可以将构造函数替换为使用`@Parameter`注解进行注入：

```java
@Parameterized.Parameter
public int expected = 0;
@Parameterized.Parameter(1)
public int input0 = 0;
@Parameterized.Parameter(2)
public int input1 = 0;
```

如果的测试只需要单个参数，则不需要将其包装成数组。这种情况下可以提供一个迭代器或对象数组(从`4.12-beta-3`版本开始)。如下：

```java
@Parameters
public static Iterable<? extends Object> data() {
    return Arrays.asList("first test", "second test");
}
```
或者

```java
@Parameters
public static Object[] data() {
    return new Object[] { "first test", "second test" };
}
```

验证代码如下：
```java
public class HelloString {
    public String hello(String name) {
        return "hello" + name;
    }
}

@RunWith(Parameterized.class)
public class HelloTest {
    @Parameterized.Parameter
    public String para;

    // 迭代器
    @Parameterized.Parameters
    public static Iterable<? extends Object> method() {
        return Arrays.asList(" world", " junit");
    }

    // // 对象数组
    // @Parameterized.Parameters
    // public static Object[] method() {
    //     return new Object[] {" world", " junit"};
    // }

    @Test
    public void test() {
        System.out.println(new HelloString().hello(para));
    }
}
```

## 参考
[JUnit—Java单元测试必备工具](https://www.imooc.com/learn/356)</br>
[Parameterized tests](https://github.com/junit-team/junit4/wiki/Parameterized-tests)
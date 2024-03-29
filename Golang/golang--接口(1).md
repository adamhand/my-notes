﻿# golang--接口(1)
---
在 Go 语言中，接口就是方法签名（Method Signature）的集合。**当一个类型实现了接口中的所有方法，我们称它实现了该接口**。这与面向对象编程（OOP）的说法很类似**。接口指定了一个类型应该具有的方法，并由该类型决定如何实现这些方法**。

# 接口的声明与实现
让我们编写代码，创建一个接口并且实现它。
```
package main

import (  
    "fmt"
)

//interface definition
type VowelsFinder interface {  
    FindVowels() []rune
}

type MyString string

//MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {  
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {  
    name := MyString("Sam Anderson")
    var v VowelsFinder
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())

}
```
在第 15 行，我们给接受者类型（Receiver Type） MyString 添加了方法 FindVowels() []rune。现在，我们称 MyString 实现了 VowelsFinder 接口。这就和其他语言（如 Java）很不同，其他一些语言要求一个类使用 implement 关键字，来显式地声明该类实现了接口。而在 Go 中，并不需要这样。如果一个类型包含了接口中声明的所有方法，那么它就隐式地实现了 Go 接口。

在第 28 行，v 的类型为 VowelsFinder，name 的类型为 MyString，我们把 name 赋值给了 v。由于 MyString 实现了 VowelFinder，因此这是合法的。

该程序输出 Vowels are [a e o]。

# 接口的实际用途
前面的例子教我们创建并实现了接口，但还没有告诉我们接口的实际用途。在上面的程序里，如果我们使用 name.FindVowels()，而不是 v.FindVowels()，程序依然能够照常运行，但接口并没有体现出实际价值。

因此，我们现在讨论一下接口的实际应用场景。

我们编写一个简单程序，根据公司员工的个人薪资，计算公司的总支出。为了简单起见，我们假定支出的单位都是美元。
```
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int
}

type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

type Contract struct {  
    empId  int
    basicpay int
}

//salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)

}
```

上面程序的第 7 行声明了一个 SalaryCalculator 接口类型，它只有一个方法 CalculateSalary() int。

在公司里，我们有两类员工，即第 11 行和第 17 行定义的结构体：Permanent 和 Contract。长期员工（Permanent）的薪资是 basicpay 与 pf 相加之和，而合同员工（Contract）只有基本工资 basicpay。在第 23 行和第 28 行中，方法 CalculateSalary 分别实现了以上关系。由于 Permanent 和 Contract 都声明了该方法，因此它们都实现了 SalaryCalculator 接口。

第 36 行声明的 totalExpense 方法体现出了接口的妙用。该方法接收一个 SalaryCalculator 接口的切片（[]SalaryCalculator）作为参数。在第 49 行，我们向 totalExpense 方法传递了一个包含 Permanent 和 Contact 类型的切片。在第 39 行中，通过调用不同类型对应的 CalculateSalary 方法，totalExpense 可以计算得到支出。

这样做最大的优点是：totalExpense 可以扩展新的员工类型，而不需要修改任何代码。假如公司增加了一种新的员工类型 Freelancer，它有着不同的薪资结构。Freelancer只需传递到 totalExpense 的切片参数中，无需 totalExpense 方法本身进行修改。只要 Freelancer 也实现了 SalaryCalculator 接口，totalExpense 就能够实现其功能。

该程序输出 `Total Expense Per Month $14050`。

# 接口的内部表示
我们可以把接口看作内部的一个元组 (type, value)。 type 是接口底层的具体类型（Concrete Type），而 value 是具体类型的值。
```
package main

import "fmt"

type Test interface {
	Tester()
}

type MyFloat float64

func (m MyFloat)Tester()  {
	fmt.Println(m)
}

func describe(t Test){
	//t既包含一个类型也包含一个值
	fmt.Printf("Interface type %T value %v\n", t, t)
}

func main(){
	var t Test
	f := MyFloat(43.90)
	t = f
	describe(t)
	t.Tester()
}

```
在第17行，t 的具体类型为 MyFloat，而 t 的值为 89.7。该程序输出：
```
Interface type main.MyFloat value 89.7  
89.7
```

# 空接口
没有包含方法的接口称为空接口。空接口表示为 interface{}。由于空接口没有方法，**因此所有类型都实现了空接口,可以给空接口传递任何类型**。

```
package main

import (  
    "fmt"
)

func describe(i interface{}) {  
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {  
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
```
该程序打印：
```
Type = string, value = Hello World  
Type = int, value = 55  
Type = struct { name string }, value = {Naveen R}
```

# 类型断言
类型断言用于提取接口的底层值（Underlying Value）。

在语法 `i.(T)` 中，接口 i 的具体类型是 T，该语法用于获得接口的底层值。

```
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) //get the underlying int value from i
    fmt.Println(s)
}
func main() {  
    var s interface{} = 56
    assert(s)
}
```
在第 8 行，我们使用了语法 i.(int) 来提取 i 的底层 int 值。该程序会打印 56。

在上面程序中，如果具体类型不是 int，会发生什么呢？
```
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) 
    fmt.Println(s)
}
func main() {  
    var s interface{} = "Steven Paul"
    assert(s)
}
```
该程序会报错：panic: interface conversion: interface {} is string, not int.。

要解决该问题，我们可以使用以下语法：
```
v, ok := i.(T)
```
如果 i 的具体类型是 T，那么 v 赋值为 i 的底层值，而 ok 赋值为 true。

如果 i 的具体类型不是 T，那么 ok 赋值为 false，v 赋值为 T 类型的零值，**此时程序不会报错**。

```
package main

import (  
    "fmt"
)

func assert(i interface{}) {  
    v, ok := i.(int)
    fmt.Println(v, ok)
}
func main() {  
    var s interface{} = 56
    assert(s)
    var i interface{} = "Steven Paul"
    assert(i)
}
```
该程序打印：
```
56 true  
0 false
```

# 类型选择（Type Switch）
类型选择用于将接口的具体类型与很多 case 语句所指定的类型进行比较。它与一般的 switch 语句类似。唯一的区别在于类型选择指定的是类型，而一般的 switch 指定的是值。

类型选择的语法类似于类型断言。类型断言的语法是 `i.(T)`，而对于类型选择，类型 T 由关键字 `type` 代替。
```
package main

import (  
    "fmt"
)

func findType(i interface{}) {  
    switch i.(type) {
    case string:
        fmt.Printf("I am a string and my value is %s\n", i.(string))
    case int:
        fmt.Printf("I am an int and my value is %d\n", i.(int))
    default:
        fmt.Printf("Unknown type\n")
    }
}
func main() {  
    findType("Naveen")
    findType(77)
    findType(89.98)
}
```
该程序输出：
```
I am a string and my value is Naveen  
I am an int and my value is 77  
Unknown type
```

**还可以将一个类型和接口相比较。如果一个类型实现了接口，那么该类型与其实现的接口就可以互相比较。**

```
package main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() {  
    fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {  
    switch v := i.(type) {
    case Describer:
        v.Describe()
    default:
        fmt.Printf("unknown type\n")
    }
}

func main() {  
    findType("Naveen")
    p := Person{
        name: "Naveen R",
        age:  25,
    }
    findType(p)
}
```
在上面程序中，结构体 Person 实现了 Describer 接口。在第 19 行的 case 语句中，v 与接口类型 Describer 进行了比较。p 实现了 Describer，因此满足了该 case 语句，于是当程序运行到第 32 行的 findType(p) 时，程序调用了 Describe() 方法。

该程序输出：
```
unknown type  
Naveen R is 25 years old
```
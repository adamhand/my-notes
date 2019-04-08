# golang--接口(2)
---
# 实现接口：指针接受者与值接受者
可以使用值接受者（Value Receiver）来实现接口，同样可以使用指针接受者（Pointer Receiver）来实现接口。
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

func (p Person) Describe() { // 使用值接受者实现  
    fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {
    state   string
    country string
}

func (a *Address) Describe() { // 使用指针接受者实现
    fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {  
    var d1 Describer
    p1 := Person{"Sam", 25}
    d1 = p1
    d1.Describe()
    p2 := Person{"James", 32}
    d1 = &p2
    d1.Describe()

    var d2 Describer
    a := Address{"Washington", "USA"}

    /* 如果下面一行取消注释会导致编译错误：
       cannot use a (type Address) as type Describer
       in assignment: Address does not implement
       Describer (Describe method has pointer
       receiver)
    */
    //d2 = a

    d2 = &a // 这是合法的
    // 因为在第 22 行，Address 类型的指针实现了 Describer 接口
    d2.Describe()

}
```
**使用值接受者声明的方法，既可以用值来调用，也能用指针调用。不管是一个值，还是一个可以解引用的指针，调用这样的方法都是合法的。**所以d1既可以使用值接收者调用p1的Describe()方法，也可以使用指针接受者调用p2的Describe()方法。

在上面程序里，如果去掉第 45 行的注释，我们会得到编译错误：main.go:42: cannot use a (type Address) as type Describer in assignment: Address does not implement Describer (Describe method has pointer receiver)。这是因为在第 22 行，我们使用 Address 类型的指针接受者实现了接口 Describer，而接下来我们试图用 a 来赋值 d2。然而 a 属于值类型，它并没有实现 Describer 接口。你应该会很惊讶，**因为我们曾经学习过，使用指针接受者的方法，无论指针还是值都可以调用它。**那么为什么第 45 行的代码就不管用呢？

**其原因是：对于使用指针接受者的方法，用一个指针或者一个可取得地址的值来调用都是合法的。但接口中存储的具体值（Concrete Value）并不能取到地址，因此在第 45 行，对于编译器无法自动获取 a 的地址，于是程序报错。**

程序正常运行的结果如下：
```
Sam is 25 years old  
James is 32 years old  
State Washington Country USA
```

# 实现多个接口
类型可以实现多个接口。
```
package main

import "fmt"

type SalaryCalculator interface {
	displaySalary()
}

type LeaveCalculator interface {
	calculatorLeavesLeft() int
}

type Employee struct {
	firstName string
	lastName string
	basicPay int
	pf int
	totalLeaves int
	leavesToken int
}

func (e Employee)displaySalary()  {
	fmt.Printf("%s %s has salary $%d\n", e.firstName, e.lastName, (e.pf + e.basicPay))
}

func (e Employee)calculatorLeavesLeft() int {
	return e.totalLeaves - e.leavesToken
}

func main()  {
	e := Employee{
		firstName:"bob",
		lastName:"steven",
		basicPay:3000,
		pf:2000,
		totalLeaves:10,
		leavesToken:3,
	}
	var s SalaryCalculator = e
	s.displaySalary()
	var l LeaveCalculator = e
	fmt.Printf("\nleaves left = %d", l.calculatorLeavesLeft())
}
```
该程序会输出：
```
bob steven has salary $5000

leaves left = 7
```

# 接口的嵌套
尽管 Go 语言没有提供继承机制，但可以通过嵌套其他的接口，创建一个新接口。

```
package main

import "fmt"

type SalaryCalculator interface {
	displySalary()
}

type LeavesCalculator interface {
	calculatorLeavesLeft() int
}

type EmployeeOprations interface {
	SalaryCalculator
	LeavesCalculator
}

type Employee struct {
	firstName string
	lastName string
	basicPay int
	pf int
	totalLeaves int
	leavesToken int
}

func (e Employee)displySalary()  {
	fmt.Printf("%s %s has %d salary\n", e.firstName, e.lastName, (e.pf + e.basicPay))
}

func (e Employee)calculatorLeavesLeft() int {
	return e.totalLeaves - e.leavesToken
}

func main()  {
	e := Employee{
		firstName:"bob",
		lastName:"steven",
		basicPay:4000,
		pf:1000,
		totalLeaves:10,
		leavesToken:5,
	}
	var empOp EmployeeOprations = e
	empOp.displySalary()
	fmt.Printf("\nleaves left = %d", empOp.calculatorLeavesLeft())
}
```

在 46 行，empOp 的类型是 EmployeeOperations，e 的类型是 Employee，我们把 empOp 赋值为 e。接下来的两行，empOp 调用了 DisplaySalary() 和 CalculateLeavesLeft() 方法。

该程序输出：
```
bob steven has 5000 salary

leaves left = 5
```

# 接口的零值
接口的零值是 nil。对于值为 nil 的接口，其底层值（Underlying Value）和具体类型（Concrete Type）都为 nil。
```
package main

import "fmt"

type Describer interface {
	describe()
}

func main()  {
	var d1 Describer
	if d1 == nil{
		fmt.Printf("d1 is nil and has type %T value %v", d1, d1)
	}
}
```
改程序输出：
```
d1 is nil and has type <nil> value <nil>
```

对于值为 nil 的接口，由于没有底层值和具体类型，当我们试图调用它的方法时，程序会产生 panic 异常。
```
package main

type Describer interface {
    Describe()
}

func main() {  
    var d1 Describer
    d1.Describe()
}
```

在上述程序中，d1 等于 nil，程序产生运行时错误 `panic： panic: runtime error: invalid memory address or nil pointer dereference [signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xc8527]` 。
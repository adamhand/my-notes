# golang--结构体
---
结构体是用户定义的类型，表示若干个字段（Field）的集合。有时应该把数据整合在一起，而不是让这些数据没有联系。这种情况下可以使用结构体。

# 结构体的声明
```
type Employee struct {
    firstName string
    lastName  string
    age       int
}
```
在上面的代码片段里，声明了一个结构体类型 Employee，它有 firstName、lastName 和 age 三个字段。通过把相同类型的字段声明在同一行，结构体可以变得更加紧凑。在上面的结构体中，firstName 和 lastName 属于相同的 string 类型，于是这个结构体可以重写为：
```
type Employee struct {
    firstName, lastName string
    age, salary         int
}
```
上面的结构体 Employee 称为 命名的结构体（Named Structure）。我们创建了名为 Employee 的新类型，而它可以用于创建 Employee 类型的结构体变量。

声明结构体时也可以不用声明一个新类型，这样的结构体类型称为 匿名结构体（Anonymous Structure）。
```
var employee struct {
    firstName, lastName string
    age int
}
```
上述代码片段创建一个匿名结构体 employee

# 创建命名的结构体
通过下面代码，我们定义了一个命名的结构体 Employee。
```
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {

    //creating structure using field names
    emp1 := Employee{
        firstName: "Sam",
        age:       25,
        salary:    500,
        lastName:  "Anderson",
    }

    //creating structure without using field names
    emp2 := Employee{"Thomas", "Paul", 29, 800}

    fmt.Println("Employee 1", emp1)
    fmt.Println("Employee 2", emp2)
}
```
而在第 15 行，通过指定每个字段名的值，我们定义了结构体变量 emp1。字段名的顺序不一定要与声明结构体类型时的顺序相同。

在上面程序的第 23 行，定义 emp2 时我们省略了字段名。在这种情况下，就需要保证字段名的顺序与声明结构体时的顺序相同。

该程序将输出：
```
Employee 1 {Sam Anderson 25 500}
Employee 2 {Thomas Paul 29 800}
```

# 创建匿名结构体
```
package main

import (
    "fmt"
)

func main() {
    emp3 := struct {
        firstName, lastName string
        age, salary         int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```
之所以称这种结构体是匿名的，是因为它只是创建一个新的结构体变量 em3，而没有定义任何结构体类型。

该程序会输出：
```
Employee 3 {Andreah Nikola 31 5000}
```

# 结构体的零值（Zero Value）
当定义好的结构体并没有被显式地初始化时，该结构体的字段将默认赋为零值。
```
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp4 Employee //zero valued structure
    fmt.Println("Employee 4", emp4)
}
```

该程序定义了 emp4，却没有初始化任何值。因此 firstName 和 lastName 赋值为 string 的零值（""）。而 age 和 salary 赋值为 int 的零值（0）。该程序会输出：
```
Employee 4 { 0 0}
```
# 访问结构体的字段
点号操作符 `.` 用于访问结构体的字段。
```
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp6 := Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp6.firstName)
    fmt.Println("Last Name:", emp6.lastName)
    fmt.Println("Age:", emp6.age)
    fmt.Printf("Salary: $%d", emp6.salary)
}
```

上面程序中的 emp6.firstName 访问了结构体 emp6 的字段 firstName。该程序输出：
```
First Name: Sam  
Last Name: Anderson  
Age: 55  
Salary: $6000
```

# 结构体的指针
还可以创建指向结构体的指针。
```
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", (*emp8).firstName)
    fmt.Println("Age:", (*emp8).age)
}
```

在上面程序中，emp8 是一个指向结构体 Employee 的指针。(*emp8).firstName 表示访问结构体 emp8 的 firstName 字段。该程序会输出：
```
First Name: Sam
Age: 55
```

**Go 语言允许我们在访问 firstName 字段时，可以使用 emp8.firstName 来代替显式的解引用 (*emp8).firstName。**
```
package main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp8.firstName)
    fmt.Println("Age:", emp8.age)
}
```

# 匿名字段
当我们创建结构体时，字段可以只有类型，而没有字段名。这样的字段称为匿名字段（Anonymous Field）。
```
以下代码创建一个 Person 结构体，它含有两个匿名字段 string 和 int。

type Person struct {  
    string
    int
}
```
**虽然匿名字段没有名称，但其实匿名字段的名称就默认为它的类型**。如下所示：
```
package main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    var p1 Person
    p1.string = "naveen"
    p1.int = 50
    fmt.Println(p1)
}
```
程序的输出如下：
```
{naveen 50}
```

# 嵌套结构体（Nested Structs）
结构体的字段有可能也是一个结构体。这样的结构体称为嵌套结构体。
```
package main

import (  
    "fmt"
)

type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age int
    address Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.address = Address {
        city: "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:",p.age)
    fmt.Println("City:",p.address.city)
    fmt.Println("State:",p.address.state)
}
```

该程序输出：
```
Name: Naveen  
Age: 50  
City: Chicago  
State: Illinois
```

# 提升字段（Promoted Fields）
如果是结构体中有匿名的结构体类型字段，则该匿名结构体里的字段就称为**提升字段**。这是因为提升字段就像是属于外部结构体一样，可以用外部结构体直接访问。
```
package main

import (
    "fmt"
)

type Address struct {
    city, state string
}
type Person struct {
    name string
    age  int
    Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city) //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```
在上面的代码片段中，Person 结构体有一个匿名字段 Address，而 Address 是一个结构体。现在结构体 Address 有 city 和 state 两个字段，访问这两个字段就像在 Person 里直接声明的一样，因此我们称之为提升字段。

# 导出结构体和字段
如果结构体名称以大写字母开头，则它是其他包可以访问的导出类型（Exported Type）。同样，如果结构体里的字段首字母大写，它也能被其他包访问到。

```
package computer

type Spec struct { //exported struct  
    Maker string //exported field
    model string //unexported field
    Price int //exported field
}
```
如上所示，在computer中定义一个结构体，它是导出的，该结构体中的Marker字段和Price字段也是导出的，而model字段不是导出的。

下面从main包中访问：
```
package main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    fmt.Println("Spec:", spec)
}
```
上述程序是正确的，但是如果试图访问未导出的字段 model，则会报错。

# 结构体相等性（Structs Equality）
结构体是值类型。如果它的每一个字段都是可比较的，则该结构体也是可比较的。如果两个结构体变量的对应字段相等，则这两个变量也是相等的。
```
package main

import (  
    "fmt"
)

type name struct {  
    firstName string
    lastName string
}


func main() {  
    name1 := name{"Steve", "Jobs"}
    name2 := name{"Steve", "Jobs"}
    if name1 == name2 {
        fmt.Println("name1 and name2 are equal")
    } else {
        fmt.Println("name1 and name2 are not equal")
    }

    name3 := name{firstName:"Steve", lastName:"Jobs"}
    name4 := name{}
    name4.firstName = "Steve"
    if name3 == name4 {
        fmt.Println("name3 and name4 are equal")
    } else {
        fmt.Println("name3 and name4 are not equal")
    }
}
```
该程序会输出：
```
name1 and name2 are equal  
name3 and name4 are not equal
```
**如果结构体包含不可比较的字段，则结构体变量也不可比较。**
```
package main

import (  
    "fmt"
)

type image struct {  
    data map[int]int
}

func main() {  
    image1 := image{data: map[int]int{
        0: 155,
    }}
    image2 := image{data: map[int]int{
        0: 155,
    }}
    if image1 == image2 {
        fmt.Println("image1 and image2 are equal")
    }
}
```
上面的例子中，map是不可比较的，所以image也不可比较。
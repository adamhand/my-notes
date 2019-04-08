# golang--结构体取代类
---

# Go 支持面向对象吗？
Go 并不是完全面向对象的编程语言。Go语言官网对此的解答为：

> 可以说是，也可以说不是。虽然 Go 有类型和方法，支持面向对象的编程风格，但却没有类型的层次结构。Go中没有类的概念，但是可以使用结构体来代替，Go 也可以将结构体嵌套使用，这与子类化（Subclassing）类似，但并不完全相同。此外，Go 提供的特性比 C++ 或 Java 更为通用：子类可以由任何类型的数据来定义，甚至是内建类型（如简单的“未装箱的”整型）。这在结构体（类）中没有受到限制。

# 使用结构体，而非类
Go 不支持类，而是提供了结构体。结构体中可以添加方法。这样可以将数据和操作数据的方法绑定在一起，实现与类相似的效果。下面通过一个例子来理解。

首先，建立employee.go文件夹，它的目录结构如下：
> workspacepath -> oop -> employee -> employee.go

内容如下：
```
package employee

import (  
    "fmt"
)

type Employee struct {  
    FirstName   string
    LastName    string
    TotalLeaves int
    LeavesTaken int
}

func (e Employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.FirstName, e.LastName, (e.TotalLeaves - e.LeavesTaken))
}
```
接着在 oop 文件夹里创建一个文件，命名为 main.go。

现在目录结构如下所示：

> workspacepath -> oop -> employee -> employee.go  
workspacepath -> oop -> main.go
```
main.go 的内容如下所示：

package main

import "oop/employee"

func main() {  
    e := employee.Employee {
        FirstName: "Sam",
        LastName: "Adolf",
        TotalLeaves: 30,
        LeavesTaken: 20,
    }
    e.LeavesRemaining()
}
```
执行main.go程序，会打印输出：
```
Sam Adolf has 10 leaves remaining
```

---
**注意：关于`import`引入自定义包的问题**
import的包都是相对`$GOPATH/src`目录引入的，如果项目的`import`路径是这样写的：
`import "github.com/yourname/projectname"`
需要将项目代码放置在：
`$GOAPTH/src/github.com/yourname/projectname/`下

如果项目的`import`是这样写的：
`import "message"`
则将`message.go`放到:
`$GOAPTH/src/message/`目录下即可。

---

# 使用 New() 函数，而非构造器
Java等语言使用构造函数来对对象进行初始化，而golang使用New()函数，该函数是由程序员提供的，就类似于Java中的set方法。

Java中的Set方法一般是对private变量进行初始化，相同的，使用New()函数进行初始化的结构体一般不可对外引用，即首字母小写。
```
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```
main函数如下：
```
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

程序的执行结果为:
```
Sam Adolf has 10 leaves remaining
```

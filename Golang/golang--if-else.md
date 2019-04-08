# golang--if-else
---
# 基本语法
if 语句的基本语法是
```
if condition {  
}
```
Go 语言里的 { } 是必要的，即使在 { } 之间只有一条语句。

if 语句还有可选的 else if 和 else 部分。主要注意的是**：else 语句应该在 if 语句的大括号 } 之后的同一行中。如果不是，编译器会不通过。**
```
if condition {  
} else if condition {
} else {
}
```


让我们编写一个简单的程序来检测一个数字是奇数还是偶数。
```
package main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    }  else {
        fmt.Println("the number is odd")
    }
}
```
运行结果为：
```
the number is even
```

if 还有另外一种形式，它包含一个 statement 可选语句部分，该组件在条件判断之前运行。它的语法是
```
if statement; condition {  
}
```
让我们重写程序，使用上面的语法来查找数字是偶数还是奇数。
```
package main

import (  
    "fmt"
)

func main() {  
    if num := 10; num % 2 == 0 { //checks if number is even
        fmt.Println(num,"is even") 
    }  else {
        fmt.Println(num,"is odd")
    }
}
```
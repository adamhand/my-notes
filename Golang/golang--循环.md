# golang--循环
---

# 基本语法
for 是 Go 语言唯一的循环语句。Go 语言中并没有其他语言比如 C 语言中的 while 和 do while 循环。

for语句有三种基本形式：第一种是常用情况；第二种是简化情况；第三种是无限循环
```
for initialisation; condition; post {  
}
```
```
for condition{  
}
```
```
for  {  
}
```


# 例子
让我们用 for 循环写一个打印出从 1 到 10 的程序。
```
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        fmt.Printf(" %d",i)
    }
}
```
上述程序可写为：
```
package main

import (  
    "fmt"
)

func main() {  
    i := 0
    for i <= 10 { //semicolons are ommitted and only condition is present
        fmt.Printf("%d ", i)
        i += 1
    }
}
```

# break
break 语句用于在完成正常执行之前突然终止 for 循环，之后程序将会在 for 循环下一行代码开始执行。

让我们写一个从 1 打印到 5 并且使用 break 跳出循环的程序。
```
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i > 5 {
            break //loop is terminated if i > 5
        }
        fmt.Printf("%d ", i)
    }
    fmt.Printf("\nline after for loop")
}
```
打印结果为：
```
1 2 3 4 5  
line after for loop
```

# continue
continue 语句用来跳出 for 循环中当前循环。在 continue 语句后的所有的 for 循环语句都不会在本次循环中执行。循环体会在一下次循环中继续执行。

让我们写一个打印出 1 到 10 并且使用 continue 的程序。
```
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Printf("%d ", i)
    }
}
```
打印结果为：
```
1 3 5 7 9
```
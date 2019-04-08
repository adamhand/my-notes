# golang--常量
---
# 定义
常量使用const关键字修饰，初始化之后不能再重新赋值为其他的值。常量的值会在编译的时候确定。因为函数调用发生在运行时，所以不能将函数的返回值赋值给常量。
```
package main

import (  
    "fmt"
    "math"
)

func main() {  
    fmt.Println("Hello, playground")
    var a = math.Sqrt(4)   // 允许
    const b = math.Sqrt(4) // 不允许
}
```

# 常用常量
## 字符串常量


[参考](https://studygolang.com/articles/11872)




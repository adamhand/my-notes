﻿# golang--指针
---

指针是一种存储变量内存地址（Memory Address）的变量。

# 指针的声明
指针变量的类型为 *T，该指针指向一个 T 类型的变量。
```
package main

import (
    "fmt"
)

func main() {
    b := 255
    var a *int = &b
    fmt.Printf("Type of a is %T\n", a)
    fmt.Println("address of b is", a)
}
```
& 操作符用于获取变量的地址。程序输出：
```
Type of a is *int  
address of b is 0x1040a124
```

# 指针的零值（Zero Value）
指针的零值是 nil。
```
package main

import (  
    "fmt"
)

func main() {  
    a := 25
    var b *int
    if b == nil {
        fmt.Println("b is", b)
        b = &a
        fmt.Println("b after initialization is", b)
    }
}
```

上面的程序中，b 初始化为 nil，接着将 a 的地址赋值给 b。程序会输出：
```
b is <nil>  
b after initialisation is 0x1040a124
```

# 指针的解引用
指针的解引用可以获取指针所指向的变量的值。将 a 解引用的语法是 *a。
```
package main  
import (  
    "fmt"
)

func main() {  
    b := 255
    a := &b
    fmt.Println("address of b is", a)
    fmt.Println("value of b is", *a)
}
```

程序会输出：
```
address of b is 0x1040a124  
value of b is 255
```
我们再编写一个程序，用指针来修改 b 的值。
```
package main

import (  
    "fmt"
)

func main() {  
    b := 255
    a := &b
    fmt.Println("address of b is", a)
    fmt.Println("value of b is", *a)
    *a++
    fmt.Println("new value of b is", b)
}
```

在上面程序的第 12 行中，我们把 a 指向的值加 1，由于 a 指向了 b，因此 b 的值也发生了同样的改变。于是 b 的值变为 256。程序会输出：
```
address of b is 0x1040a124  
value of b is 255  
new value of b is 256
```

# 向函数传递指针参数
```
package main

import (  
    "fmt"
)

func change(val *int) {  
    *val = 55
}
func main() {  
    a := 58
    fmt.Println("value of a before function call is",a)
    b := &a
    change(b)
    fmt.Println("value of a after function call is", a)
}
```
该程序会输出：
```
value of a before function call is 58  
value of a after function call is 55
```

# 不要向函数传递数组的指针，而应该使用切片
假如我们想要在函数内修改一个数组，并希望调用函数的地方也能得到修改后的数组，一种解决方案是把一个指向数组的指针传递给这个函数。
```
package main

import (  
    "fmt"
)

func modify(arr *[3]int) {  
    (*arr)[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```
**a[x] 是 (*a)[x] 的简写形式，因此上面代码中的 (*arr)[0] 可以替换为 arr[0]。**上面的代码输出为：`[90 90 91]`

**这种方式向函数传递一个数组指针参数，并在函数内修改数组。尽管它是有效的，但却不是 Go 语言惯用的实现方式。我们最好使用切片来处理。**

接下来我们用切片来重写之前的代码。
```
package main

import (  
    "fmt"
)

func modify(sls []int) {  
    sls[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(a[:])
    fmt.Println(a)
}
```

# Go 不支持指针运算
Go 并不支持其他语言（例如 C）中的指针运算。
```
package main

func main() {  
    b := [...]int{109, 110, 111}
    p := &b
    p++
}
```

上面的程序会抛出编译错误：main.go:6: invalid operation: p++ (non-numeric type *[3]int)。
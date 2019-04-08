# golang--可变参数函数
---

# 语法
如果函数最后一个参数被记作 `...T` ，这时函数可以接受任意个 T 类型参数作为最后一个参数。

请注意只有函数的最后一个参数才允许是可变的。

# 例子
append 函数的实现中就使用了恶变参数函数，使得它可以将任意个参数值加入到切片中的。它的定义如下：
```
func append(slice []Type, elems ...Type) []Type
```
下面是一个例子：
```
package main

import (
    "fmt"
)

func find(num int, nums ...int) {
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {
    find(89, 89, 90, 95)
    find(45, 56, 67, 45, 90, 109)
    find(78, 38, 56, 98)
    find(87)
}
```
**可变参数函数的工作原理是把可变参数转换为一个新的切片。以上面程序中的第 22 行为例，find 函数中的可变参数是 89，90，95 。 find 函数接受一个 int 类型的可变参数。因此这三个参数被编译器转换为一个 int 类型切片 int []int{89, 90, 95} 然后被传入 find函数。**

上述程序的结果如下：
```
type of nums is []int
89 found at index 0 in [89 90 95]

type of nums is []int
45 found at index 2 in [56 67 45 90 109]

type of nums is []int
78 not found in  [38 56 98]

type of nums is []int
87 not found in  []
```

# 给可变参数函数传入切片
给可变参数函数传入切片，编译件无法通过，因为可变参数`...type`表示可以接受的是多个type类型的参数，而传入切片时却传入的是`[]type`类型的，所以编译不通过，比如下面的程序：
```
package main

import (
    "fmt"
)

func find(num int, nums ...int) {
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {
    nums := []int{89, 90, 95}
    find(89, nums)
}
```
如何想将切片传入可变参数，**有一个可以直接将切片传入可变参数函数的语法糖，你可以在在切片后加上 `...` 后缀。如果这样做，切片将直接传入函数，不再创建新的切片**。

在上面的程序中，如果将第 23 行的 find(89, nums) 替换为 find(89, nums...) ，程序将成功编译并有如下输出
```
type of nums is []int
89 found at index 0 in [89 90 95]
```

# 不直观的错误
下面让我们来看一个简单的例子。
```
package main

import (
    "fmt"
)

func change(s ...string) {  
    s[0] = "Go"
}

func main() {
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome)
}
```

在第 13 行，我们使用了语法糖 ... 并且将切片作为可变参数传入 change 函数。

正如前面我们所讨论的，**如果使用了 ... ，welcome 切片本身会作为参数直接传入，不需要再创建一个新的切片。这样参数 welcome 将作为参数传入 change 函数**。

在 change 函数中，切片的第一个元素被替换成 Go，这样程序产生了下面的输出值
```
[Go world]
```

这里还有一个例子来理解可变参数函数。
```
package main

import (
    "fmt"
)

func change(s ...string) {
    s[0] = "Go"
    s = append(s, "playground")
    fmt.Println(s)
}

func main() {
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome)
}
```
程序的输出结果为：
```
[Go world playground]
[Go world]
```
GO 语言的参数传递都是 **值传递**。 slice 包含 3部分： 长度、容量和指向数组第零个元素的指针，所谓的值传递怎么理解呢，就是传递slice 的时候，把这三个值拷一个副本，传递过去。注意：指针作为值拷贝的副本，指向的是同一个地址，所以修改地址的内容时，原slice也就随之改变。 反之，对拷贝slice副本的修改，如：append，改变的是副本的len、cap，原slice的len、cap并不受影响。
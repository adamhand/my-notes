# golang--数组和切片
---

# 数组声明
一个数组的表示形式为 [n]T。n 表示数组中元素的数量，T 代表每个元素的类型。数组的索引从 0 开始到 length - 1 结束。
```
package main

import (
    "fmt"
)

func main() {
    var a [3]int //int array with length 3
    fmt.Println(a)
}
```
还可以简略声明 来创建相同的数组。
```
package main

import (
    "fmt"
)

func main() {
    a := [3]int{12, 78, 50} // short hand declaration to create array
    fmt.Println(a)
}
```
可以忽略声明数组的长度，并用 ... 代替，让编译器为你自动计算长度，这在下面的程序中实现。
```
package main

import (
    "fmt"
)

func main() {
    a := [...]int{12, 78, 50} // ... makes the compiler determine the length
    fmt.Println(a)
}
```
需要注意的是，**数组的大小是类型的一部分。因此 [5]int 和 [25]int 是不同类型。数组不能调整大小**。
```
package main

func main() {
    a := [3]int{5, 78, 8}
    var b [5]int
    b = a // not possible since [3]int and [5]int are distinct types
}
```
# 数组是值类型
**Go 中的数组是值类型而不是引用类型。这意味着当数组赋值给一个新的变量时，该变量会得到一个原始数组的一个副本。如果对新变量进行更改，则不会影响原始数组。**
```
package main

import "fmt"

func main() {
    a := [...]string{"USA", "China", "India", "Germany", "France"}
    b := a // a copy of a is assigned to b
    b[0] = "Singapore"
    fmt.Println("a is ", a)
    fmt.Println("b is ", b) 
}
```
输出结果为：
```
a is [USA China India Germany France]  
b is [Singapore China India Germany France]
```
同样，当数组作为参数传递给函数时，它们是按值传递，而原始数组保持不变。
```
package main

import "fmt"

func changeLocal(num [5]int) {
    num[0] = 55
    fmt.Println("inside function ", num)
}
func main() {
    num := [...]int{5, 6, 7, 8, 8}
    fmt.Println("before passing to function ", num)
    changeLocal(num) //num is passed by value
    fmt.Println("after passing to function ", num)
}
```
结果为：
```
before passing to function  [5 6 7 8 8]
inside function  [55 6 7 8 8]
after passing to function  [5 6 7 8 8]
```

# 数组的长度
通过将数组作为参数传递给 len 函数，可以得到数组的长度。
```
package main

import "fmt"

func main() {
    a := [...]float64{67.7, 89.8, 21, 78}
    fmt.Println("length of a is",len(a))
}
```
上面的程序输出为 length of a is 4。

# 使用 range 迭代数组
range 返回索引和该索引处的值。
```
package main

import "fmt"

func main() {
    a := [...]float64{67.7, 89.8, 21, 78}
    sum := float64(0)
    for i, v := range a {//range returns both the index and value
        fmt.Printf("%d the element of a is %.2f\n", i, v)
        sum += v
    }
    fmt.Println("\nsum of all elements of a",sum)
}
```
结果为：
```
0 the element of a is 67.70
1 the element of a is 89.80
2 the element of a is 21.00
3 the element of a is 78.00

sum of all elements of a 256.5
```
如果你只需要值并希望忽略索引，则可以通过用 _ 空白标识符替换索引来执行。
```
for _, v := range a { // ignores index  
}
```
上面的 for 循环忽略索引，同样值也可以被忽略。

# 多维数组
Go 语言可以创建多维数组。创建二维数组的基本语法为：
```
var name [row][col]type
```
例如：
```
package main

import (
    "fmt"
)

func printarray(a [3][2]string) {
    for _, v1 := range a {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}

func main() {
    a := [3][2]string{
        {"lion", "tiger"},
        {"cat", "dog"},
        {"pigeon", "peacock"}, // 逗号是必须的
    }
    printarray(a)
    var b [3][2]string
    b[0][0] = "apple"
    b[0][1] = "samsung"
    b[1][0] = "microsoft"
    b[1][1] = "google"
    b[2][0] = "AT&T"
    b[2][1] = "T-Mobile"
    fmt.Printf("\n")
    printarray(b)
}
```
输出结果为：
```
lion tiger
cat dog
pigeon peacock

apple samsung
microsoft google
AT&T T-Mobile
```

# 切片
数组的长度不能更改，但是切片可以。切片本身不拥有任何数据。它们只是对现有数组的引用。

带有 T 类型元素的切片由 []T 表示
```
package main

import (
    "fmt"
)

func main() {
    a := [5]int{76, 77, 78, 79, 80}
    var b []int = a[1:4] // creates a slice from a[1] to a[3]
    fmt.Println(b)
}
```
切片 b 的值为 [77 78 79]，可以看到，切片也是半开半闭区间。

创建切片还有另一种方法，创建数组时，不声明数组的长度，也不用`...`修饰，创建的就是切片。
```
package main

import (  
    "fmt"
)

func main() {  
    c := []int{6, 7, 8} // creates and array and returns a slice reference
    fmt.Println(c)
}
```

# 切片的修改
切片自己不拥有任何数据。它只是底层数组的一种表示。对切片所做的任何修改都会反映在底层数组中。
```
package main

import (
    "fmt"
)

func main() {
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before", darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after", darr)
    fmt.Println("array after", dslice)
}
```
打印结果为：
```
array before [57 89 90 82 100 78 67 69 59]
array after [57 89 91 83 101 78 67 69 59]
array after [91 83 101]
```

当多个切片共用相同的底层数组时，每个切片所做的更改将反映在数组中。
```
package main

import (
    "fmt"
)

func main() {
    numa := [3]int{78, 79 ,80}
    nums1 := numa[:] // creates a slice which contains all elements of the array
    nums2 := numa[:]
    fmt.Println("array before change 1", numa)
    nums1[0] = 100
    fmt.Println("array after modification to slice nums1", numa)
    nums2[1] = 101
    fmt.Println("array after modification to slice nums2", numa)
}
```
在 9 行中，numa [:] 缺少开始和结束值。开始和结束的默认值分别为 0 和 len (numa)。两个切片 nums1 和 nums2 共享相同的数组。该程序的输出是
```
array before change 1 [78 79 80]  
array after modification to slice nums1 [100 79 80]  
array after modification to slice nums2 [100 101 80]
```

# 切片的长度和容量
切片的长度是切片中的元素数。**切片的容量是从创建切片索引开始的底层数组中元素数。**
```
package main

import (
    "fmt"
)

func main() {
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d", len(fruitslice), cap(fruitslice)) // length of is 2 and capacity is 6
}
```
在上面的程序中，fruitslice 是从 fruitarray 的索引 1 和 2 创建的。 因此，fruitlice 的长度为 2。

fruitarray 的长度是 7。fruiteslice 是从 fruitarray 的索引 1 创建的。因此, fruitslice 的容量是从 fruitarray 索引为 1 开始，也就是说从 orange 开始，该值是 6。因此, fruitslice 的容量为 6。该程序输出切片的 长度为 2 容量为 6 。

# 使用 make 创建一个切片
func make（[]T，len，cap）[]T 通过传递类型，长度和容量来创建切片。容量是可选参数, 默认值为切片长度。make 函数创建一个数组，并返回引用该数组的切片。
```
package main

import (
    "fmt"
)

func main() {
    i := make([]int, 5, 5)
    fmt.Println(i)
}
```

使用 make 创建切片时默认情况下这些值为零。上述程序的输出为 [0 0 0 0 0]。

# 追加切片元素
切片是动态的，使用 append 可以将新元素追加到切片上。append 函数的定义是 func append（s[]T，x ... T）[]T。

当新的元素被添加到切片时，会创建一个新的数组。现有数组的元素被复制到这个新数组中，并返回这个新数组的新切片引用。现在新切片的容量是旧切片的两倍。
```
package main

import (
    "fmt"
)

func main() {
    cars := []string{"Ferrari", "Honda", "Ford"}
    fmt.Println("cars:", cars, "has old length", len(cars), "and capacity", cap(cars)) // capacity of cars is 3
    cars = append(cars, "Toyota")
    fmt.Println("cars:", cars, "has new length", len(cars), "and capacity", cap(cars)) // capacity of cars is doubled to 6
}
```
输出结果为：
```
cars: [Ferrari Honda Ford] has old length 3 and capacity 3  
cars: [Ferrari Honda Ford Toyota] has new length 4 and capacity 6
```

也可以使用 ... 运算符将一个切片添加到另一个切片。 你可以在可变参数函数教程中了解有关此运算符的更多信息。
```
package main

import (
    "fmt"
)

func main() {
    veggies := []string{"potatoes", "tomatoes", "brinjal"}
    fruits := []string{"oranges", "apples"}
    food := append(veggies, fruits...)
    fmt.Println("food:",food)
}
```

在上述程序的第 10 行，food 是通过 append(veggies, fruits...) 创建。程序的输出为 food: [potatoes tomatoes brinjal oranges apples]。

# 切片的函数传递
我们可以认为，切片在内部可由一个结构体类型表示。这是它的表现形式，
```
type slice struct {  
    Length        int
    Capacity      int
    ZerothElement *byte
}
```
切片包含长度、容量和指向数组第零个元素的指针。当切片传递给函数时，即使它通过值传递，指针变量也将引用相同的底层数组。因此，当切片作为参数传递给函数时，函数内所做的更改也会在函数外可见。**这是不同于数组的，对于函数中一个数组的变化在函数外是不可见的。**
```
package main

import (
    "fmt"
)

func subtactOne(numbers []int) {
    for i := range numbers {
        numbers[i] -= 2
    }
}
func main() {
    nos := []int{8, 7, 6}
    fmt.Println("slice before function call", nos)
    subtactOne(nos)                               // function modifies the slice
    fmt.Println("slice after function call", nos) // modifications are visible outside
}
```
执行结果为：
```
array before function call [8 7 6]  
array after function call [6 5 4]
```

# 多维切片
类似于数组，切片可以有多个维度。
```
package main

import (
    "fmt"
)

func main() {  
     pls := [][]string {
            {"C", "C++"},
            {"JavaScript"},
            {"Go", "Rust"},
            }
    for _, v1 := range pls {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}
```
程序的输出为，
```
C C++  
JavaScript  
Go Rust
```

# 内存优化
切片持有对底层数组的引用。只要切片在内存中，数组就不能被垃圾回收。一种解决方法是使用 copy 函数 func copy(dst，src[]T)int 来生成一个切片的副本。这样我们可以使用新的切片，原始数组可以被垃圾回收。
```
package main

import (
    "fmt"
)

func countries() []string {
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
    neededCountries := countries[:len(countries)-2]
    countriesCpy := make([]string, len(neededCountries))
    copy(countriesCpy, neededCountries) //copies neededCountries to countriesCpy
    return countriesCpy
}
func main() {
    countriesNeeded := countries()
    fmt.Println(countriesNeeded)
}
```
在上述程序的第 9 行，neededCountries := countries[:len(countries)-2 创建一个去掉尾部 2 个元素的切片 countries，在上述程序的 11 行，将 neededCountries 复制到 countriesCpy 同时在函数的下一行返回 countriesCpy。现在 countries 数组可以被垃圾回收, 因为 neededCountries 不再被引用。
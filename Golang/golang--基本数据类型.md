# golang--基本数据类型
---

# golang的数据类型
golang的数据类型大概如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/golang-type.png" width="500">
</center>

这里主要谈论基本数据类型：数字、字符串和布尔。它们又可以细分为如下：

- bool
- 数字类型
    - 整型
        - int8, int16, int32, int64, int
        - uint8, uint16, uint32, uint64, uint
    - 浮点型
        - float32, float64
    - 复数
        - complex64, complex128
    - byte
    - rune
    - uintptr
- string

# 数字类型
## 整型
### 有符号整型
int8、int16、int32和int64分别表示8位、16位、32位和64位的有符号整型。而int表示的位数和编译器以及平台有关，一般在32位系统上为32位，在64位系统上为64位。**注意，int和int32或64并不等价，虽然它们表示的位数是相同的，它们之间转换时需要显式类型转换**。

```golang
package main

import (  
    "fmt"
    "unsafe"
)

func main() {  
    var a int = 89
    b := 95
    fmt.Println("value of a is", a, "and b is", b)
    fmt.Printf("type of a is %T, size of a is %d", a, unsafe.Sizeof(a)) // a 的类型和大小
    fmt.Printf("\ntype of b is %T, size of b is %d", b, unsafe.Sizeof(b)) // b 的类型和大小
}
```
在32位系统上，以上程序会输出：
```
value of a is 89 and b is 95  
type of a is int, size of a is 4  
type of b is int, size of b is 4
```

### 无符号整型
uint8、uint16、uint32和uint64分别表示8位、16位、32位和64位的无符号整型。uint表示的位数和编译器以及平台有关，一般在32位系统上为32位，在64位系统上为64位。

## 浮点型

- float32：32 位浮点数
- float64：64 位浮点数(默认)

下面一个简单程序演示了整型和浮点型的运用。
```golang
package main

import (  
    "fmt"
)

func main() {  
    a, b := 5.67, 8.97
    fmt.Printf("type of a %T b %T\n", a, b)
    sum := a + b
    diff := a - b
    fmt.Println("sum", sum, "diff", diff)

    no1, no2 := 56, 89
    fmt.Println("sum", no1+no2, "diff", no1-no2)
}
```
结果为：
```
type of a float64 b float64  
sum 14.64 diff -3.3000000000000007  
sum 145 diff -33
```

## 复数类型

- complex64：实部和虚部都是 float32 类型的的复数。
- complex128：实部和虚部都是 float64 类型的的复数。

内建函数 complex 用于创建一个包含实部和虚部的复数。complex 函数的定义如下：
```
func complex(r, i FloatType) ComplexType
```
该函数的参数分别是实部和虚部，并返回一个复数类型。实部和虚部应该是相同类型，也就是 float32 或 float64。如果实部和虚部都是 float32 类型，则函数会返回一个 complex64 类型的复数。如果实部和虚部都是 float64 类型，则函数会返回一个 complex128 类型的复数。

还可以使用简短语法来创建复数：
```
c := 6 + 7i
```
下面我们编写一个简单的程序来理解复数。
```golang
package main

import (  
    "fmt"
)

func main() {  
    c1 := complex(5, 7)
    c2 := 8 + 27i
    cadd := c1 + c2
    fmt.Println("sum:", cadd)
    cmul := c1 * c2
    fmt.Println("product:", cmul)
}
```
结果为：
```
sum: (13+34i)  
product: (-149+191i)
```
## 其他数字类型

- byte 是 uint8 的别名。
- rune 是 int32 的别名。
- uintptr是一种无符号整型，大小并不明确，但是足以完整存放指针。仅仅用于底层编程。

# string 类型
string类型是不可变的，和Java相似，每对一个string类型的变量进行操作，都会生成一个新的变量。

两个字符串连接操作可以使用加号"+"来实现，如下：
```golang
package main

import (  
    "fmt"
)

func main() {  
    first := "Naveen"
    last := "Ramanathan"
    name := first +" "+ last
    fmt.Println("My name is",name)
}
```
结果为：
```
My name is Naveen Ramanathan
```

# 类型转换
Go 没有自动类型提升或类型转换。比如下面的例子会出现错误：
```golang
package main

import (  
    "fmt"
)

func main() {  
    i := 55      //int
    j := 67.8    //float64
    sum := i + j //不允许 int + float64
    fmt.Println(sum)
}
```
可以使用强制类型转换，把上面的例子改为下面所示，就没有错误了。
```golang
package main

import (  
    "fmt"
)

func main() {  
    i := 55      //int
    j := 67.8    //float64
    sum := i + int(j) //j is converted to int
    fmt.Println(sum)
}
```
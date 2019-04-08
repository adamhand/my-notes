# golang--函数
---
# 声明
函数声明通用语法如下：
```golang
func functionname(parametername type) [returntype] {  
    // 函数体（具体实现的功能）
}
```
函数中的参数列表和返回值并非是必须的，所以下面这个函数的声明也是有效的
```
func functionname() {  
    // 译注: 表示这个函数不需要输入参数，且没有返回值
}
```
# 示例函数
我们以写一个计算商品价格的函数为例，输入参数是单件商品的价格和商品的个数，两者的乘积为商品总价，作为函数的输出值。

注意：**如果有连续若干个参数，它们的类型一致，那么我们无须一一罗列，只需在最后一个参数后添加该类型。**
```
func calculateBill(price, no int) int {  
    var totalPrice = price * no
    return totalPrice
}
```

# 多返回值
Go 语言支持一个函数可以有多个返回值。我们来写个以矩形的长和宽为输入参数，计算并返回矩形面积和周长的函数 rectProps。矩形的面积是长度和宽度的乘积, 周长是长度和宽度之和的两倍。即：

- 面积 = 长 * 宽
- 周长 = 2 * ( 长 + 宽 )

```
package main

import (  
    "fmt"
)

func rectProps(length, width float64)(float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}

func main() {  
    area, perimeter := rectProps(10.8, 5.6)
    fmt.Printf("Area %f Perimeter %f", area, perimeter) 
}
```
结果为：
```
Area 60.480000 Perimeter 32.800000
```

# 命名返回值
从函数中可以返回一个命名值。一旦命名了返回值，可以认为这些值在函数第一行就被声明为变量了。

上面的 rectProps 函数也可用这个方式写成：
```
func rectProps(length, width float64)(area, perimeter float64) {  
    area = length * width
    perimeter = (length + width) * 2
    return // 不需要明确指定返回值，默认返回 area, perimeter 的值
}
```
请注意, 函数中的 return 语句没有显式返回任何值。由于 area 和 perimeter 在函数声明中指定为返回值, 因此当遇到 return 语句时, 它们将自动从函数返回。

# 空白符
_ 在 Go 中被用作空白符，可以用作表示任何类型的任何值。

我们继续以 rectProps 函数为例，该函数计算的是面积和周长。假使我们只需要计算面积，而并不关心周长的计算结果，该怎么调用这个函数呢？这时，空白符 _ 就上场了。

下面的程序我们只用到了函数 rectProps 的一个返回值 area
```
package main

import (  
    "fmt"
)

func rectProps(length, width float64) (float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}
func main() {  
    area, _ := rectProps(10.8, 5.6) // 返回值周长被丢弃
    fmt.Printf("Area %f ", area)
}
```
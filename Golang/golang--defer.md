# golang--defer
---
defer关键字的含义是：**延迟函数（`Deferred Function`）**。

defer 语句的用途是：含有 defer 语句的函数，会在该函数将要返回之前，调用另一个函数。

# 例子
```
package main

import (
	"fmt"
)

func finished()  {
	fmt.Println("function has finished")
}

func findLagest(nums []int)  {
	defer finished()
	max := nums[0]
	for _, v := range nums{
		if v > max{
			max = v
		}
	}
	fmt.Println("the largest number is ", max)
}

func main()  {
	nums := []int{1,2,3,4,5,6,7,8,9}
	findLagest(nums)
}
```
上述`findLagest`函数在执行完之后会调用`finished`函数。上述程序的执行结果为：
```
the largest number is  9
function has finished
```

# 延迟方法
defer 不仅限于**函数**的调用，调用**方法**也是合法的。
```
package main

import "fmt"

type person struct {
	firstName string
	lastName string
}

func (p person)fullName()  {
	fmt.Printf("%s", p.firstName + p.lastName)
}

func main()  {
	p := person{
		firstName:"Bob",
		lastName:"Steven",
	}
	defer p.fullName()
	fmt.Print("welcome ")
}
```
上述程序的执行结果为：
```
welcome BobSteven
```

# 实参取值
在 Go 语言中，并非在调用延迟函数的时候才确定实参，而是当**执行 defer 语句的时候，就会对延迟函数的实参进行求值**。
```
package main

import "fmt"

func printValue(v int)  {
	fmt.Printf("value of v is %d\n", v)
}

func main()  {
	v := 5
	defer printValue(v)
	v = 10
	fmt.Printf("value of v is %d\n", v)
}
```

# defer 栈
当一个函数内多次调用 defer 时，Go 会把 defer 调用放入到一个栈中，随后按照后进先出（Last In First Out, LIFO）的顺序执行。

```
package main

import "fmt"

func main()  {
	name := "stevem bob"
	fmt.Printf("name is: %s\n", name)
	fmt.Printf("reverse name is: ")

	for _, v := range []rune(name){
		defer fmt.Printf("%c", v)
	}
}
```
上述程序的执行结果为：
```
name is: stevem bob
reverse name is: bob mevets
```

# defer 的实际应用
**当一个函数应该在与当前代码流（Code Flow）无关的环境下调用时，可以使用 defer。**

下面首先会写一个没有使用 defer 的程序，然后用 defer 来修改，看到 defer 带来的好处。

```
package main

import (
	"fmt"
	"sync"
)

type rect struct {
	length int
	width int
}

func (r rect)area(wg *sync.WaitGroup)  {
	if r.length < 0{
		fmt.Printf("length should greater than zero\n")
		wg.Done()
		return
	}

	if r.width < 0{
		fmt.Printf("width should greater than zero\n")
		wg.Done()
		return
	}

	area := r.length * r.width
	fmt.Printf("area is %d\n", area)
	wg.Done()
}

func main()  {
	var wg sync.WaitGroup
	r1 := rect{-67, 89}
	r2 := rect{5, -67}
	r3 := rect{8, 9}
	rects := []rect{r1, r2, r3}
	for _, v := range rects {
		wg.Add(1)
		go v.area(&wg)
	}
	wg.Wait()
	fmt.Println("All go routines finished executing")
}
```
仔细观察上述程序，会发现 **`wg.Done()` 只在 `area` 函数返回的时候才会调用。`wg.Done()` 应该在 `area` 将要返回之前调用，并且与代码流的路径（`Path`）无关，因此我们可以只调用一次 `defer`，来有效地替换掉 `wg.Done()` 的多次调用**。

下面使用`defer`来改写程序中的`area()`函数。
```
func (r rect)area(wg *sync.WaitGroup)  {
	defer wg.Done()
	if r.length < 0{
		fmt.Printf("length should greater than zero\n")
		return
	}

	if r.width < 0{
		fmt.Printf("width should greater than zero\n")
		return
	}

	area := r.length * r.width
	fmt.Printf("area is %d\n", area)
}
```
更改后的程序可以和之前的程序达到相同的效果。


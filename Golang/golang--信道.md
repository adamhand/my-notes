# golang--信道
---

信道可以想像成 Go 协程之间通信的管道。如同管道中的水会从一端流到另一端，通过使用信道，数据也可以从一端发送，在另一端接收。
# 信道的声明
所有信道都关联了一个类型。信道只能运输这种类型的数据，而运输其他类型的数据都是非法的。

`chan T` 表示 `T` 类型的信道。

信道的零值为 nil。信道的零值没有什么用，应该像对 map 和切片所做的那样，用 make 来定义信道。
```
package main

import "fmt"

func main() {  
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```
程序输出为：
```
channel a is nil, going to define it  
Type of a is chan int
```
简短声明通常也是一种定义信道的简洁有效的方法。
```
a := make(chan int)
```

# 通过信道进行发送和接收
如下所示，该语法通过信道发送和接收数据。
```
data := <- a // 读取信道 a  
a <- data // 写入信道 a
```
信道旁的箭头方向指定了是发送数据还是接收数据。

在第一行，箭头对于 a 来说是向外指的，因此我们读取了信道 a 的值，并把该值存储到变量 data。

在第二行，箭头指向了 a，因此我们在把数据写入信道 a。

发送与接收默认是阻塞的。**当把数据发送到信道时，程序控制会在发送数据的语句处发生阻塞，直到有其它 Go 协程从信道读取到数据，才会解除阻塞。与此类似，当读取信道的数据时，如果没有其它的协程把数据写入到这个信道，那么读取过程就会一直阻塞着**。

信道的这种特性能够帮助 Go 协程之间进行高效的通信，不需要用到其他编程语言常见的显式锁或条件变量。

# 代码示例
```
package main

import (  
    "fmt"
)

func hello(done chan bool) {  
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {  
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```
在程序的第9行，向信道done中写入true，协程会一直阻塞在这里，直到在14行将done中的数据读出来。

该程序输出如下：
```
Hello world goroutine  
main function
```

# 另一个例子
下面的程序会计算一个数中每一位的平方和与立方和，然后把平方和与立方和相加并打印出来。

例如，如果输出是 123，该程序会如下计算输出：
```
squares = (1 * 1) + (2 * 2) + (3 * 3) 
cubes = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3) 
output = squares + cubes = 50
```

```
package main

import (  
    "fmt"
)

func calcSquares(number int, squareop chan int) {  
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0 
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
} 

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```
在第 7 行，函数 calcSquares 计算一个数每位的平方和，并把结果发送给信道 squareop。与此类似，在第 17 行函数 calcCubes 计算一个数每位的立方和，并把结果发送给信道 cubop。

改程序的输出如下：
```
Final output 1536
```

# 死锁
使用信道需要考虑的一个重点是死锁。当 Go 协程给一个信道发送数据时，照理说会有其他 Go 协程来接收数据。如果没有的话，程序就会在运行时触发 panic，形成死锁。

同理，当有 Go 协程等着从一个信道接收数据时，我们期望其他的 Go 协程会向该信道写入数据，要不然程序就会触发 panic。

# 单向信道
我们目前讨论的信道都是双向信道，即通过信道既能发送数据，又能接收数据。其实也可以创建单向信道，这种信道只能发送或者接收数据。
```
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```

上面程序的第 10 行，我们创建了唯送（`Send Only`）信道 `sendch`。`chan<- int` 定义了唯送信道，因为箭头指向了 `chan`。在第 12 行，我们试图通过唯送信道接收数据，于是编译器报错：

`main.go:11: invalid operation: <-sendch (receive from send-only type chan<- int)`
一切都很顺利，只不过一个不能读取数据的唯送信道究竟有什么意义呢？

**这就需要用到信道转换（`Channel Conversion`）了。把一个双向信道转换成唯送信道或者唯收（`Receive Only`）信道都是行得通的，但是反过来就不行。**

```
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    cha1 := make(chan int)
    go sendData(cha1)
    fmt.Println(<-cha1)
}
```

在上述程序的第 10 行，我们创建了一个双向信道 cha1。在第 11 行 cha1 作为参数传递给了 sendData 协程。在第 5 行，函数 sendData 里的参数 sendch chan<- int 把 cha1 转换为一个唯送信道。于是该信道在 sendData 协程里是一个唯送信道，而在 Go 主协程里是一个双向信道。该程序最终打印输出 10。

# 关闭信道和使用 for range 遍历信道
数据发送方可以关闭信道，通知接收方这个信道不再有数据发送过来。

当从信道接收数据时，接收方可以多用一个变量来检查信道是否已经关闭。
```
v, ok := <- ch
```
上面的语句里，如果成功接收信道所发送的数据，那么 ok 等于 true。而如果 ok 等于 false，说明我们试图读取一个关闭的通道。从关闭的信道读取到的值会是该信道类型的零值。例如，当信道是一个 int 类型的信道时，那么从关闭的信道读取的值将会是 0

```
package main

import "fmt"

func producer(channel chan int)  {
	for i := 0; i < 10; i++{
		channel <- i
	}
	close(channel)
}

func main()  {
	ch := make(chan int)
	go producer(ch)

	for v := range ch{
		fmt.Println("received: ", v)
	}
}
```

在第 16 行，for range 循环从信道 ch 接收数据，直到该信道关闭。一旦关闭了 ch，循环会自动结束。该程序会输出：
```
Received  0  
Received  1  
Received  2  
Received  3  
Received  4  
Received  5  
Received  6  
Received  7  
Received  8  
Received  9
```


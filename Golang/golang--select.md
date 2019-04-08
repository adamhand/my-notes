# golang--select
---

# 什么是 select？
select 语句用于在多个发送/接收信道操作中进行选择。select 语句会一直阻塞，直到发送/接收操作准备就绪。如果有多个信道操作准备完毕，select 会随机地选取其中之一执行。该语法与 switch 类似，所不同的是，这里的每个 case 语句都是信道操作。
```
package main

import (
	"fmt"
	"time"
)

func server1(ch chan string)  {
	time.Sleep(100*time.Millisecond)
	ch <- "server 1"
}

func server2(ch chan string)  {
	time.Sleep(200*time.Millisecond)
	ch <- "server 2"
}

func main()  {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go server1(ch1)
	go server2(ch2)

	select {
	case out1 := <- ch1:
		fmt.Println(out1)
	case out2 := <- ch2:
		fmt.Println(out2)
	}
}
```
上面的程序中，在main函数中创建了两个协程，其中server1会在睡眠100ms之后向channel中写入数据，而server2会在睡眠200ms之后向channel中写入数据。select语句会首先接收到server1的数据，之后就会跳出select语句了。该程序的输出结果为：
```
server 1
```

# select 的应用
在上面程序中，函数之所以取名为 server1 和 server2，是为了展示 select 的实际应用。

假设我们有一个关键性应用，需要尽快地把输出返回给用户。这个应用的数据库复制并且存储在世界各地的服务器上。假设函数 server1 和 server2 与这样不同区域的两台服务器进行通信。每台服务器的负载和网络时延决定了它的响应时间。我们向两台服务器发送请求，并使用 select 语句等待相应的信道发出响应。select 会选择首先响应的服务器，而忽略其它的响应。使用这种方法，我们可以向多个服务器发送请求，并给用户返回最快的响应了。

# 默认情况与死锁
在没有 case 准备就绪时，可以执行 select 语句中的默认情况（Default Case）。这通常用于防止 select 语句一直阻塞。

```
package main

import (
	"fmt"
	"time"
)

func process(ch chan string)  {
	time.Sleep(1000 * time.Millisecond)
	ch <- "process successful"
}

func main()  {
	ch := make(chan string)
	go process(ch)
	
	for {
		time.Sleep(200 * time.Millisecond)
		select {
		case v := <-ch:
			fmt.Println("received value ", v)
			return
		default:
			fmt.Println("no value received")

		}
	}
}
```
上述程序中，process方法先睡眠1000ms，然后向信道中写入数据。在main函数中，以协程的形式调用process函数，然后启用一个无限循环，在循环中先让协程睡眠200ms，然后执行select语句，当没有在信道中收到消息时，打印`no value received`，否则打印收到的值。

程序的执行结果如下：
```
no value received
no value received
no value received
no value received
received value  process successful
```

**如果在试图读取信道 ch。没有 Go 协程向该信道写入数据，select 语句会一直阻塞，导致死锁，然后会触发运行时 panic。如果存在默认情况，就不会发生死锁，因为在没有其他 case 准备就绪时，会执行默认情况。**

**如果 select 只含有值为 nil 的信道，也同样会执行默认情况。**如下所示：
```
package main

import "fmt"

func main()  {
	ch := make(chan string)
	select {
	case v := <- ch:
		fmt.Println("received value ", v)
	default:
		fmt.Println("no value received")
	}
}
```
上述程序打印结果如下：
```
no value received
```

# 随机选取
当 select 由多个 case 准备就绪时，将会随机地选取其中之一去执行。
```
package main

import (
	"fmt"
	"time"
)

func server1(ch chan string)  {
	ch <- "server1"
}

func server2(ch chan string)  {
	ch <- "server 2"
}

func main()  {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go server1(ch1)
	go server2(ch2)
	time.Sleep(100 * time.Millisecond)

	select {
	case out1 := <- ch1:
		fmt.Println("received value ", out1)
	case out2 := <- ch2:
		fmt.Println("received value ", out2)
	}
}
```
上述程序中，在22行由于main线程休眠100ms，导致server1和server2信道中的数据都准备好了，在接下来的select语句中就会随机选择两个信道中的数据。

# 空 select
```
package main

func main()  {
	select {}
}
```
除非有 case 执行，select 语句就会一直阻塞着。在这里，select 语句没有任何 case，因此它会一直阻塞，导致死锁。该程序会触发 panic，输出如下：

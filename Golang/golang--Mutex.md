# golang--Mutex
---

# 临界区和竞态
当某个资源可以被多个Go协程共同访问时，这个资源叫做**共享资源**。这段修改共享资源的代码称为**临界区**。当多个Go协程同时执行临界区的代码时，就有可能发生**竞态**。

比如下面的代码：
```
x = x + 1
```
上述代码的执行过程有三步：

- `MOV AX,X`，将x放到累加器
- `INC AX` ，在累加器中将x加1
- `MOV X,AX`，将结果赋值给x

我们假设 x 的初始值为 0。而协程 1 获取 x 的初始值，并计算 x + 1。而在协程 1 将计算值赋值给 x 之前，系统上下文切换到了协程 2。于是，协程 2 获取了 x 的初始值（依然为 0），并计算 x + 1。接着系统上下文又切换回了协程 1。现在，协程 1 将计算值 1 赋值给 x，因此 x 等于 1。然后，协程 2 继续开始执行，把计算值（依然是 1）复制给了 x，因此在所有协程执行完毕之后，x 都等于 1。

# Mutex
Mutex的英文意思是“互斥”。Mutex 用于提供一种加锁机制（Locking Mechanism），可确保在某时刻只有一个协程在临界区运行，以防止出现竞态条件。

Mutex 定义了两个方法：Lock 和 Unlock。所有在 Lock 和 Unlock 之间的代码，都只能由一个 Go 协程执行，于是就可以避免竞态条件。
```
mutex.Lock()  
x = x + 1  
mutex.Unlock()
```
如果有一个 Go 协程已经持有了锁（Lock），当其他协程试图获得该锁时，这些协程会被阻塞，直到 Mutex 解除锁定为止。

# 竞态问题和Mutex
下面的第一个程序有竞态问题，而第二个程序使用Mutex修复了竞态问题。
```
package main

import (
	"fmt"
	"sync"
)

var i = 0

func incresment(wg *sync.WaitGroup)  {
	i = i + 1
	wg.Done()
}

func main()  {
	var wg sync.WaitGroup

	for x := 0; x < 1000; x++{
		wg.Add(1)
		go incresment(&wg)
	}
	wg.Wait()
	fmt.Printf("result is %d", i)
}
```
上面的程序每次输出的结果都不相同，但是大部分时间都不是1000。

下面的程序使用Mutex修复了上述竞态问题。
```
package main

import (
	"fmt"
	"sync"
)

var i = 0

func increment(wg *sync.WaitGroup, lock *sync.Mutex)  {
	lock.Lock()
	i = i + 1
	lock.Unlock()
	wg.Done()
}

func main()  {
	var wg sync.WaitGroup
	var lock sync.Mutex

	for x := 0; x < 1000; x++{
		wg.Add(1)
		go increment(&wg, &lock)
	}
	wg.Wait()
	fmt.Printf("result is %d", i)
}
```
上述程序每次输出均为1000。

需要注意的是，**传递 Mutex 的地址很重要。如果传递的是 Mutex 的值，而非地址，那么每个协程都会得到 Mutex 的一份拷贝，竞态条件还是会发生。**

# 使用信道处理竞态条件
还可以使用信道来处理竞态问题：
```
package main

import (
	"fmt"
	"sync"
)

var x = 0

func increment(wg *sync.WaitGroup, ch chan bool)  {
	ch <- true
	x = x + 1
	<- ch
	wg.Done()
}

func main()  {
	var wg sync.WaitGroup
	ch := make(chan bool, 1)

	for i := 0; i < 1000; i++{
		wg.Add(1)
		go increment(&wg, ch)
	}
	wg.Wait()
	fmt.Printf("result is %d", x)
}
```
因为缓冲信道的容量为1，所以当一个协程向信道中写入true之后没有读出时，其他线程不能向其中写入数据，就会被阻塞，从而达到同一时刻只有一个协程能够执行临界区代码。

# Mutex VS 信道
对于这两种方式之间的选择，当 Go 协程需要与其他协程通信时，可以使用信道。而当只允许一个协程访问临界区时，可以使用 Mutex。
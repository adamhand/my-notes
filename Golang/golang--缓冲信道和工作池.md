# golang--缓冲信道和工作池
---

# 什么是缓冲信道？
无缓冲信道的发送和接收过程是阻塞的；除此之外，还可以创建一个有缓冲（Buffer）的信道。**只在缓冲已满的情况，才会阻塞向缓冲信道（Buffered Channel）发送数据。同样，只有在缓冲为空的时候，才会阻塞从缓冲信道接收数据。**

通过向 make 函数再传递一个表示容量的参数（指定缓冲的大小），可以创建缓冲信道。
```
ch := make(chan type, capacity)
```
要让一个信道有缓冲，上面语法中的 capacity 应该大于 0。无缓冲信道的容量默认为 0。

看一个例子：
```
package main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}
```
在上面的程序中，第 16 行在 Go 主协程中创建了容量为 2 的缓冲信道 ch，而第 17 行把 ch 传递给了 write 协程。接下来 Go 主协程休眠了两秒。在这期间，write 协程在并发地运行。write 协程有一个 for 循环，依次向信道 ch 写入 0～4。而缓冲信道的容量为 2，因此 write 协程里立即会向 ch 写入 0 和 1，接下来发生阻塞，直到 ch 内的值被读取。因此，该程序立即打印出下面两行：
```
successfully wrote 0 to ch  
successfully wrote 1 to ch
```
打印上面两行之后，write 协程中向 ch 的写入发生了阻塞，直到 ch 有值被读取到。而 Go 主协程休眠了两秒后，才开始读取该信道，因此在休眠期间程序不会打印任何结果。主协程结束休眠后，在第 19 行使用 for range 循环，开始读取信道 ch，打印出了读取到的值后又休眠两秒，这个循环一直到 ch 关闭才结束。所以该程序在两秒后会打印下面两行：
```
read value 0 from ch  
successfully wrote 2 to ch
```
该过程会一直进行，直到信道读取完所有的值，并在 write 协程中关闭信道。最终输出如下：
```
successfully wrote 0 to ch  
successfully wrote 1 to ch  
read value 0 from ch  
successfully wrote 2 to ch  
read value 1 from ch  
successfully wrote 3 to ch  
read value 2 from ch  
successfully wrote 4 to ch  
read value 3 from ch  
read value 4 from ch
```

# 死锁
当信道被阻塞之后，比如信道为空或者信道满，如果没有改变这种阻塞情况的方式，比如信道为空时没有协程往里写数据，信道为满时没有协程往外读数据，就会发生**死锁（deadlock）**。

# 长度 vs 容量
**缓冲信道的容量是指信道可以存储的值的数量。我们在使用 make 函数创建缓冲信道的时候会指定容量大小。**

**缓冲信道的长度是指信道中当前排队的元素个数。**

```
package main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch))
    fmt.Println("length is", len(ch))
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch))
}
```
该程序会输出：
```
capacity is 3  
length is 2  
read value naveen  
new length is 1
```

# WaitGroup
**WaitGroup 用于等待一批 Go 协程执行结束。程序控制会一直阻塞，直到这些协程全部执行完毕**。假设我们有 3 个并发执行的 Go 协程（由 Go 主协程生成）。Go 主协程需要等待这 3 个协程执行结束后，才会终止。这就可以用 WaitGroup 来实现。

**WaitGroup 用于实现工作池**。
```
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```
当调用 WaitGroup 的 Add 并传递一个 int 时，WaitGroup 的计数器会加上 Add 的传参。要减少计数器，可以调用 WaitGroup 的 Done() 方法。Wait() 方法会阻塞调用它的 Go 协程，直到计数器变为 0 后才会停止阻塞。

**在第 21 行里，传递 wg 的地址是很重要的。如果没有传递 wg 的地址，那么每个 Go 协程将会得到一个 WaitGroup 值的拷贝，因而当它们执行结束时，main 函数并不会知道。**

该程序输出：
```
started Goroutine  2  
started Goroutine  0  
started Goroutine  1  
Goroutine 0 ended  
Goroutine 2 ended  
Goroutine 1 ended  
All go routines finished executing
```

# 工作池的实现
工作池就是一组等待任务分配的线程。一旦完成了所分配的任务，这些线程可继续等待任务的分配。

下面的例子使用缓冲信道来实现工作池，工作池的任务是计算所输入的数字的每一位的和。

工作池的核心功能如下：

- 创建一个 Go 协程池，监听一个等待作业分配的输入型缓冲信道。
- 将作业添加到该输入型缓冲信道中。
- 作业完成后，再将结果写入一个输出型缓冲信道。
- 从输出型缓冲信道读取并打印结果。

```
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

/*
工作和结果两个结构体
 */
type Job struct {
	id int
	randomno int
}

type Results struct {
	job Job
	sumofdigits int
}

/*
存放作业和结果的信道
 */
var jobs = make(chan Job, 10)
var results = make(chan Results, 10)

/*
用来计算结果的函数
 */
func digits(number int) int {
	sum := 0
	no := number
	for no != 0{
		digit := no % 10
		sum += digit
		no /= 10
	}
	time.Sleep(100 * time.Millisecond)
	return sum
}

/*
用来计算的协程，将结果写入Results信道中
 */
func worker(wg * sync.WaitGroup)  {
	for job := range jobs{
		output := Results{job, digits(job.randomno)}
		results <- output
	}
	wg.Done()
}

/*
创建工作池
 */
func createWorkerPool(noOfWorkers int)  {
	var wg sync.WaitGroup
	for i := 0; i < noOfWorkers; i++{
		wg.Add(1)
		go worker(&wg)
	}
	wg.Wait()
	close(results)
}

/*
创建工作
 */
func alloate(noOfJobs int)  {
	for i := 0; i < noOfJobs; i++{
		randomno := rand.Intn(999)
		job := Job{i, randomno}
		jobs <- job
	}
	close(jobs)
}

/*
打印结果
 */
func result(done chan bool)  {
	for r := range results{
		fmt.Printf("job id %d, input num %d, result %d\n", r.job.id, r.job.randomno, r.sumofdigits)
	}
	done <- true
}

func main()  {
	startTime := time.Now()
	noOfJobs := 100
	noOfWorkers := 10
	done := make(chan bool)

	go alloate(noOfJobs)
	go result(done)
	go createWorkerPool(noOfWorkers)
	<- done

	endTime := time.Now()
	diffTime :=endTime.Sub(startTime)
	fmt.Println("time costs is ", diffTime.Seconds(), " s")
}
```
程序会打印100行，对应着100个工作，结果如下：
```
job id 0, input num 878, result 23
job id 2, input num 407, result 11
job id 1, input num 636, result 15
...
time costs is  1.0187975  s
```

## 注意
**问题：**在allocate函数中如果for循环结束后，jobs里面的数据worker还来不及取走，这时执行到close，会不会导致works取数据失败？或者取不到足额的任务？

**答案：** golang 里的 close 只是用于通知信道的接收方，所有数据都已经发送完毕，信道没有真正关闭。
若用 for range 接收数据时，对于关闭了的信道，会接收完剩下的有效数据，并退出循环。如果没有 close 提示数据发送完毕的话，for range 会接收完剩下所有有效数据后发生阻塞。
所以接收方 worker 是可以把 jobs 剩下的数据取走的。后面垃圾收集器会自动回收掉该信道的内存。
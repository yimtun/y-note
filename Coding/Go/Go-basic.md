[toc]



#  1 读取json文件













# 2 channel





## 1 goroutine





### ex1 main goroutine不会等其他goroutine执行完再退出



```go
package main

import (
	"fmt"
)

func Count() {
	fmt.Println("goroutine 内部执行")
}

func main() {
	for i := 0; i < 10; i++ {
		go Count()
		fmt.Println("执行第", i+1, "个 goroutine")
	}
	fmt.Println("main goroutine 退出前")
}
```





```
执行第 1 个 goroutine
执行第 2 个 goroutine
执行第 3 个 goroutine
goroutine 内部执行
执行第 4 个 goroutine
goroutine 内部执行
goroutine 内部执行
执行第 5 个 goroutine
执行第 6 个 goroutine
执行第 7 个 goroutine
执行第 8 个 goroutine
执行第 9 个 goroutine
goroutine 内部执行
执行第 10 个 goroutine
main goroutine 退出前
goroutine 内部执行



启动10个goroutine 这里看执行完的goroutine只有5个，main goroutine不会主动等他其他goroutine执行完再退出。
```





### ex2 channel 会阻塞  直到数据被取出



```go
package main

import (
	"fmt"
)

func Count(ch chan int) {
	ch <- 1 // 阻塞 直到数据被取出
	fmt.Println("goroutine 内部执行")
}

func main() {
	defer fmt.Println("main goroutine 退出前")
	chs := make([]chan int, 10) // 元素类型是 chan int

	for i := 0; i < 10; i++ {
		chs[i] = make(chan int) // 为每个元素赋值
		go Count(chs[i])
		fmt.Println("执行第", i+1, "个 goroutine")
	}

}
```



```
fmt.Println("goroutine 内部执行") 不会被执行，因为没有从channel取出数据的动作
```





### ex3 用channel 协调goroutine间的通信



```go
package main

import (
	"fmt"
)

func Count(ch chan int) {
	fmt.Println("goroutine 内部执行")
	ch <- 1 // 阻塞 直到数据被取出

}

func main() {
	defer fmt.Println("main goroutine 退出前")
	chs := make([]chan int, 10) // 元素类型是 chan int

	for i := 0; i < 10; i++ {
		chs[i] = make(chan int) // 为每个元素赋值
		go Count(chs[i])
		fmt.Println("执行第", i+1, "个 goroutine")
	}

	for _, ch := range chs {
		<-ch //阻塞  等待读取channel 中的数据
	}

}
```



```bash
执行第 1 个 goroutine
执行第 2 个 goroutine
goroutine 内部执行
执行第 3 个 goroutine
goroutine 内部执行
执行第 4 个 goroutine
执行第 5 个 goroutine
执行第 6 个 goroutine
执行第 7 个 goroutine
执行第 8 个 goroutine
执行第 9 个 goroutine
执行第 10 个 goroutine
goroutine 内部执行
goroutine 内部执行
goroutine 内部执行
goroutine 内部执行
goroutine 内部执行
goroutine 内部执行
goroutine 内部执行
goroutine 内部执行
main goroutine 退出前
```







### ex4 无缓冲区的channel和死锁





```go
package main

func main() {
	ch := make(chan int) //无缓冲的channel

	ch <- 0 // deadlock
	//go func(chan int) { ch <- 0 }(ch) // no deadlock

}
// fatal error: all goroutines are asleep - deadlock!
/*
只有一个 main goroutine 
/*
```







```
package main

func main() {
	ch := make(chan int) //无缓冲的channel

	//go func(chInt chan int) { <-chInt }(ch)

	//go func(chInt chan int) { chInt <- 0 }(ch)

	ch <- 0

}
```





```go
//不会产生死锁
package main

import "time"

func main() {
	ch := make(chan int) //无缓冲的channel

	go func(chInt chan int) { <-chInt }(ch)

	//go func(chInt chan int) { chInt <- 0 }(ch)

	ch <- 0
	time.Sleep(5 * time.Second)
}
```









### 1 箭头符号书写方向是确定的：  <-





### 2 只能发送和只能接收的chanel



```
chan <- int      // 只可以用来发送 int 类型的数据     
< -chan int      // 只可以用来接收 int 类型的数据  ch <- 10
```
















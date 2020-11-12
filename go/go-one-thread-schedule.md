
# 又一个程序包含一样多的奇数和偶数，请写一个程序，交替打印奇数偶数。（ wesure.cn 面试题）
```go
package main
import (
	"fmt"
	"runtime"
)

func printOdd(list []int, done chan struct{}) {
	for _, i := range list {
		if i%2 != 0 {
			runtime.Gosched()
			fmt.Println(i)
		}
	}
	done <- struct {}{}
}

func printEven(list []int, done chan struct{}) {
	for _, i := range list {
		if i%2 == 0 {
			runtime.Gosched()
			fmt.Println(i)
		}
	}
	done <- struct {}{}
}

func PrintList(list []int) {
	runtime.GOMAXPROCS(1)
	done1 := make(chan struct{}, 0)
	done2 := make(chan struct{}, 0)
	go printEven(list, done1)
	go printOdd(list, done2)
	<-done1
	<-done2
}

func main() {
  list := []int{1,1,2,2,3,4}
	PrintList(list)
}
```
单核平均调度算法。

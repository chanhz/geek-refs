# Golang陷阱

1. string 类型是常量，底层虽然是 rune 数组，但通过下标访问只能读不能写。 只能用作右值表达式。


2. 未初始化的空切片为 nil ; 已初始化的切片即使其长度为0，但不等于nil. 对一个切片赋予 nil 其 len(s) 为 0， cap 为底层数组的大小

3. 声明未初始化的通道为 nil ，关闭 nil 的通道会 panic， 发送和接收都会阻塞

4. 关闭已关闭的通道会 panic

5. 往已关闭的通道发送数据会 panic

6. 用接收表达式同时复制给两个变量时，当通道中有数据，第二个值会为 true; 没有数据时，第二个值为 nil ,第一个值为 0 值。

7. fast-http

8. goroutine 的 context, 父子之间的变量是全局的

9. 限制 web 框架的并发请求数，在 gin 可以通过 gin-limit 实现：
某种程度可以限制WEB请求连接数,避免压垮服务器。遗憾的是，DDOS 仍然可以通过这种方式耗尽请求资源。
因此, 抗DDOS可以
1. 在更下一层做，基于特征值拦截
2. 程序层面拦截，统计对于请求量过大的IP，标记session, 插入验证码。
```go
package main

import (
  "github.com/aviddiviner/gin-limit"
  "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()
  r.Use(limit.MaxAllowed(20))
  // ...
  r.Run(":8080")
}
```
limit.go
```go
package limit

import (
	"github.com/gin-gonic/gin"
)

func MaxAllowed(n int) gin.HandlerFunc {
	sem := make(chan struct{}, n)
	acquire := func() { sem <- struct{}{} }
	release := func() { <-sem }
	return func(c *gin.Context) {
		acquire() // before request
		defer release() // after request
		c.Next()
		
	}
}

// Simple as that. Now you know :)
```
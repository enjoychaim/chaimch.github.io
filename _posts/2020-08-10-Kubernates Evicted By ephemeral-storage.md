---
title: Golang Context 使用与理解
tags: golang context
key: "golang:context:2020-08-10"
---

# 业务场景

用户发出请求后, 点击了取消. 此时该请求会对应几个 goroutine, 一个获取身份信息, 一个获取 db 数据, 一个校验 token. 一旦请求被取消, 该请求涉及到的其他 goroutine 也应该被取消

# 实现方式

## 原始的 Demo 处理过程

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func worker() {
	for {
		fmt.Println("worker")
		time.Sleep(time.Second)
	}
	// 如何接收外部命令实现退出
	wg.Done()
}
func main() {
	wg.Add(1)
	go worker()
	// 如何优雅的实现结束子goroutine
	wg.Wait()
	fmt.Println("main done")
}
```

## 全局变量方式处理

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

var wg sync.WaitGroup
var exit bool

// 全局变量方式存在的问题：
// 1. 使用全局变量在跨包调用时不容易统一
// 2. 如果worker中再启动goroutine，就不太好控制
func worker() {
    for {
        fmt.Println("worker")
        time.Sleep(time.Second)
        if exit {
            break
        }
    }
    wg.Done()
}

func main() {
    wg.Add(1)
    go worker()
    time.Sleep(time.Second * 3) // sleep3秒以免程序过快退出
    exit = true                 // 修改全局变量实现子goroutine的退出
    wg.Wait()
    fmt.Println("main done")
}
```

## 通道解决方式

```go
package main
import (
    "fmt"
    "sync"
    "time"
)

var wg sync.WaitGroup
// 管道方式存在的问题：
// 1. 使用全局变量在跨包调用时不容易实现规范和统一，需要维护一个共用的channel

func worker(exitChan chan struct{}) {
LOOP:
    for {
        fmt.Println("worker")
        time.Sleep(time.Second)
        select {
        case <-exitChan: // 等待接收上级通知
            break LOOP
        default:
        }
    }
    wg.Done()
}

func main() {
    var exitChan = make(chan struct{})
    wg.Add(1)
    go worker(exitChan)
    time.Sleep(time.Second * 3) // sleep3秒以免程序过快退出
    exitChan <- struct{}{}      // 给子goroutine发送退出信号
    close(exitChan)
    wg.Wait()
    fmt.Println("main done")
}
```

## 官方推荐版本

只需要将 context 对象传入, 即可以解决多个 goroutine 的上下文问题

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

var wg sync.WaitGroup

func worker(ctx context.Context) {
LOOP:
    for {
        fmt.Println("worker")
        time.Sleep(time.Second)
        select {
        case <-ctx.Done(): // 等待上级通知
            break LOOP
        default:
        }
    }
    wg.Done()
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    wg.Add(1)
    go worker(ctx)
    time.Sleep(time.Second * 3)
    cancel() // 通知子goroutine结束
    wg.Wait()
    fmt.Println("main done")
}
```

# context 是什么

## 官方定义

Golang 1.7 之后加入了一个标准库 context. 该标准库定义为 Context 类型, 专门用于简化对于处理单个请求的多个 goroutine 之间与请求域的数据, 取消信号, 截止时间等相关操作, 这些操作往往涉及到调用多个 API.

### 网络请求使用背景

如果有一个网络请求 request, 每个 request 都需要开启一个 goroutine 处理一些事情, 这些 goroutine 可能又会开启其他的 goroutine. 此时可以通过 context 来跟踪所有的 goroutine, 并且通过 context 来控制他们.

### 服务器程序使用背景

在 Go 服务器程序中, 每个请求都会有 goroutine 去处理. 然而, 处理程序往往还需要创建额外的 goroutine 去访问后端资源, 比如数据库, RPC 服务等. 由于 goroutine 都处理同一个请求, 往往还会访问一些共享的资源, 比如用户身份信息, 认证 token, 请求截止时间等. 如果请求超时或者被取消后, 所有的 goroutine 都应该马上退出, 并且释放相关资源. 此时就需要 context 来取消所有的 goroutine.

### context 接口定义

```go
// A Context carries a deadline, a cancellation signal, and other values across
// API boundaries.
//
// Context's methods may be called by multiple goroutines simultaneously.
type Context interface {
	// Deadline returns the time when work done on behalf of this context
	// should be canceled. Deadline returns ok==false when no deadline is
	// set. Successive calls to Deadline return the same results.
	Deadline() (deadline time.Time, ok bool)

	// Done returns a channel that's closed when work done on behalf of this
	// context should be canceled. Done may return nil if this context can
	// never be canceled. Successive calls to Done return the same value.
	// The close of the Done channel may happen asynchronously,
	// after the cancel function returns.
	//
	// WithCancel arranges for Done to be closed when cancel is called;
	// WithDeadline arranges for Done to be closed when the deadline
	// expires; WithTimeout arranges for Done to be closed when the timeout
	// elapses.
	//
	// Done is provided for use in select statements:
	//
	//  // Stream generates values with DoSomething and sends them to out
	//  // until DoSomething returns an error or ctx.Done is closed.
	//  func Stream(ctx context.Context, out chan<- Value) error {
	//  	for {
	//  		v, err := DoSomething(ctx)
	//  		if err != nil {
	//  			return err
	//  		}
	//  		select {
	//  		case <-ctx.Done():
	//  			return ctx.Err()
	//  		case out <- v:
	//  		}
	//  	}
	//  }
	//
	// See https://blog.golang.org/pipelines for more examples of how to use
	// a Done channel for cancellation.
	Done() <-chan struct{}

	// If Done is not yet closed, Err returns nil.
	// If Done is closed, Err returns a non-nil error explaining why:
	// Canceled if the context was canceled
	// or DeadlineExceeded if the context's deadline passed.
	// After Err returns a non-nil error, successive calls to Err return the same error.
	Err() error

	// Value returns the value associated with this context for key, or nil
	// if no value is associated with key. Successive calls to Value with
	// the same key returns the same result.
	//
	// Use context values only for request-scoped data that transits
	// processes and API boundaries, not for passing optional parameters to
	// functions.
	//
	// A key identifies a specific value in a Context. Functions that wish
	// to store values in Context typically allocate a key in a global
	// variable then use that key as the argument to context.WithValue and
	// Context.Value. A key can be any type that supports equality;
	// packages should define keys as an unexported type to avoid
	// collisions.
	//
	// Packages that define a Context key should provide type-safe accessors
	// for the values stored using that key:
	//
	// 	// Package user defines a User type that's stored in Contexts.
	// 	package user
	//
	// 	import "context"
	//
	// 	// User is the type of value stored in the Contexts.
	// 	type User struct {...}
	//
	// 	// key is an unexported type for keys defined in this package.
	// 	// This prevents collisions with keys defined in other packages.
	// 	type key int
	//
	// 	// userKey is the key for user.User values in Contexts. It is
	// 	// unexported; clients use user.NewContext and user.FromContext
	// 	// instead of using this key directly.
	// 	var userKey key
	//
	// 	// NewContext returns a new Context that carries value u.
	// 	func NewContext(ctx context.Context, u *User) context.Context {
	// 		return context.WithValue(ctx, userKey, u)
	// 	}
	//
	// 	// FromContext returns the User value stored in ctx, if any.
	// 	func FromContext(ctx context.Context) (*User, bool) {
	// 		u, ok := ctx.Value(userKey).(*User)
	// 		return u, ok
	// 	}
	Value(key interface{}) interface{}
}
```

1. Deadline方法是获取设置的截止时间的意思，第一个返回是截止时间，到了这个时间点，Context会自动发起取消请求；第二个返回值ok==false时表示没有设置截止时间，如果需要取消的话，需要调用取消函数进行取消。

2. Done方法返回一个只读的chan，类型为struct{}，我们在goroutine中，如果该方法返回的chan可以读取，则意味着parent context已经发起了取消请求，我们通过Done方法收到这个信号后，就应该做清理操作，然后退出goroutine，释放资源。之后，Err 方法会返回一个错误，告知为什么 Context 被取消。

3. Err方法返回取消的错误原因，因为什么Context被取消。

4. Value方法获取该Context上绑定的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值一般是线程安全的。

# context 实现方法

```go
var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)

// Background returns a non-nil, empty Context. It is never canceled, has no
// values, and has no deadline. It is typically used by the main function,
// initialization, and tests, and as the top-level Context for incoming
// requests.
func Background() Context {
	return background
}

// TODO returns a non-nil, empty Context. Code should use context.TODO when
// it's unclear which Context to use or it is not yet available (because the
// surrounding function has not yet been extended to accept a Context
// parameter).
func TODO() Context {
	return todo
}
```

## Background 方法

该方法主要用于 main 函数, 初始化以及测试代码中, 作为 context 的树结构的最顶层 context, 也被称为根 context, 不能被取消

## TODO 方法

如果不知道使用什么 context 时候, 可以使用这个.

## emptyCtx 本质

```go
// An emptyCtx is never canceled, has no values, and has no deadline. It is not
// struct{}, since vars of this type must have distinct addresses.
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}

func (e *emptyCtx) String() string {
	switch e {
	case background:
		return "context.Background"
	case todo:
		return "context.TODO"
	}
	return "unknown empty Context"
}
```

Background 以及 TODO 实际上都是返回了一个 emptyCtx 实例. 实现的 Deadline 方法中均未不可手动取消

## context 继承

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)

func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

func WithValue(parent Context, key, val interface{}) Context
```

1. WithCancel函数，传递一个父Context作为参数，返回子Context，以及一个取消函数用来取消Context。

2. WithDeadline函数，和WithCancel差不多，它会多传递一个截止时间参数，意味着到了这个时间点，会自动取消Context，当然我们也可以不等到这个时候，可以提前通过取消函数进行取消。

3. WithTimeout和WithDeadline基本上一样，这个表示是超时自动取消，是多少时间后自动取消Context的意思。

4. WithValue函数和取消Context无关，它是为了生成一个绑定了一个键值对数据的Context，这个绑定的数据可以通过Context.Value方法访问到，这是我们实际用经常要用到的技巧，一般我们想要通过上下文来传递数据时，可以通过这个方法，如我们需要tarce追踪系统调用栈的时候。
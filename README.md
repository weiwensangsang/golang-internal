# How Golang works



Go language has many amazing features and syntax, such as how to swap the values of two variables.

```go
package main

import "fmt"

func main() {
    a := 5
    b := 10
    fmt.Println("Before swapping, values of a and b are:", a, b)
    a, b = b, a
    fmt.Println("After swapping, values of a and b are:", a, b)
}
```

Output:

```
Before swapping, values of a and b are: 5 10
After swapping, values of a and b are: 10 5
```



This interesting syntax feature is used to test whether an engineer has enough understanding of the Go language. But why is this way of exchanging values valid? How is this syntax feature implemented in Go?

The goal of this project is to answer a series of questions like this from the perspective of the underlying implementation in Go, by explaining the source code, including syntax, data structures, and a series of concurrency issues.



#### Problems

1. [How to Swap the Values of Two Variables](problems/swap-the-values-of-two-variables.md)


#### What's more
Variable
    types of Assigning a variable
    How can I get the type of object?,
    Expressions. Diff b/w Curly brackets and Square brackets?
    how you can compare 2 objects in GO?, 
    Go 是传值还是传引用？
    Go 面试官问我如何实现面向对象？
    Go 结构体和结构体指针调用有什么区别吗？
    Go new 和 make 是什么，差异在哪？
    Go 结构体是否可以比较，为什么？
    Interfaces
    context 使用场景及注意事项
    context 是如何一层一层通知子 context
    What are Golang pointers?
    string literals?
    check the type of a variable at runtime in Go
    What are the uses of an empty struct?
    Shadowing in Go?
    variadic functions in Go
    byte and rune data types

GC
    How garbage collector works? ,
    go 的内存分配是怎么样的？
    Go 的逃逸行为是指？

Map
    how map works?
    sync.map
    为什么 Go map 和 slice 是非线程安全的？
    Go sync.map 和原生 map 谁的性能好，为什么？
    为什么 Go map 的负载因子是 6.5？
    map 为什么是不安全的？
    map 的 key 为什么得是可比较类型的？
    Slice 的扩容机制， range, slice
    slice 和 array 的区别
    How can we copy a slice and a map in Go?

Concurrency
    sync.pool
    mutex and a read-write lock in go?
    什么是协程，协程和线程的区别和联系？
    mutex 的正常模式、饥饿模式、自旋？
    sync.Once 原理
    waitgroup 
    concurrent data access? Channels or Maps?

GPM
    Why need P?

Goroutine
    单核 CPU，开两个 Goroutine，其中一个死循环，会怎么样？
    进程、线程都有 ID，为什么 Goroutine 没有 ID？
    Goroutine 数量控制在多少合适，会影响 GC 和调度？
    Goroutine 泄露的情况有哪些
    Go 在什么时候会抢占 P
    会诱发 Goroutine 挂起的 27 个原因
    goroutine 的协程有什么特点，和线程相比？
    channel 的内部实现是怎么样的？
    对已经关闭的 channel 进行读写，会怎么样？
    timer 
    
Develop
    Godep
    Gin
    

Overall
    详解 Go 程序的启动流程，你知道 g0，m0 是什么吗
    Go defer 万恶的闭包问题
    相比较于其他语言, Go 有什么优势或者特点？
    defer、panic、recover 
    What are Golang packages?
    Is Golang case sensitive or insensitive?
    return multiple values from a function in Go?
    Type Assertion in Go
    How is GoPATH different from GoROOT variables in Go?
    In Go, are there any good error handling practices?
    How to config working environments, parameters
    Go Version Update

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



### Problems

#### Variable

1. [How to Swap the Values of Two Variables](problems/swap-the-values-of-two-variables.md)
2. 

#### GC

1. ​	




#### What's more
Variable

1. ​    types of Assigning a variable
2. ​    How can I get the type of object?,
3. ​    Expressions. Diff b/w Curly brackets and Square brackets?
4. ​    how you can compare 2 objects in GO?, 
5. ​    Go 是传值还是传引用？
6. ​    Go 面试官问我如何实现面向对象？
7. ​    Go 结构体和结构体指针调用有什么区别吗？
8. ​    Go new 和 make 是什么，差异在哪？
9. ​    Go 结构体是否可以比较，为什么？
10. ​    Interfaces
11. ​    context 使用场景及注意事项
12. ​    context 是如何一层一层通知子 context
13. ​    What are Golang pointers?
14. ​    string literals?
15. ​    check the type of a variable at runtime in Go
16. ​    What are the uses of an empty struct?
17. ​    Shadowing in Go?
18. ​    variadic functions in Go
19. ​    byte and rune data types

GC

1. ​    How garbage collector works? ,
2. ​    go 的内存分配是怎么样的？
3. ​    Go 的逃逸行为是指？

Map

1. ​    how map works?
2. ​    sync.map
3. ​    为什么 Go map 和 slice 是非线程安全的？
4. ​    Go sync.map 和原生 map 谁的性能好，为什么？
5. ​    为什么 Go map 的负载因子是 6.5？
6. ​    map 为什么是不安全的？
7. ​    map 的 key 为什么得是可比较类型的？
8. ​    Slice 的扩容机制， range, slice
9. ​    slice 和 array 的区别
10. ​    How can we copy a slice and a map in Go?

Concurrency

1. ​    sync.pool
2. ​    mutex and a read-write lock in go?
3. ​    什么是协程，协程和线程的区别和联系？
4. ​    mutex 的正常模式、饥饿模式、自旋？
5. ​    sync.Once 原理
6. ​    waitgroup 
7. ​    concurrent data access? Channels or Maps?

GPM

1. ​    Why need P?

Goroutine

1. ​    单核 CPU，开两个 Goroutine，其中一个死循环，会怎么样？
2. ​    进程、线程都有 ID，为什么 Goroutine 没有 ID？
3. ​    Goroutine 数量控制在多少合适，会影响 GC 和调度？
4. ​    Goroutine 泄露的情况有哪些
5. ​    Go 在什么时候会抢占 P
6. ​    会诱发 Goroutine 挂起的 27 个原因
7. ​    goroutine 的协程有什么特点，和线程相比？
8. ​    channel 的内部实现是怎么样的？
9. ​    对已经关闭的 channel 进行读写，会怎么样？
10. ​    timer 

​    
Develop

1. ​    Godep
2. ​    Gin

​    

Overall

1. ​    详解 Go 程序的启动流程，你知道 g0，m0 是什么吗
2. ​    Go defer 万恶的闭包问题
3. ​    相比较于其他语言, Go 有什么优势或者特点？
4. ​    defer、panic、recover 
5. ​    What are Golang packages?
6. ​    Is Golang case sensitive or insensitive?
7. ​    return multiple values from a function in Go?
8. ​    Type Assertion in Go
9. ​    How is GoPATH different from GoROOT variables in Go?
10. ​    In Go, are there any good error handling practices?
11. ​    How to config working environments, parameters
12. ​    Go Version Update

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
2. [Use New or Make](problems/use-new-or-make.md)
3. string literals, byte and rune data types
4. [struct, Interfaces,  point and Object](problems/struct.md)
5. [How Context Works](problems/how-context-works.md)



#### GPM

1. [How Goroutine Works](problems/how-goroutine-works.md)
2. [What is GPM and Why need P?](problems/what-is-gpm-and-why-need-p.md)
3. Go 在什么时候会抢占 P
4. Channel channel 的内部实现是怎么样的？对已经关闭的 channel 进行读写，会怎么样？
5. timer



#### GC

1. ​    How garbage collector works? ,
2. ​    go 的内存分配是怎么样的？
3. ​    Go 的逃逸行为是指？



#### Map

1. [Map in Go](problems/map-in-go.md)
2. [New One: Slice](problems/new-one-slice.md)



#### Concurrency

1. [sync.map](problems/sync-map.md)
2.   sync.pool
3. ​    mutex and a read-write lock in go? mutex 的正常模式、饥饿模式、自旋？
4. ​    sync.Once
5. ​    waitgroup 



#### Develop

1. ​    Godep
2. ​    Gin

​    

#### Overall

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
13. ​    Expressions. Diff b/w Curly brackets and Square brackets?
14. Go 面试官问我如何实现面向对象？

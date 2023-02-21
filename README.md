# Golang Internal



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

#### Variable and Keywords

1. [How to Swap the Values of Two Variables](problems/swap-the-values-of-two-variables.md)
2. [Use New or Make](problems/use-new-or-make.md)
3. [String literals, Byte and Rune](problems/string.md)
4. [Struct, Interface, Pointer](problems/struct-interface-pointer.md)
5. [How Context Works](problems/how-context-works.md)
6. [How to use defer](problems/defer.md)
7. [Panic and Recover]()



#### GPM

1. [How Goroutine Works](problems/how-goroutine-works.md)
4. [How to Use Channel](problems/how-to-use-channel.md)



#### GC

1. [Go Memory Allocate](problems/go-memory-allocate.md)
2. [How Garbage Collector Works? ]()



#### Map

1. [Map in Go](problems/map-in-go.md)
2. [New One: Slice](problems/new-one-slice.md)



#### Concurrency

1. [sync.map](problems/sync-map.md)
2. [More sync: sync.pool, sync.once](problems/more-sync.md)
3. [Mutex, waitgroup and timer](problems/more-concurrency-tools.md)



#### Develop Tools

1. [Gin](problems/gin.md)
2. Godep

 

#### Code Practices

1. [Go Version Update](problems/go-versions.md)
2. [What are Golang packages?](problems/.md)
3. [Is Golang case sensitive or insensitive?](problems/golang-sensitive-problem.md)
4. [return multiple values from a function in Go?](problems/return-multiple-values.md)
5. [Type Assertion in Go](problems/type-aeeertion.md)
6. [How is GoPATH different from GoROOT variables in Go?](problems/gopath-and-goroot.md)
7. [In Go, are there any good error handling practices?](problems/error-handling.md)
8. [Expressions. Diff b/w Curly brackets and Square brackets?](problems/curly-brackets-and-square-brackets.md)

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
5. [How to Use Defer](problems/defer.md)
6. [Panic and Recover](problems/panic-and-recover.md)



#### GPM

1. [How Goroutine Works](problems/how-goroutine-works.md)
2. [How to Use Channel](problems/how-to-use-channel.md)
3. [How Context Works](problems/how-context-works.md)



#### GC

1. [Go Memory Allocate](problems/go-memory-allocate.md)
2. [How Garbage Collector Works? ](problems/go-garbage-collector.md)



#### Map

1. [Map in Go](problems/map-in-go.md)
2. [New One: Slice](problems/new-one-slice.md)



#### Concurrency

1. [Sync.map](problems/sync-map.md)
2. [More Sync: Sync.pool, Sync.once](problems/more-sync.md)
3. [Mutex](problems/mutex.md)
4. [Waitgroup](problems/waitgroup.md)
5. [Timer](problems/timer.md)



#### Develop Tools

1. [Gin](problems/gin.md)
2. Godep

 

#### Code Practices

1. Go Version Update
2. [What Are Golang Packages?](problems/package.md)
3. [Is Golang Case sensitive or Insensitive?](problems/golang-sensitive-problem.md)
4. [Return Multiple Values From A Function in Go?](problems/return-multiple-values.md)
5. [Type Assertion in Go](problems/type-assertion.md)
6. [How Is GoPATH Different From GoROOT Variables In Go?](problems/gopath-and-goroot.md)
7. [In Go, Are There Any Good Error Handling Practices?](problems/error-handling.md)
8. Expressions. Diff b/w Curly Brackets and Square Brackets?

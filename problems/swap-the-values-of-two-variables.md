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

1. swap the values of two variables.

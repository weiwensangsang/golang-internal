# Waitgroup



**WaitGroup** is a synchronization primitive in the Go programming language that allows you to wait for a group of **goroutines** to finish. To use **WaitGroup**, you first initialize its counter to a non-zero value, and then you increment the counter before each **goroutine** starts executing. Each **goroutine** should then call the Done method to signal that it has finished executing and decrement the counter. Finally, you call the Wait method to wait until the counter is zero.

```go
package main

import (
    "fmt"
    "sync"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()

    fmt.Printf("Worker %d starting\n", id)

    for i := 0; i < 100000000; i++ {
        _ = i * 2
    }
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup

    // 3 worker goroutine
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }


    wg.Wait()

    fmt.Println("All workers done")
}

```





Here's an example that shows how to use **WaitGroup** in Go. First, you create a **WaitGroup** object and start several worker **goroutines**. Each worker **goroutine** increments the **WaitGroup** counter before it starts executing and calls the Done method when it finishes. The main **goroutine** calls the Wait method to wait until all the worker **goroutines** have finished before proceeding.



The underlying structure of **WaitGroup** looks simple, but **WaitGroup.state1** actually represents three fields: counter, waiter, sema.

- counter : It can be understood as a counter, which calculates the value after wg.Add(N), wg.Done().
- waiter : The number of waiters currently waiting for the WaitGroup task to end. In fact, it is the number of calls to wg.Wait(), so usually this value is 1.
- sema: semaphore, used to wake up the Wait() function.

```go
type WaitGroup struct {
    noCopy noCopy
    state1 [3]uint32
}
```

The callers who use **WaitGroup** are generally concurrent operations. If the counter and waiter are not obtained at the same time, the obtained counter and waiter may not match, resulting in program deadlock or premature termination of the program. So put counter and waiter together.
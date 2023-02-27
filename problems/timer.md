# TImer



In Go, `time.Timer` is a commonly used timer that can be used to execute an operation after a specified time interval. Its source code implementation is based on Go's coroutines and the underlying operating system's scheduler, and can trigger an operation precisely after the specified time.

The following is the main source code implementation principle of `time.Timer`:

1. When a `time.Timer` object is created, the `runtime_startTimer` function is called to start a new timer. This function calculates the corresponding absolute time based on the timer's expiration time and adds the timer to the scheduler's timer queue.
2. When the timer expires, the scheduler adds the timer to the run queue and waits for the Go coroutine to execute. When the scheduler detects a timer in the run queue, it removes the timer from the queue and adds the coroutine corresponding to the timer to the run queue.
3. The coroutine corresponding to the timer is awakened by the scheduler and executes the corresponding operation. If the timer is a looping timer, the coroutine restarts the timer after executing the operation; otherwise, the timer is destroyed.

Overall, the implementation principle of `time.Timer` is based on the scheduler's timer queue and run queue, using the underlying operating system's timer mechanism to achieve efficient and accurate timer functionality.



Whether it is business development or infrastructure development, **timers** cannot be bypassed, which shows the importance of **timers**.

Regardless of whether we use NewTimer, timer.After, or timer.AfterFun to initialize a **timer**, this timer **will** eventually be added to a global **timer** heap, which is managed uniformly by the Go runtime.

Timer is the most complex and difficult data structure in Go



### Quad Heap

The global heap of the timer is a quadruple heap, and each P maintains a quadruple heap, which reduces the concurrency problem between **Goroutines** and improves the performance of the **timer**.

A quad heap is actually a quad tree. How does the Go timer maintain the quad heap?

When the Go runtime schedules **timers**, the **timers** whose trigger time is earlier should reduce the number of queries and be triggered as soon as possible. Therefore, the trigger time of the parent node of the quadtree must be less than that of the child node.
As the name implies, the quadtree has up to four child nodes. In order to take into account the speed of insertion, deletion, and rearrangement of the quadtree, the four sibling nodes are not required to be sorted by triggering sooner or later.

![quad-heap](..\pictures\quad-heap.gif)\


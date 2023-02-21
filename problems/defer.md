# How to use defer?



Let us start with code. What is the result for this code? defer is first in last out, This is very natural, the following statement will depend on the previous resource, so if the previous resource is released first, the subsequent statement cannot be executed.

```go
package main

import (
  "fmt"
)

func main() {
  defer_call()
}

func defer_call() {
  defer func() { fmt.Println("1") }()
  defer func() { fmt.Println("2") }()
  defer func() { fmt.Println("3") }()

  panic("ERROR")
}
```

Little weird but it should be,

```text
3
2
1
panic: ERROR
```



### defer

The execution logic of defer, return, and return value should be: return is executed first, and return is responsible for writing the result into the return value; then defer starts to perform some finishing work; finally, the function exits with the current return value.

In fact, to understand the defer closure, as long as you understand the following points, you will fully understand

1. 
   Closure is to manipulate variables in the parent function through pointers

2. 
   The process defined by defer is equivalent to generating a function body and then pushing it onto the stack, and then popping it out for execution when it returns

3. 
   defer is executed before return. It should be noted here that if the return variable is not declared, you need to declare a return value first and then assign it to return.



```go
type number int

func (n number) print()   { fmt.Println(n) }
func (n *number) pprint() { fmt.Println(*n) }

func main() {
    var n number

    defer n.print()
    defer n.pprint()
    defer func() { n.print() }()
    defer func() { n.pprint() }()

    n = 3
}
```

At first n = 0.



The first defer, push the code **0.print()** into defer stack (linkedList).

The second defer, push the code **(pointer of n).print()** into defer stack.

The third defer, push the code **func() { (pointer of n).print() }()** into defer stack.

The 4th defer, push the code **func() { (pointer of n).pprint() }()** into defer stack.



Then n=3.

Then return.

Then esecute 4th defer, print 3.

Then esecute third defer, print 3.

Then esecute second defer, print 3.

Then esecute first defer, print 0.



### What's more: How is defer implemented

It can be seen that the defer function in the Go language is a **last-in-first-out** mechanism. Why is it like this?

let's take a look at the implementation of defer(From src/runtime/runtime2.go):



```go
type _defer struct {
   started bool
   heap    bool
   // openDefer indicates that this _defer is for a frame with open-coded
   // defers. We have only one defer record for the entire frame (which may
   // currently have 0, 1, or more defers active).
   openDefer bool
   sp        uintptr // sp at time of defer
   pc        uintptr // pc at time of defer
   fn        func()  // can be nil for open-coded defers
   _panic    *_panic // panic that is running defer
   link      *_defer // next defer on G; can point to either heap or stack!

   // If openDefer is true, the fields below record values about the stack
   // frame and associated function that has the open-coded defer(s). sp
   // above will be the sp for the frame, and pc will be address of the
   // deferreturn call in the function.
   fd   unsafe.Pointer // funcdata for the function associated with the frame
   varp uintptr        // value of varp for the stack frame
   // framepc is the current pc associated with the stack frame. Together,
   // with sp above (which is the sp associated with the stack frame),
   // framepc/sp can be used as pc/sp pair to continue a stack trace via
   // gentraceback().
   framepc uintptr
}
```

![defer](../pictures/defer.png)

Intuitively, defer should be directly inserted by the compiler into the required function call. It seems to be a compile-time feature, and there should be no runtime performance problems. 

But the actual situation is that since defer is not linked to its dependent resources, **it is also allowed to appear in conditions and loop statements**, which makes the semantics of defer relatively complicated, and sometimes it is impossible to determine how many defer calls exist at compile time. This makes it possible for defer to be implemented on the heap or on the stack.



The main steps for the Go compiler to compile the Go code written by the programmer into machine-executable binary code are:
1) The compiler entry is in the gc.Main() method of the cmd/compile/internal/gc/main.go package;

2) gc.Main() performs lexical analysis, syntax analysis and type checking, and generates AST;

3) Main() calls the walk.Walk() method of the cmd/compile/internal/walk/walk.go package to traverse and rewrite the AST nodes in the code to generate the final abstract syntax tree AST. It should be noted that some keywords and built-in functions will be converted into runtime function calls in the walk.Walk() method, for example, the two built-in functions panic and recover will be converted into runtime.gopanic and runtime.gorecover Two real runtime functions, the keyword new will also be converted to call the runtime.newobject function, and **keywords such as Channel, map, make, new and select will be converted into corresponding runtime functions; and the main processing of the defer keyword logic is not here;**

4) Then, Main() generates the SSA intermediate code from the abstract syntax tree AST, and specifically calls ssagen.buildssa();

5) ssagen.buildssa() calls the state.stmtList() of the same file, state.stmtList() will call the state.stmt() method for each node passed in, and state.stmt() will The current AST node is converted into the corresponding intermediate code; **Note: the processing of the defer keyword is here in the state.stmt() method;**

From cmd/compile/internal/ssagen/ssa.go, we can see there are three ways of defer

```go
case ir.ODEFER:
   n := n.(*ir.GoDeferStmt)
   if base.Debug.Defer > 0 {
      var defertype string
      if s.hasOpenDefers {
         defertype = "open-coded"
      } else if n.Esc() == ir.EscNever {
         defertype = "stack-allocated"
      } else {
         defertype = "heap-allocated"
      }
      base.WarnfAt(n.Pos(), "%s defer", defertype)
   }
   if s.hasOpenDefers {
      s.openDeferRecord(n.Call.(*ir.CallExpr))// open defer
   } else {
      d := callDefer  // heap
      if n.Esc() == ir.EscNever {
         d = callDeferStack  //satch
      }
      s.callResult(n.Call.(*ir.CallExpr), d)
   }
```

From this logic we can know:

1) There are three ways to implement defer: 
   1) open coding,

   2)  stack allocation

   3) and heap allocation

2) If open coding is allowed, this method is preferred to implement defer, which was introduced by Go1.14 and has the best performance; we will analyze later that the conditions for realizing open coding are: 
   1) The compiler does not set parameters -N, that is, no inline optimization is disabled; 

   2) the number of defer in the function does not exceed the number of bits in a byte, that is, no more than 8; 

   3) the product of the number of defer functions and the number of parameters is less than 15; 

   4) The defer keyword cannot be executed in a loop. The defer call scenario that meets these conditions is simple, and most of the information can be determined at compile time;

3) If there is no memory escape to the heap, the stack allocation method is preferred to implement defer. This is an optimization method introduced by Go1.13, which reduces the extra overhead of memory allocation on the heap and improves the performance by about 30%;

4) If the previous two methods do not meet the conditions, then the method of heap allocation is used to implement defer by default;

The following first analyzes the implementation method of defer allocation on the heap first adopted by Go, then analyzes the allocation of defer on the stack, and finally the implementation method of open coding.



1. Go first introduced the implementation of defer allocation on the heap. The implementation principle is: the compiler first converts the defer delay statement into a runtime.deferproc call, and inserts the runtime.deferreturn function before the return of the defer function; Call the runtime.deferproc function to obtain a runtime._defer structure to record the relevant parameters of the defer function, and push the _defer structure onto the stack of the current Goroutine's defer delay chain head; when the defer function returns, call the runtime.deferreturn function from the Goroutine The linked list head takes out the runtime._defer structure and executes it sequentially, which ensures that multiple defers in the same function can be executed in LIFO order; the performance problem of this type of defer is that each defer statement must allocate memory on the heap, worst performance;

2. Go1.13 introduces the implementation of defer allocation on the stack. The implementation principle is: the compiler will directly create a runtime._defer structure on the stack without memory allocation, and pass the _defer structure pointer as a parameter. Enter the runtime.deferprocStack function, and at the same time, runtime._defer will be pushed into the delayed call list of Goroutine; when the function exits, deferreturn will be called, and the runtime._defer structure will be popped and executed; the cost of this type of defer is that it needs to run The parameters determined at time are assigned to the runtime._defer structure created on the stack, and the runtime performance is 30% higher than that allocated on the heap;

3. Go1.14 introduces an open coding method to implement defer, which realizes defer calls with almost zero cost. When -N is not set, inline is disabled, the product of the number of defer functions and the number of return values does not exceed 15, and the number of defer functions When there are no more than 8 defers and the defer does not appear in the loop statement, this type of open coding method will be used to implement the defer; the implementation principle is: the compiler will store the defer-related parameters according to the delay bits deferBits and the state.openDeferInfo structure, and return At the end of the statement, the defer call is executed according to whether the relevant bit of the delay bit is 1; the runtime cost of this type of defer is to determine whether the relevant defer function needs to be executed according to the condition and the delay bit, and the runtime performance of open coding is the best.



Defer

https://github.com/golang/go/issues/51839

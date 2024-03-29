# Panic and Recover

The **panic** keyword in the Go language is mainly used to actively throw exceptions, similar to the **throw** keyword in languages such as java. 

Panic can change the control flow of the program. After calling panic, it will immediately stop executing the remaining code of the current function, and recursively execute the caller's **defer** in the current Goroutine;

The **recover** keyword in the Go language is mainly used to **catch** exceptions and return the program to a normal state, similar to **try ... catch** in languages such as java. **recover** can abort the program crash caused by **panic**. It is a function that can only work in **defer**, and calling it in other scopes will not work;

**recover** can only be used in defer. **This** has been clearly written in the comments of the standard library. We can take a look:

```go
// The recover built-in function allows a program to manage behavior of a
// panicking goroutine. Executing a call to recover inside a deferred
// function (but not any function called by it) stops the panicking sequence
// by restoring normal execution and retrieves the error value passed to the
// call of panic. If recover is called outside the deferred function it will
// not stop a panicking sequence. In this case, or when the goroutine is not
// panicking, or if the argument supplied to panic was nil, recover returns
// nil. Thus the return value from recover reports whether the goroutine is
// panicking.
func recover() interface{}
```

To summarize the process of program crash and recovery:

1. The compiler will be responsible for doing the work of converting keywords;

   1. Convert **panic** and **recover** to **runtime.gopanic** and **runtime.gorecover** respectively;
   2. Convert **defer** to **runtime.deferproc** function;
   3. Call the **runtime.deferreturn** function at the end of the function that calls **defer**;

2. When the r**untime.gopanic** method is encountered during the running process, the **runtime._defer** structure will be sequentially taken out from the **Goroutine** linked list and executed;

3. **If runtime.gorecover is encountered when calling the delayed execution function, it will mark _panic.recovered as true and return the parameter of panic;**  (This is why you should add recover in defer)

   1. After this call ends, **runtime.gopanic** will take out the program counter **pc** and stack pointer **sp** from the **runtime._defer** structure and call the **runtime.recovery** function to restore the program;
   2. **runtime.recovery** will jump back to **runtime.deferproc** according to the incoming **pc** and **sp**;
   3. The code automatically generated by the compiler will find that the return value of **runtime.deferproc** is not 0, then it will jump back to **runtime.deferreturn** and return to the normal execution flow;

4. If **runtime.gorecover** is not encountered, it will traverse al**l runtime._defer** in turn, and finally call **runtime.fatalpanic** to abort the program, print the parameters of panic and return error code 2;

   

### Panic

Here we first introduce the principle of analyzing the **panic** function to terminate the program. The compiler will convert the keyword **panic** into **runtime.gopanic**, and the execution of this function includes the following steps:

1. Create a new **runtime._panic** and add it to the front of the **Goroutine's** **_panic** list;
2. In the loop, continuously obtain **runtime._defer** from the linked list in the **_defer** of the current **Goroutine** and call **runtime.reflectcall** to run the delayed call function;
3. Call **runtime.fatalpanic** to abort the entire program;



```go
func gopanic(e interface{}) {
	gp := getg()
	...
	var p _panic
	p.arg = e
	p.link = gp._panic
	gp._panic = (*_panic)(noescape(unsafe.Pointer(&p)))

	for {
		d := gp._defer
		if d == nil {
			break
		}

		d._panic = (*_panic)(noescape(unsafe.Pointer(&p)))

		reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))

		d._panic = nil
		d.fn = nil
		gp._defer = d.link

		freedefer(d)
		if p.recovered {
			...
		}
	}

	fatalpanic(gp._panic)
	*(*int)(nil) = 0
}
```



### Recover

```go
func gorecover(argp uintptr) interface{} {
	gp := getg()
	p := gp._panic
	if p != nil && !p.recovered && argp == uintptr(p.argp) {
		p.recovered = true
		return p.arg
	}
	return nil
}
```


# How to use channel

Don't communicate by sharing memory, but share memory by communicating. 

In many good programming languages, the way multiple threads transfer data is generally shared memory. 

In order to solve thread competition, we need to limit the number of threads that can read and write these variables at the same time. 

However, this is not the same as the design encouraged by the Go language.

Although we can also use shared memory with mutex for communication in Go language, Go language provides a different concurrency model, namely Communicating sequential processes (CSP). Goroutine and Channel respectively correspond to the entity in CSP and the medium for transmitting information, and Goroutine will transmit data through Channel.



### Use it

Here is the most simple way to use channel.

```go
package main
import "fmt"

func main() {
  c := make(chan int)
  go func() {
    // send 1 to channel
    c <- 1
  }()
  // get data from channel
  x := <- c
  close(c)
  fmt.Println(x)
}

func makechan(t *chantype, size int) *hchan {} //make(chan int)对应
func chansend1(c *hchan, elem unsafe.Pointer) {} //ch <- 1
func chanrecv1(c *hchan, elem unsafe.Pointer) {} //x := <- c
func closechan(c *hchan) {} //close(c)
```



The next code is to create an unbuffered **channel**. Once a **goroutine** sends data to the **channel**, the current **goroutine** will be blocked until other **goroutines** consume the data in the **channel** before continuing to run.

```go
 ch := make(chan int)
```

There is another buffered **channel**, which is created like this:

```text
ch := make(chan int, 2)
```



The second parameter indicates the capacity of the **channel** to buffer data. As long as the total number of elements in the current **channel** is not greater than the bufferable capacity, the current **goroutine** will not be blocked.



### **select**

When we want to communicate with multiple **goroutines**, we will use the **select** to manage the communication data of multiple **channel:



```go
 ch1 := make(chan struct{})
 ch2 := make(chan struct{})

 // ch1, ch2 发送数据
 go sendCh1(ch1)
 go sendCh1(ch2)

 // channel 数据接受处理
 for {
  select {
  case <-ch1:
   doSomething1()
  case <-ch2:
   doSomething2()
  }
 }
```



### runtime.hchan

Channel in Go language is represented by runtime.hchan structure at runtime. When we create a new Channel in Go language, what we actually create is the following structure:

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters, read more in the type waitq
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}

type waitq struct {
	first *sudog // read more in the type sudog
	last  *sudog
}

// sudog represents a g in a wait list, such as for sending/receiving
// on a channel.
//
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many. A g can be on many wait lists, so there may be
// many sudogs for one g; and many gs may be waiting on the same
// synchronization object, so there may be many sudogs for one object.
//
// sudogs are allocated from a special pool. Use acquireSudog and
// releaseSudog to allocate and free them.
type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool

	// success indicates whether communication over channel c
	// succeeded. It is true if the goroutine was awoken because a
	// value was delivered over channel c, and false if awoken
	// because c was closed.
	success bool

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```


As we said before, All Channel creation in Go language will use the make keyword. The compiler will convert the make(chan int, 10) expression to a node of type OMAKE, and converts a node of type OMAKE to type OMAKECHAN during the type checking phase:

```go
// tcMake typechecks an OMAKE node.
func tcMake(n *ir.CallExpr) ir.Node {
  ...
	switch t.Kind() {
	default:
		base.Errorf("cannot make type %v", t)
		n.SetType(nil)
		return n

	...
	case types.TCHAN:
		l = nil
		if i < len(args) {
			l = args[i]
			i++
			l = Expr(l)
			l = DefaultLit(l, types.Types[types.TINT])
			if l.Type() == nil {
				n.SetType(nil)
				return n
			}
			if !checkmake(t, "buffer", &l) {
				n.SetType(nil)
				return n
			}
		} else {
			l = ir.NewInt(0)
		}
		nn = ir.NewMakeExpr(n.Pos(), ir.OMAKECHAN, l, nil) // change to op OMAKECHAN
	}

	...
}

// walkMakeChan walks an OMAKECHAN node.
func walkMakeChan(n *ir.MakeExpr, init *ir.Nodes) ir.Node {
	// When size fits into int, use makechan instead of
	// makechan64, which is faster and shorter on 32 bit platforms.
	size := n.Len
	fnname := "makechan64"
	argtype := types.Types[types.TINT64]

	// Type checking guarantees that TIDEAL size is positive and fits in an int.
	// The case of size overflow when converting TUINT or TUINTPTR to TINT
	// will be handled by the negative range checks in makechan during runtime.
	if size.Type().IsKind(types.TIDEAL) || size.Type().Size() <= types.Types[types.TUINT].Size() {
		fnname = "makechan"     // use makechan function
		argtype = types.Types[types.TINT]
	}

	return mkcall1(chanfn(fnname, 1, n.Type()), n.Type(), init, reflectdata.MakeChanRType(base.Pos, n), typecheck.Conv(size, argtype))
}
```

Finally, we can find the code for make a channel, we use hchan.

```go
func makechan(t *chantype, size int) *hchan {
   elem := t.elem

   ...

   // Hchan does not contain pointers interesting for GC when elements stored in buf do not contain pointers.
   // buf points into the same allocation, elemtype is persistent.
   // SudoG's are referenced from their owning thread so they can't be collected.
   // TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
   var c *hchan
   switch {
   case mem == 0:
      // Queue or element size is zero.
      c = (*hchan)(mallocgc(hchanSize, nil, true))
      // Race detector uses this location for synchronization.
      c.buf = c.raceaddr()
   case elem.ptrdata == 0:
      // Elements do not contain pointers.
      // Allocate hchan and buf in one call.
      c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
      c.buf = add(unsafe.Pointer(c), hchanSize)
   default:
      // Elements contain pointers.
      c = new(hchan)
      c.buf = mallocgc(mem, elem, true)
   }

   c.elemsize = uint16(elem.size)
   c.elemtype = elem
   c.dataqsiz = uint(size)
   ...
}
```



### How channel work?

#### no-buffer channel 

##### write before reading

There are 2 **goroutines** using **channel** communication, in the order of writing first and then reading, the specific process is as follows:



channel

|     buf    |

|     lock   |

|     ...       |

|   sendq |   <===  G1 

|   recvq  |



It can be seen that because the **channel** is unbuffered, G1 is temporarily suspended in the sendq queue, and then G1 calls gopark to sleep.

Then, another **goroutine** comes to the **channel** to read data:



channel

|     buf    |

|     lock   |

|     ...       |                          goready

|   sendq |   ------  G1    <=======>  G2

|   recvq  |                          data



At this time, G2 finds that there are **goroutines** in the sendq waiting queue, so it directly copies the data from G1, and sets the goready function for G1, so that when the next scheduling occurs, G1 can continue to run and will be removed from the waiting queue

##### read before writing

G1 want to read but get nothing, so this **goroutine** should sleep in the recvq



channel

|     buf    |

|     lock   |

|     ...       |

|   sendq |  

|   recvq  |   <===  G1 



When G2 is writing data, it finds that there are **goroutines** in the recvq queue, so it directly sends the data to G1. At the same time, set the G1 goready function and wait for the next scheduled operation.



channel

|     buf    |

|     lock   |

|     ...       |                          

|   sendq |                    goready and ata

|   recvq  |    ------  G1    <=======  G2



#### channel with buffer

##### write before reading



channel            data

|     buf    |  <=======  G1 (if G1 can write data into buf,)

|     lock   |

|     ...       |                          

|   sendq |   <=======  G1 (if G1 can not write data into buf, it means buf is full of data,,)

|   recvq  |    



When G2 wants to read data, it will read from the buffer data area first, and after reading, it will check the sendq queue. If the **goroutine** has a waiting queue, it will add the data on it to the buffer data area, and Also set the goready function on it.



channel              1. read data

|     buf    |  =========>  G2

|     lock   |

|     ...       |                2. scan sendq and push G1 to buf

|   sendq |  ------  G1 ========> to buf

|   recvq  |  



#### channel read first, then write

This situation is the same process as unbuffered first read and then write, and will not be repeated here.

# Mutex



If you want to know the **mutex**, you should konw **CAS** first.

### CAS

**CAS**  means compare and swap.
Take the atomic.CompareAndSwapInt32 method in the go standard library as an example

```go
// CompareAndSwapInt32 executes the compare-and-swap operation for an int32 value.
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
```

It receives three input parameters, a pointer addr, an old value old, and a new value new
What this method does is divided into three steps:

1. First take the value pointed to by addr
2. Then compare with old, if not equal, return false, the operation failed.
3. Finally, change the value pointed to by addr to new

The key point is that the atomicity of these three steps is guaranteed at the machine level, that is, the execution process will not be interrupted by other coroutines. Ordinary code may be interrupted suddenly during execution, and the cpu turns to execute other coroutines.
In addition to **CAS**, go's atomic package also provides other atomic methods, which can solve data competition problems in some concurrent scenarios.

Let us build a mutex v0!



### Mutex V0



```go
package v0

import "sync/atomic"

type Mutex struct {
    key int32
}

func (m *Mutex) Lock(){
    for {
        if atomic.CompareAndSwapInt32(&m.key,0,1) {
            return
        }
    }
}

func (m *Mutex) Unlock() {
    atomic.CompareAndSwapInt32(&m.key,1,0) 
}
```

 

It is a 100% **mutex**. **Mutex** has a key to record the lock status. How to lock?

Suppose there are two goroutines g1 and g2,  competing for the same lock by the infinite-for loop with CAS operations. if g1 is a little fast,

 1. g1 executes first and gets the lock. When operating critical resources, g2 comes to get the lock
 2. At this time, the Lock method of g2 will try to execute **CAS**, but in the second step of **CAS**, it will fail because g1 has changed the key to 1. g2 will continue to try **CAS**, so g2 blocks here.
 3. g1 has completed the operation of critical resources, called the Unlock method, and changed the key value back to 0 through **CAS**.
 4. Only then the Lock method of g2 can succeed.

- 

But, g2 has nothing to do but stay in th CPU, it is awful.



### Mutex V1

```go
type Mutex struct {
    key int32 //0 means the lock is not held, 1 means the lock is already held, n means the lock is held and there are n-1 lock waiters
    sema int32 /*Represents a semaphore
				It is used for sleep and wake-up. In the sample code, the following pseudo-code is used to express the function, 				but there are corresponding methods in the go runtime.
				semacquire(sema)
				Hibernate the current goroutine
				semrelease(sema)
				Wake up to take the first coroutine from the coroutine queue that sleeps using the sema semaphore
				*/
}

func xadd(val *int32, delta int32) (new int32) { // +-1
    for {
        v := *val
        // val address
        // v old value
        // v+delta new value
        if atomic.CompareAndSwapInt32(val,v,v+delta) {
            return v + delta
        }
    }
    panic("unreached")
}

func (m *Mutex) Lock() {
    // CAS + 1
    if xadd(&m.key,1) == 1 {
        return
    }
    // stop goroutine
    semacquire(&m.sema)
    return
}

func (m *Mutex) Unlock() {

    if xadd(&m.key,-1) == 0 {
        return
    }
    // wake up goroutine
    semrelease(&m.sema)
    return
}
```

The Lock method and the Unlock method do not replace the key between 0 and 1, but use ±1 operation, so that not only the lock holding status can be recorded, but also the number of queued goroutines can be recorded

Still have some problems,

- If we have lots of **gorroutines** try to get mutes, only one goroutine can get it, others will sleep.
- Sleep and wake up, means goroutine change context, it is just fine.
- Can we do this? if one goroutine Unlock, find the active goroutine and just give the lock? More fast. 



### Mutex V2



```go
   type Mutex struct {
        state int32
        sema  uint32 // same as V1
    }


    const (
        mutexLocked = 1 << iota // mutex is locked // 1
        mutexWoken //2
        mutexWaiterShift = iota //2
    )


```



The 32bit state is a composite field, which can be divided into three parts: mutexWaiters | mutexWoken | mutexLocked

1. mutexWaiters indicates the number of lock waiters
2. mutexWoken indicates whether the lock has a wake-up goroutine
3. mutexLocked indicates whether the lock is held

Its calculation method is bit operation, which can be understood as combining three fields into one field, reducing storage space.



### Lock



```go
func (m *Mutex) Lock() {
        // Fast path: lucky case，just get lock
        if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
            return
        }

        awoke := false
        for {
            old := m.state
            new := old | mutexLocked // add lock
            if old&mutexLocked != 0 {
                new = old + 1<<mutexWaiterShift //add 1 in lock waiter
            }
            if awoke {
                // goroutine is awake

                new &^= mutexWoken
            }
            // 这个CAS的竞争者是awake goroutine和 new goroutine
            //The optimization of the v2 version is to make active goroutines have a higher priority than dormant goroutines during the lock grabbing process. So in this case, I'm sorry, I'm going to jump in the queue, and the current goroutine directly gets the lock.
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                if old&mutexLocked == 0 { 
                    break
                }
                runtime.Semacquire(&m.sema) 
                awoke = true
            }
        }
    }
```



Here is the problem,
If i am a goroutine, and i can not get in the lock. Should i just sleep? Probably not! **Can i just wait some times and make me keep active status**? Because the logic want to keep "active goroutine" get the lock first, right? Go to sleep immediately, not a good idea.



### Mutex V3

Everything is same, just add spin.

```go
func (m *Mutex) Lock() {
        // Fast path: lucky
        if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
            return
        }

        awoke := false
        iter := 0
        for { 
            old := m.state //
            new := old | mutexLocked 
            if old&mutexLocked != 0 { 
                if runtime_canSpin(iter) { // We can spin! if runtime_canSpin(iter) determines whether it can spin, depending on the hardware conditions and the number of spins allowed (iter).
                    if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
                        atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
                        awoke = true
                    }              
                    runtime_doSpin()
                    iter++
                    continue /
                }
                new = old + 1<<mutexWaiterShift
            }
            if awoke { 
                if new&mutexWoken == 0 {
                    panic("sync: inconsistent mutex state")
                }
                new &^= mutexWoken 
            }
            if atomic.CompareAndSwapInt32(&m.state, old, new) {
                if old&mutexLocked == 0 {
                    break
                }
                runtime_Semacquire(&m.sema) 
                awoke = true 
                iter = 0
            }
        }
    }
```

The new **goroutine** "cheats" into giving itself priority. If this phenomenon continues to occur, the dormant **goroutine** will never come out,



### Mutex V4

Compared with the previous implementation, the most important change of the current **Mutex** is the addition of **starvation** **mode**.
Once the waiter waits for more than 1 millisecond, the lock may enter **starvation** **mode**. The awakened waiter may not be able to grab the lock. At this time, it will be dormant again and inserted into the front of the queue
If the waiter fails to acquire the lock for more than the threshold of 1 millisecond, it enters **starvation mode**.
In **starvation mod**e, after the current owner of the **Mutex** finishes using the lock, it is directly handed over to the waiter at the front of the queue.

```go
const (
        mutexLocked = 1 << iota // mutex is locked
        mutexWoken
        mutexStarving // in state
        mutexWaiterShift = iota
    
        starvationThresholdNs = 1e6
    )

    func (m *Mutex) Lock() {
        // Fast path: lucky
        if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
            return
        }
        // Slow path：try spin or strave
        m.lockSlow()
    }
    
    func (m *Mutex) lockSlow() {
        var waitStartTime int64
        starving := false 
        awoke := false 
        iter := 0 // spin number
        old := m.state 
        for {

            if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
                if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
                    atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
                    awoke = true
                }
                runtime_doSpin()
                iter++
                old = m.state 
                continue
            }
            new := old
            if old&mutexStarving == 0 {
                new |= mutexLocked 
            }
            if old&(mutexLocked|mutexStarving) != 0 {
                new += 1 << mutexWaiterShift 
            }
            if starving && old&mutexLocked != 0 {
                new |= mutexStarving // set mutex Starving
            }
            if awoke {
                if new&mutexWoken == 0 {
                    throw("sync: inconsistent mutex state")
                }
                new &^= mutexWoken 
            }

            if atomic.CompareAndSwapInt32(&m.state, old, new) {

                if old&(mutexLocked|mutexStarving) == 0 {
                    break // locked the mutex with CAS
                }


                // remove to the head of queue
                queueLifo := waitStartTime != 0
                if waitStartTime == 0 {
                    waitStartTime = runtime_nanotime()
                }

                runtime_SemacquireMutex(&m.sema, queueLifo, 1)

                starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
                old = m.state
                // in straving
                if old&mutexStarving != 0 {
                    if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
                        throw("sync: inconsistent mutex state")
                    }

                    delta := int32(mutexLocked - 1<<mutexWaiterShift)
                    if !starving || old>>mutexWaiterShift == 1 {
                        delta -= mutexStarving 
                    }
                    atomic.AddInt32(&m.state, delta)
                    break
                }
                awoke = true
                iter = 0
            } else {
                old = m.state
            }
        }
    }
    
```
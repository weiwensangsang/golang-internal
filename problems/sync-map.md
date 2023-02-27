

# Sync.map



Go's built-in **map** does not support concurrent write operations. The reason is that **map** write operations are not concurrently safe. When you try to operate the same **map** with multiple **Goroutines**, an error will be reported: 

```
fatal error: concurrent map writes.
```

Therefore, the official introduced **sync.Map** to meet the application in concurrent programming.
The implementation principle of **sync.Map** can be summarized as:

1. Read and write are separated by two fields: **read** and **dirty**. The **read** data is stored in the read-only field **read**, and the latest written data is stored in the **dirty** field.
2. When reading, it will first query **read**, and then query **dirty** if it does not exist, and only write **dirty** when writing
   Reading **read** does not require locking, but reading or writing **dirty** requires locking
3. In addition, there is a **misses** field to count the number of **read** penetrations (penetration refers to the need to read **dirty**), and if the number exceeds a certain number, the **dirty** data will be synchronized to the **read**
4. For deleting **data**, directly mark it to delay deletion

The data structure is this,

![sync-map](..\pictures\sync-map.png)



```go
type Map struct {
    // mutex to protect dirty
    mu Mutex
    // readOnly
    read atomic.Value
    // save new data
    dirty map[interface{}]*entry
    // counter for read dirty
    misses int
}

type readOnly struct {
    // inner map
    m  map[interface{}]*entry
    // if should read dirty
    amended bool
}

type entry struct {
    p unsafe.Pointer  //  *interface{}
}
```



### Load

A get function.



```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
    // try to get read first readOnly
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]

    // try to get key from dirty
    if !ok && read.amended {
        m.mu.Lock()
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]

        // try get data from dirty
        if !ok && read.amended {
            e, ok = m.dirty[key]
            // call miss 
            m.missLocked()
        }
        m.mu.Unlock()
    }

    if !ok {
        return nil, false
    }

    return e.load()
}

func (m *Map) missLocked() {
    m.misses++
    if m.misses < len(m.dirty) {
        return
    }
    // if too much miss save dirty to read
    m.read.Store(readOnly{m: m.dirty})
    m.dirty = nil
    m.misses = 0
}
```



### Store

```go
func (m *Map) Store(key, value interface{}) {
    read, _ := m.read.Load().(readOnly)
    // if in read save to entry
    if e, ok := read.m[key]; ok && e.tryStore(&value) {
        return
    }

    m.mu.Lock()
    read, _ = m.read.Load().(readOnly)

    if e, ok := read.m[key]; ok {

        if e.unexpungeLocked() {

            m.dirty[key] = e
        }

        e.storeLocked(&value)
    } else if e, ok := m.dirty[key]; ok {

        e.storeLocked(&value)
    } else {

        if !read.amended {

            m.dirtyLocked()

            m.read.Store(readOnly{m: read.m, amended: true})
        }

        m.dirty[key] = newEntry(value)
    }
    m.mu.Unlock()
}

func (e *entry) tryStore(i *interface{}) bool {
    for {
        p := atomic.LoadPointer(&e.p)
        if p == expunged {
            return false
        }
        if atomic.CompareAndSwapPointer(&e.p, p, unsafe.Pointer(i)) {
            return true
        }
    }
}

func (e *entry) unexpungeLocked() (wasExpunged bool) {
    return atomic.CompareAndSwapPointer(&e.p, expunged, nil)
}

func (e *entry) storeLocked(i *interface{}) {
    atomic.StorePointer(&e.p, unsafe.Pointer(i))
}

func (m *Map) dirtyLocked() {
    if m.dirty != nil {
        return
    }

    read, _ := m.read.Load().(readOnly)
    m.dirty = make(map[interface{}]*entry, len(read.m))
    for k, e := range read.m {

        if !e.tryExpungeLocked() {
            m.dirty[k] = e
        }
    }
}

func (e *entry) tryExpungeLocked() (isExpunged bool) {
    p := atomic.LoadPointer(&e.p)
    for p == nil {

        if atomic.CompareAndSwapPointer(&e.p, nil, expunged) {
            return true
        }
        p = atomic.LoadPointer(&e.p)
    }
    return p == expunged
}
```



It can be seen that through this design of read-write separation, the write security in the concurrent situation is solved, and the read speed can be close to the built-in map in most cases, which is very suitable for the situation of more reads and less writes.

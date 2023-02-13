# Use New or Make?

New is this from source code.

```
// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```



Make is this from source code.

```
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
// Slice: The size specifies the length. The capacity of the slice is
// equal to its length. A second integer argument may be provided to
// specify a different capacity; it must be no smaller than the
// length, so make([]int, 0, 10) allocates a slice of length 0 and
// capacity 10.
// Map: An empty map is allocated with enough space to hold the
// specified number of elements. The size may be omitted, in which case
// a small starting size is allocated.
// Channel: The channel's buffer is initialized with the specified
// buffer capacity. If zero, or the size is omitted, the channel is
// unbuffered.
func make(t Type, size ...IntegerType) Type
```



In short, there are some distinct differences between them:

1. Different purposes: `make` is used to create instances of built-in types (such as slices, maps, and channels), while `new` is used to create instances of any type.
2. Different parameters: `make` takes a type and an initial size as parameters, while `new` only takes a type as a parameter.
3. Different return values: `make` returns an initialized object, while `new` returns a pointer to an initialized object.
4. Different initialization: The object returned by `make` has already been initialized to its zero value, while the object returned by `new` still needs to be initialized.



#### Why?

Implementation principles:

- `make` function: It is a built-in function, a special memory allocation function used to create instances of built-in types. It implements the underlying memory allocation function at the bottom and also includes specific initialization steps.
- `new` function: It is also a built-in function used to allocate an unnamed piece of memory and return a pointer to that memory. On the underlying implementation, it calls the memory allocation function to allocate memory for the object, but does not perform any initialization.



From src/cmd/compile/internal/ir/fmt.go, we can see this.

```
ONEW:              "new",
```

ONEW is the operation ENUM for new. 

# Struct, Interface, Pointer

We should talk about this 4 thing in this doc since go is an object-oriented language.



### Pointer

Pointers are an addressing mechanism supported in Go. Pointers have two operators called addressing (&) and taking the value of the variable pointed to by the pointer (*).

```go
package main
import (
     "fmt"
)
func main() {
     a := 42
     //prints out the value of a
     fmt.Println("value of a: ", a)

     //prints out the memory address that pointing to "a" variable
     fmt.Println("memory address of a: ", &a)

}
```

output:

```
value of a: 42
memory address of a: 0xc000014080
```



Using pointers, a variable can be addressed to another variable. For example, the b variable can address another variable (in this code, the variable is called a).

```go
package main
import (
     "fmt"
)
func main() {
     a := 42
     //prints out the value of a
     fmt.Println("value of a: ", a)

     //prints out the memory address that pointing to "a" variable
     fmt.Println("memory address of a: ", &a)
    
     b := &a
     //prints out the memory address of "b" variable
     fmt.Println("value of b:", b)
    
     //to prints out the value of b, use * (dereference) operator
     fmt.Println("value of b using * operator: ", *b)

}
```

output

```
value of a: 42
memory address of a: 0xc000014080
value of b: 0xc000014080
value of b using * operator: 42
```



### Structure

Classes are an important part of OOP. In Go, there is a similar concept called struct.

```go
// struct declaration
type structName struct {
     //field name type
     member int
     member2 string
     member3 [] string
}
```

Example usage of struct. Illustrates a person with the ability to speak.

```go
//declare a struct called person
type person struct {
     name string
     age int
}

//declare a method say() with type person as receiver
func (p person) say() {
     fmt.Println("Hello, my name is: ", p.name)
}
To use the defined struct, assign it to a variable, as follows:

package main

import "fmt"

//declare a struct called person
type person struct {
     name string
     age int
}

//declare a method say() with type person as receiver
func (p person) say() {
     fmt.Println("Hello, my name is: ", p.name)
}

func main() {
     //Instantiate person and assign value
     p1 := person{name: "Enki Gilbert", age: 42}
     //call a method say()
     p1. say()
}

// output
// Hello, my name is: Enki Gilbert
Go also provides anonymous struct.

s1 := struct{
     //declare some fields
     field1 int
     field2[]string
}{
     //instantiate directly
     field1: 12,
     field2: []string{"hi","mate"},
}
```



### Interface

Interface is another powerful concept in programming. Interface is similar to struct, but only contains some abstract methods. In Go, Interface defines an abstraction for common behavior.

```go
package main
import (
     "fmt"
)

//declare a rectangle struct
type rectangle struct {
     length int
     width int
}

//declare an interface with area() as a member
type shape interface {
     area() int
}

//declare a method area()
//the rectangle struct implements area() method in shape interface
func (r rectangle) area() int {
     return r.length * r.width
}

//declare a method with type shape as a parameter
func info(s shape) {
     fmt. Println("the area: ", s. area())
}

func main() {
     r1 := rectangle{12, 12}
     //r1 is a rectangle type. rectangle implements all methods in shape interface.
     info(r1)
}

// output
// the area: 144
```

Following the example, we declare a struct for rectangles and an interface for shapes. Rectangle implements area() in the shape interface. info() takes a shape type as an argument. The struct that implements all the methods in the shape interface can be used as the parameter of info().

If there is another struct called square. The info() method is also available, since squares also implement all the methods in the shape interface.

```go
package main

import (
     "fmt"
)

//declare a rectangle struct
type rectangle struct {
     length int
     width int
}

//declare a square struct
type square struct {
     side int
}

//declare an interface with area() as a member
type shape interface {
     area() int
}

//declare a method area()
//the rectangle struct implements area() method in shape interface
func (r rectangle) area() int {
     return r.length * r.width
}

//the square struct implements area() method in shape interface
func (s square) area() int {
     return s.side * s.side
}

//declare a method with type shape as a parameter
/**
anything that implements all methods in shape interface is considered as a shape in general.
for this case the rectangle and square is a shape because implements all methods in shape interface
**/
func info(s shape) {
     fmt. Println("the area: ", s. area())
}

func main() {
     r1 := rectangle{12, 12}
     info(r1)

     s1 := square{25}
     info(s1)

}

// output
// the area: 144
// the area: 625
```

It is also worth noting that interfaces can also be combined.

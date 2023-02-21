# String literals, Byte and Rune

String is immutable. 

Like the string in the C++ language, it has its own memory space, and modifying the string is supported. But in the implementation of Go, string does not contain memory space, only a memory pointer. The advantage of this is that string becomes very lightweight and can be passed easily without worrying about memory copying.

```go
type stringStruct struct {
   str unsafe.Pointer // only a pointer
   len int
}
```



### [] byte to string

A byte slice can be easily converted to a string, as follows:

```go
func GetStringBySlice(s []byte) string {
     return string(s)
}
```



### rune

The rune type is a special numeric type of the Go language. In the builtin/builtin.go file, its definition: 

```
type rune = int32;
```

The official explanation for it is: rune is an alias of type int32, which is equivalent to it in all respects, and is used to distinguish character values from integer values. Defined with single quotes, returns a Unicode code point encoded in UTF-8. The Go language handles Chinese through runes and supports international multilingualism.

Strings are composed of characters, the bottom layer of characters is composed of bytes, and the bottom layer representation of a string is a sequence of bytes. In the Go language, characters can be divided into two types: for English characters that occupy 1 byte, you can use byte (or unit8); for other characters that occupy 1 to 4 bytes, you can use rune (or int32), such as Chinese, special symbols, etc.



The Go language divides characters into two types: 

1. byte 
2. rune

byte is an alias of type unit8, which is used to store ASCII characters occupying 1 byte, such as English characters, and returns the original byte of the character. rune is an alias of type int32, which is used to store multi-byte characters, such as **Chinese characters** occupying 3 bytes, and returns the Unicode code point value of the character.
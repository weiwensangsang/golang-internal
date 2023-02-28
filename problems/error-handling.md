# Error Handling

Golang's Error Handling has always been controversial, and the community continue to propose various improvement plans. As a language-level error support, Go's implementation of error is extremely simple, even rudimentary. So, what is the best practice for Golang Error Handling?

### Errors are values

"Errors are values" originated from the design concept of error by Rob Pike, one of the founders of the Go language. Rob Pike believes that error is equal to other return values of functions, and it is just one of many return values, and there is nothing special about it. Therefore, errors are treated just like the return value of a function.

After understanding Rob Pike's design philosophy, you can naturally understand why it appears repeatedly in the Go language:

```go
if err != nil {
return err
}
```

Because error is an ordinary return value, it is also handled simply in the code flow. However, for most cases of the Go language, the error handling method only needs to determine the non-null return. Therefore, the Go language is often complained about for its lengthy and repetitive error handling.



The error design of the Go language has been controversial in long-term practice. The main complaints of developers are:

1. Error processing is interspersed in the code of Go language, which separates the normal logic code and affects the readability.
2. A large number of repeated if err != nil fragments cannot be simplified.
3. A simple return err cannot be used in all scenarios.



Go2 plans to help simplify error's separation of code logic and a large number of repetitions of if err != nil by introducing two keywords **handle** and **check**.

Go1

```go
func printSum(a, b string) error {
	x, err := strconv.Atoi(a)
	if err != nil {
		return err
	}
	y, err := strconv.Atoi(b)
	if err != nil {
		return err
	}
	fmt.Println("result:", x + y)
	return nil
}
```

Go2：

```go
func printSum(a, b string) error {
	handle err { return err }
	x := check strconv.Atoi(a)
	y := check strconv.Atoi(b)
	fmt.Println("result:", x + y)
	return nil
}
```
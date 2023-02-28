# Assertion

Type assertion (assertion) is an operation for interface value, the syntax is x.(T), x is an expression of interface type, and T is assertd type, the type being asserted.
         

There are two main scenarios for the use of assertions: If the asserted type is a concrete type, an instance class type, the assertion will check whether the dynamic type of x is the same as T, if they are the same, the result of the assertion is the dynamic value of x, of course the dynamic value type is T. In other words, the assertion on the concrete type actually fetches the dynamic value of x.
        

 If the asserted type is an interface type, the purpose of the assertion is to detect whether the dynamic type of x satisfies T, and if so, the result of the assertion is an expression that satisfies T, but its dynamic type and dynamic value are the same as x. In other words, an assertion on an interface type actually changes the type of x, usually the interface type of a larger method set, but preserves the original dynamic type and dynamic value.


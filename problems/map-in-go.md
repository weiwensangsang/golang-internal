# Map In Go

Go build a much more simple Map, simple but effective.

1. How map works?
2. Why is Go map not thread-safe?
2. Why is the load factor of Go map 6.5?
3. Why does the key of the map have to be of a comparable type?
5. How can we copy a map in Go?
6. Can we override the hashcode()?



The code for map is in src/runtime/map.go. The realization principle is a variant of the zipper method. To be honest, there is nothing new, so we simply analyze it from the four behaviors of init, get, set, and expansion.



Get
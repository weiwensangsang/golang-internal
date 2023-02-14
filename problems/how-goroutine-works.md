# How Goroutine Works

Let us start with some questions about Goroutine, 

1. Single-core CPU, open two Goroutines, one of which has an infinite loop, what will happen?
2. Processes and threads have IDs, why doesn't Goroutine have an ID?
3. How much is the right number of Goroutines, will it affect GC and scheduling?
4. What are the Goroutine leaks?
5. What can cause a Goroutine to hang?



At first, what is Goroutine?
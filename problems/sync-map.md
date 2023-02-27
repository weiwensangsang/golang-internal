

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



// fixme: https://juejin.cn/post/6844904100287496206


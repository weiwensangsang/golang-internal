# Array

### Two Sum

```go
func Solution(nums []int, target int) []int {
   indexMap := make(map[int]int)
   for currIndex, currNum := range nums {
      if requiredIdx, isPresent := indexMap[target-currNum]; isPresent {
         return []int{requiredIdx, currIndex}
      }
      indexMap[currNum] = currIndex
   }
   return []int{}
}

func TestSolution(t *testing.T) {
   input1 := []int{2, 7, 11, 15}
   input2 := 9
   expectedOutput := []int{0, 1}
   output := Solution(input1, input2)
   t.Logf("Solution(%v, %v) return %v, we want %v", input1, input2, output, expectedOutput)
}
```



Fixme

https://github.com/yuchia0221/Grind-75
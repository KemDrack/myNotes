
```go
func twoSum(nums []int, target int) []int {
    left:= 0
    right:= len(nums) - 1
    for left < right {
        currentSum:= nums[left] + nums[right]
        if currentSum == target {
            return  []int{left+1, right+1}
        }
        if currentSum > target{
            right-=1
        }else {
            left +=1
        }
    }
    return []int{}
}
```


Сложность O(n) по времени
Сложность O(1) по памяти
## 剑指 Offer 03.数组中重复的数字

### [题目传送门](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

#### 方法一：遍历数组
1. 由于只需要找出数组中任意一个重复的数字，因此遍历数组，遇到重复的数字即返回。为了判断一个数字是否重复遇到，使用集合存储已经遇到的数字，如果遇到的一个数字已经在集合中，则当前的数字是重复数字
2. Python 3 版本
    ```python
    class Solution:
        def findRepeatNumber(self, nums: List[int]) -> int:
            result_set = set()
            repeated = -1
            for _, v in enumerate(nums):
                if v in result_set:
                    repeated = v
                    break
                result_set.add(v)
            return repeated
    ```
3. Golang 版本
   ```go
   func findRepeatNumber(nums []int) int {
       resultMap := make(map[int]int)
       repeated := -1
       for i := 0; i < len(nums); i++ {
           _, ok := resultMap[nums[i]]
           if ok {
               repeated = nums[i]
               break
           }
           resultMap[nums[i]] = 1
       }
       return repeated
   }
   ```

4. 复杂度分析
   - 时间复杂度：O(N)，遍历数组一遍。使用哈希集合(HashSet)，添加元素的时间复杂度为 O(1) ，故总的时间复杂度 O(N)
   - 空间复杂度：O(N)。不重复的每个元素都可能存入集合，因此占用 O(N) 额外空间

## 剑指 Offer 11.旋转数组的最小数字

### [题目传送门](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

#### 方法一：遍历

1. 思路

    - 直接遍历寻找旋转点

2. Python 3 版本

   ```python
   class Solution:
       def minArray(self, numbers: List[int]) -> int:
           for i in range(len(numbers) - 1):
               if numbers[i] <= numbers[i+1]:
                   continue
               else:
                   return (numbers[i+1:] + numbers[:i+1])[0]
           return numbers[0]
   ```

3. Golang 版本

   ```go
   func minArray(numbers []int) int {
       loop := false
       resultSlice := make([]int, 0)
       for i := 0; i < len(numbers) - 1; i++ {
           if numbers[i] <= numbers[i+1] {
               continue
           } else {
               resultSlice = append(resultSlice, numbers[i+1:]...)
               resultSlice = append(resultSlice, numbers[:i+1]...)
               loop = true
           }
       }
       if loop {
           return resultSlice[0]
       } else {
           return numbers[0]
       }
   }
   ```

4. 复杂度分析

    - 时间复杂度：O(N)
    - 空间复杂度：O(1)

#### 方法二：二分法

1. 思路

    - 排序数组的查找问题首先考虑使用二分法解决，其可将遍历法的线性级别时间复杂度降低至对数级别

2. 算法流程

    - 初始化：声明 i, j 双指针分别指向 nums 数组左右两端
    - 循环二分：设 m = (i + j) / 2 为每次二分的中点，可分为以下三种情况：
        - 当 nums[m] > nums[j] 时：m 一定在左排序数组中，即旋转点 x 一定在 [m + 1, j] 闭区间内，因此执行 i = m + 1
        - 当 nums[m] < nums[j] 时：m 一定在右排序数组中，即旋转点 x 一定在[i, m] 闭区间内，因此执行 j = m
        - 当 nums[m] = nums[j] 时：无法判断 m 在哪个排序数组中，即无法判断旋转点 x 在 [i, m] 还是 [m + 1, j] 区间中。解决方案：执行 j = j - 1 缩小判断范围
    - 返回值：当 i = j 时跳出二分循环，并返回旋转点的值 nums[i] 即可

3. Python 3 版本

   ```python
   class Solution:
       def minArray(self, numbers: List[int]) -> int:
           i, j = 0, len(numbers) - 1
           while i < j:
               m = (i + j) // 2
               if numbers[m] < numbers[j]:
                   j = m
               elif numbers[m] > numbers[j]:
                   i = m + 1
               else:
                   j -= 1
           return numbers[i]
   ```

4. Golang 版本

   ```go
   func minArray(numbers []int) int {
       i, j := 0, len(numbers) - 1
       for i < j {
           m := (i + j) / 2
           if numbers[m] < numbers[j] {
               j = m
           } else if numbers[m] > numbers[j] {
               i = m + 1
           } else {
               j = j - 1
           }
       }
       return numbers[i]
   }
   ```

5. 复杂度分析

    - 时间复杂度 <img src="http://latex.codecogs.com/svg.latex? O(log_2N)" height="14px">：在特例情况下(例如 [1, 1, 1, 1])，会退化到 O(N)
    - 空间复杂度 O(1)：i, j, m 变量使用常数大小的额外空间



------

### [转载文章](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/solution/mian-shi-ti-11-xuan-zhuan-shu-zu-de-zui-xiao-shu-3/)




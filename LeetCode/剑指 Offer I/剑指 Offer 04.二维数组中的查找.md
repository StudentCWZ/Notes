## 剑指 Offer 04.二维数组中的查找

### [题目传送门](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

#### 方法一：暴力

1. 思路
   - 如果不考虑二维数组排好序的特点，直接遍历整个二维数组的每一个元素，判断目标值是否在二维数组中存在
   - 依次遍历二维数组的每一行和每一列。如果找到一个元素等于目标值，则返回 true 。如果遍历完毕仍未找到等于目标值的元素，则返回 false

2. Python 3 版本

   ```python
   class Solution:
       def findNumberIn2DArray(self, matrix: List[List[int]], target: int) -> bool:
           for row in matrix:
               if target in row:
                   return True 
           return False
   ```

3. Golang 版本

   ```go
   func findNumberIn2DArray(matrix [][]int, target int) bool {
       for _, s := range matrix {
           for _, v := range s {
               if v == target {
                   return true
               }
           } 
       }
       return false
   }
   ```

4. 复杂度分析

    - 时间复杂度 O(NM)：二维数组中的每个元素都被遍历，因此时间复杂度为二维数组的大小
    - 空间复杂度 O(1)

#### 方法二：二叉搜索树 + dfs

1. 思路

    - 我们将矩阵逆时针旋转 45° ，发现其类似于二叉搜索树 ，即对于每个元素，其左分支元素更小、右分支元素更大。通过从根节点开始搜索，遇到比 target 大的元素就向左，反之向右，即可找到目标值 target
    - 根节点对应的是矩阵的左下角和右上角元素，本文称之为标志数，以 matrix 中的左下角元素为标志数 flag ，则有:
        - 若 flag > target ，则 target 一定在 flag 所在行的上方 ，即 flag 所在行可被消去
        - 若 flag < target ，则 target 一定在 flag 所在列的右方 ，即 flag 所在列可被消去

2. 算法流程

    - 从矩阵 matrix 左下角元素(索引设为 (i, j) )开始遍历，并与目标值对比：
        - 当 `matrix[i][j] > target` 时，执行 i-- ，即消去第 i 行元素
        - 当 `matrix[i][j] < target` 时，执行 j++ ，即消去第 j 列元素
        - 当 `matrix[i][j] = target` 时，返回 true ，代表找到目标值
    - 若行索引或列索引越界，则代表矩阵中无目标值，返回 false

3. Python 3 版本

   ```python
   class Solution:
       def findNumberIn2DArray(self, matrix: List[List[int]], target: int) -> bool:
           i, j = len(matrix) - 1, 0
           while i >= 0 and j < len(matrix[0]):
               if matrix[i][j] > target:
                   i -= 1
               elif matrix[i][j] < target:
                   j += 1
               else:
                   return True
           return False
   ```

4. Golang 版本

   ```go
   func findNumberIn2DArray(matrix [][]int, target int) bool {
       i, j := len(matrix) - 1, 0
       for i >= 0 && j < len(matrix[0]) {
           if matrix[i][j] > target {
               i -= 1
           } else if matrix[i][j] < target {
               j += 1
           } else {
               return true
           }
       }
       return false
   }
   ```

5. 复杂度分析

    - 时间复杂度 O(N + M)：其中，N 和 M 分别为矩阵行数和列数，此算法最多循环 M + N 次
    - 空间复杂度 O(1)：i, j 指针使用常数大小额外空间



------

### [转载文章](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/solution/mian-shi-ti-04-er-wei-shu-zu-zhong-de-cha-zhao-zuo/)




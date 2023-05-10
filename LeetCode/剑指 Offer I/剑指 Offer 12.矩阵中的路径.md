## 剑指 Offer 12.矩阵中的路径

### [题目传送门](https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/)

#### 方法一：深度优先搜索(DFS) + 剪枝

1. 思路

    - 深度优先搜索：可以理解为暴力法遍历矩阵中所有字符串可能性。DFS 通过递归，先朝一个方向搜到底，再回溯至上个节点，沿另一个方向搜索，以此类推
    - 剪枝：在搜索中，遇到这条路不可能和目标字符串匹配成功的情况(例如：此矩阵元素和目标字符不同、此元素已被访问)，则应立即返回，称之为可行性剪枝

2. 算法流程

    - DFS 解析

        - 递归参数：当前元素在矩阵 board 中的行列索引 i 和 j ，当前目标字符在 word 中的索引 k

        - 终止条件：

            - 返回 false
                - (1) 行或列索引越界
                - (2) 当前矩阵元素与目标字符不同
                - (3) 当前矩阵元素已访问过((3) 可合并至 (2))

            - 返回 true ：k = len(word) - 1 ，即字符串 word 已全部匹配

        - 递推工作：

            - 标记当前矩阵元素：将 board[i][j] 修改为空字符 '' ，代表此元素已访问过，防止之后搜索时重复访问
            - 搜索下一单元格：朝当前元素的 上、下、左、右 四个方向开启下层递归，使用或连接(代表只需找到一条可行路径就直接返回，不再做后续 DFS)，并记录结果至 res
            - 还原当前矩阵元素：将 `board[i][j]` 元素还原至初始值，即 word[k]

        - 返回值：返回布尔量 res ，代表是否搜索到目标字符串

3. Python 3 版本

   ```python
   class Solution:
       def exist(self, board: List[List[str]], word: str) -> bool:
           def dfs(i, j, k):
               if not 0 <= i < len(board) or not 0 <= j < len(board[0]) or board[i][j] != word[k]:
                   return False
               if k == len(word) - 1: return True
               board[i][j] = ''
               res = dfs(i + 1, j, k + 1) or dfs(i - 1, j, k + 1) or dfs(i, j + 1, k + 1) or dfs(i, j - 1, k + 1)
               board[i][j] = word[k]
               return res
   
           for i in range(len(board)):
               for j in range(len(board[0])):
                   if dfs(i, j, 0): return True
           return False
   ```

4. Golang 版本

   ```go
   type pair struct{ x, y int }
   
   var directions = []pair{{-1, 0}, {1, 0}, {0, -1}, {0, 1}} // 上下左右
   
   func exist(board [][]byte, word string) bool {
   	h, w := len(board), len(board[0])
   	vis := make([][]bool, h)
   	for i := range vis {
   		vis[i] = make([]bool, w)
   	}
   	var check func(i, j, k int) bool
   	check = func(i, j, k int) bool {
   		if board[i][j] != word[k] { // 剪枝：当前字符不匹配
   			return false
   		}
   		if k == len(word)-1 { // 单词存在于网格中
   			return true
   		}
   		vis[i][j] = true
   		defer func() { vis[i][j] = false }() // 回溯时还原已访问的单元格
   		for _, dir := range directions {
   			if newI, newJ := i+dir.x, j+dir.y; 0 <= newI && newI < h && 0 <= newJ && newJ < w && !vis[newI][newJ] {
   				if check(newI, newJ, k+1) {
   					return true
   				}
   			}
   		}
   		return false
   	}
   	for i, row := range board {
   		for j := range row {
   			if check(i, j, 0) {
   				return true
   			}
   		}
   	}
   	return false
   }
   ```

5. 复杂度分析

    - 时间复杂度 <img src="http://latex.codecogs.com/svg.latex? O(3^KMN)" height="14px">：最差情况下，需要遍历矩阵中长度为 K 字符串的所有方案，时间复杂度为 <img src="http://latex.codecogs.com/svg.latex? O(3^K)" height="14px"> ；矩阵中共有 MN 个起点，时间复杂度为 O(MN)
        - 方案数计算：设字符串长度为 K ，搜索中每个字符有上、下、左、右四个方向可以选择，舍弃回头(上个字符)的方向，剩下 3 种选择，因此方案数的复杂度为 <img src="http://latex.codecogs.com/svg.latex? O(3^K)" height="14px">
    - 空间复杂度 O(K)：搜索过程中的递归深度不超过 K ，因此系统因函数调用累计使用的栈空间占用 O(K) (因为函数返回后，系统调用的栈空间会释放)。最坏情况下 K = M ，递归深度为 MN ，此时系统栈使用 O(MN) 的额外空间



------

### [转载文章](https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/solution/mian-shi-ti-12-ju-zhen-zhong-de-lu-jing-shen-du-yo/)




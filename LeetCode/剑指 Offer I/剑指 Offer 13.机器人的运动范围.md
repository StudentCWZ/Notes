## 剑指 Offer 13.机器人的运动范围

### [题目传送门](https://leetcode.cn/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

#### 方法一：深度优先遍历 DFS

1. 思路

    - 本题与矩阵中的路径类似，是典型的搜索&回溯问题。在介绍回溯算法算法前，为提升计算效率，首先讲述两项前置工作：数位之和计算、可达解分析

        - 数位之和计算

            - 设一数字 x ，向下取整除法符号 // ，求余符号 <img src="http://latex.codecogs.com/svg.latex? \odot"> ，则有：

                - x <img src="http://latex.codecogs.com/svg.latex? \odot"> 10：得到 x 的个位数字
                - x // 10：令 x 的十进制数向右移动一位，即删除个位数字

            - 因此，可通过循环求得数位和 s ，数位和计算的封装函数如下所示

              ```python
              def sums(x):
                  s = 0
                  while x != 0:
                      s += x % 10
                      x = x // 10
                  return s
              ```

        - 由于机器人每次只能移动一格(即只能从 x 运动至 x <img src="http://latex.codecogs.com/svg.latex? \pm"> 1)，因此每次只需计算 x 到 <img src="http://latex.codecogs.com/svg.latex? \pm"> 1 的数位和增量。本题说明 1 <img src="http://latex.codecogs.com/svg.latex? \leq"> n , m <img src="http://latex.codecogs.com/svg.latex? \leq"> 100 ，以下公式仅在此范围适用

        - 数位和增量公式：设 x 的数位和为 <img src="http://latex.codecogs.com/svg.latex? s_x"> ，x+1 的数位和为 <img src="http://latex.codecogs.com/svg.latex? s_{x+1}">

            - 当 (x + 1) <img src="http://latex.codecogs.com/svg.latex? \odot"> 10 = 0 时：<img src="http://latex.codecogs.com/svg.latex? s_{x+1}"> = <img src="http://latex.codecogs.com/svg.latex? s_x"> - 8，例如 19, 20 的数位和分别为 10, 2
            - 当 (x + 1) <img src="http://latex.codecogs.com/svg.latex? \odot"> 10 <img src="http://latex.codecogs.com/svg.latex? \neq"> 0 时：<img src="http://latex.codecogs.com/svg.latex? s_{x+1}"> = <img src="http://latex.codecogs.com/svg.latex? s_x"> + 1，例如 1, 2 的数位和分别为 1, 2

        - 以下代码为增量公式的三元表达式写法，将整合入最终代码中

          ```python
          s_x + 1 if (x + 1) % 10 else s_x - 8
          ```

        - 可达解分析

            - 根据数位和增量公式得知，数位和每逢进位突变一次。根据此特点，矩阵中满足数位和的解构成的几何形状形如多个等腰直角三角形，每个三角形的直角顶点位于 0, 10, 20, ... 等数位和突变的矩阵索引处
            - 三角形内的解虽然都满足数位和要求，但由于机器人每步只能走一个单元格，而三角形间不一定是连通的，因此机器人不一定能到达，称之为不可达解；同理，可到达的解称为可达解(本题求此解)
            - 根据可达解的结构和连通性，易推出机器人可 通过向右和向下移动，访问所有可达解。三角形内部：全部连通，易证；两三角形连通处：若某三角形内的解为可达解，则必与其左边或上边的三角形连通(即相交)，即机器人必可从左边或上边走进此三角形

          ![可达解结构分析](https://github.com/StudentCWZ/Notes/blob/main/Pictures/%E5%89%91%E6%8C%87%20Offer%20I/%E5%89%91%E6%8C%87%20Offer%2013.%E6%9C%BA%E5%99%A8%E4%BA%BA%E7%9A%84%E8%BF%90%E5%8A%A8%E8%8C%83%E5%9B%B4/%E5%8F%AF%E8%BE%BE%E8%A7%A3%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90.png)

2. 算法流程

    - 深度优先遍历 + DFS
        - 深度优先搜索：可以理解为暴力法模拟机器人在矩阵中的所有路径。DFS 通过递归，先朝一个方向搜到底，再回溯至上个节点，沿另一个方向搜索，以此类推
        - 剪枝：在搜索中，遇到数位和超出目标值、此元素已访问，则应立即返回，称之为可行性剪枝
    - 解析
        - 递归参数：当前元素在矩阵中的行列索引 i 和 j ，两者的数位和 si, sj
        - 终止条件：当 ① 行列索引越界 或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过时，返回 0 ，代表不计入可达解
        - 递推工作：
            - 标记当前单元格：将索引 (i, j) 存入 Set visited 中，代表此单元格已被访问过
            - 搜索下一单元格：计算当前元素的下、右两个方向元素的数位和，并开启下层递归
        - 回溯返回值：返回 1 + 右方搜索的可达解总数 + 下方搜索的可达解总数，代表从本单元格递归搜索的可达解总数

3. Python 3 版本

   ```python
   class Solution:
       def movingCount(self, m: int, n: int, k: int) -> int:
           def dfs(i, j, si, sj):
               if i >= m or j >= n or k < si + sj or (i, j) in visited: return 0
               visited.add((i,j))
               return 1 + dfs(i + 1, j, si + 1 if (i + 1) % 10 else si - 8, sj) + dfs(i, j + 1, si, sj + 1 if (j + 1) % 10 else sj - 8)
   
           visited = set()
           return dfs(0, 0, 0, 0)
   ```

4. Golang 版本

   ```go
   var n1, m1, k1 int
   var visited [][]bool
   
   func movingCount(m int, n int, k int) int {
   	m1 = m
   	n1 = n
   	k1 = k
   	visited = make([][]bool, m)
   	for i := 0; i < len(visited); i++ {
   		visited[i] = make([]bool, n)
   	}
   
   	return dfs(0, 0, 0, 0)
   }
   
   func dfs(i int, j int, si int, sj int) int {
   	if i >= m1 || j >= n1 || si+sj > k1 || visited[i][j] {
   		return 0
   	}
   	visited[i][j] = true
   
   	sj1 := sj + 1
   	si1 := si + 1
   	if (j+1)%10 == 0 {
   		sj1 = sj - 8
   	}
   	if (i+1)%10 == 0 {
   		si1 = si - 8
   	}
   
   	return 1 + dfs(i, j+1, si, sj1) + dfs(i+1, j, si1, sj)
   }
   ```

5. 复杂度分析

    - 时间复杂度 O(MN)：最差情况下，机器人遍历矩阵所有单元格，此时时间复杂度为 O(MN)
    - 空间复杂度 O(MN)：最差情况下，Set visited 内存储矩阵所有单元格的索引，使用 O(MN) 的额外空间

#### 方法二：广度优先遍历 BFS

1. 思路

    - BFS/DFS ：两者目标都是遍历整个矩阵，不同点在于搜索顺序不同。DFS 是朝一个方向走到底，再回退，以此类推；BFS 则是按照平推的方式向前搜索
    - BFS 实现：通常利用队列实现广度优先遍历

2. 算法流程

    - 初始化：将机器人初始点 (0, 0) 加入队列 queue
    - 迭代终止条件：queue 为空。代表已遍历完所有可达解
    - 迭代工作：
        - 单元格出队：将队首单元格的索引、数位和弹出，作为当前搜索单元格。
        - 判断是否跳过：若 ① 行列索引越界或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过时，执行 continue
        - 标记当前单元格：将单元格索引 (i, j) 存入 Set visited 中，代表此单元格已被访问过
        - 单元格入队：将当前元素的下方、右方单元格的索引、数位和加入 queue
    - 返回值：Set visited 的长度 len(visited) ，即可达解的数量

3. Python 3 版本

   ```python
   class Solution:
       def movingCount(self, m: int, n: int, k: int) -> int:
           queue, visited = [(0, 0, 0, 0)], set()
           while queue:
               i, j, si, sj = queue.pop(0)
               if i >= m or j >= n or k < si + sj or (i, j) in visited: continue
               visited.add((i,j))
               queue.append((i + 1, j, si + 1 if (i + 1) % 10 else si - 8, sj))
               queue.append((i, j + 1, si, sj + 1 if (j + 1) % 10 else sj - 8))
           return len(visited)
   ```

4. Golang 版本

   ```go
   func movingCount1(m int, n int, k int) int {
   	queue := list.List{}
   	visited := make([][]bool, m)
   	for i := 0; i < len(visited); i++ {
   		visited[i] = make([]bool, n)
   	}
   	queue.PushBack([]int{0, 0, 0, 0})
   	res := 0
   	for queue.Len() > 0 {
   		back := queue.Back()
   		queue.Remove(back)
   		bfs := back.Value.([]int)
   		i := bfs[0]
   		j := bfs[1]
   		si := bfs[2]
   		sj := bfs[3]
   		if i >= m || j >= n || si+sj > k || visited[i][j] {
   			continue
   		}
   		res++
   		visited[i][j] = true
   
   		sj1 := sj + 1
   		si1 := si + 1
   		if (j+1)%10 == 0 {
   			sj1 = sj - 8
   		}
   		if (i+1)%10 == 0 {
   			si1 = si - 8
   		}
   
   		queue.PushBack([]int{i + 1, j, si1, sj})
   		queue.PushBack([]int{i, j + 1, si, sj1})
   	}
   	return res
   }
   ```

5. 复杂度分析

    - 时间复杂度 O(MN)：最差情况下，机器人遍历矩阵所有单元格，此时时间复杂度为 O(MN)
    - 空间复杂度 O(MN)：最差情况下，Set visited 内存储矩阵所有单元格的索引，使用 O(MN) 的额外空间



------

### [转载文章](https://leetcode.cn/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/solution/mian-shi-ti-13-ji-qi-ren-de-yun-dong-fan-wei-dfs-b/)




## 剑指 Offer 10-I.斐波那契数列

### [题目传送门](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/)

#### 方法一：动态规划

1. 思路

    - 斐波那契数列的定义是 f(n+1) = f(n) + f(n-1) ，生成第 n 项的做法有以下几种：

        - 递归法
            - 原理：把 f(n) 问题的计算拆分成 f(n-1) 和 f(n-2) 两个子问题的计算，并递归，以 f(0) 和 f(1) 为终止条件
            - 缺点：大量重复的递归计算，例如 f(n) 和 f(n-1) 两者向下递归需要各自计算 f(n-2) 的值
        - 记忆化递归法
            - 原理：在递归法的基础上，新建一个长度为 n 的数组，用于在递归时存储 f(0) 至 f(n) 的数字值，重复遇到某数字则直接从数组取用，避免了重复的递归计算
            - 缺点：记忆化存储需要使用 O(N) 的额外空间

        - 动态规划

            - 原理：以斐波那契数列性质 f(n+1) = f(n) + f(n-1) 为转移方程

            - 从计算效率、空间复杂度上看，动态规划是本题的最佳解法

2. 算法思路

    - 动态规划解析
        - 状态定义：设 dp 为一维数组，其中 dp[i] 的值代表斐波那契数列第 i 个数字
        - 转移方程：dp[i+1] = dp[i] + dp[i-1] ，即对应数列定义 f(n+1) = f(n) + f(n-1)
        - 初始状态：dp[0] = 0 , dp[1] = 1 ，即初始化前两个数字
        - 返回值：dp[n] ，即斐波那契数列的第 n 个数字
    - 空间复杂度优化
        - 若新建长度为 n 的 dp 列表，则空间复杂度为 O(N)
        - 由于 dp 列表第 i 项只与第 i-1 和第 i-2 项有关，因此只需要初始化三个整形变量 sum, a, b ，利用辅助变量 sum 使 a, b 两数字交替前进即可
        - 节省了 dp 列表空间，因此空间复杂度降至 O(1)
    - 循环求余法
        - 大数越界：随着 n 增大, f(n) 会超过 Int32 甚至 Int64 的取值范围，导致最终的返回值错误
        - 求余运算规则：设正整数 x, y, p ，求余符号为 <img src="http://latex.codecogs.com/svg.latex? \odot"> ，则有 <img src="http://latex.codecogs.com/svg.latex? (x+y)\odot{p}=({x}\odot{p}+{y}\odot{p})\odot{p}" height="14px">
        - 解析：根据以上规则，可推出 <img src="http://latex.codecogs.com/svg.latex? f(n)\odot{p}=[f(n-1)\odot{p}+f(n-2)\odot{p}]\odot{p}" height="14px"> ，从而可以在循环过程中每次计算 <img src="http://latex.codecogs.com/svg.latex? sum=(a+b)\odot{1000000007}" height="14px"> ，此操作与最终返回前取余等价

3. Python 3 版本

   ```python
   class Solution:
       def fib(self, n: int) -> int:
           if n < 2: return n
           p, q, r = 0, 0, 1
           for _ in range(2, n+1):
               p = q
               q = r
               r = (p + q) % 1000000007
           return r
   ```

4. Python 3 优化版

   ```python
   class Solution:
       def fib(self, n: int) -> int:
           a, b = 0, 1
           for _ in range(n):
               a, b = b, a+b
           return a % 1000000007
   ```

5. Golang 版本

   ```go
   func fib(n int) int {
       const mod int = 1e9+7
       if n < 2 {
           return n
       }
       p, q, r := 0, 0, 1
       for i := 2; i <= n; i++ {
           p = q
           q = r
           r = (p + q) % mod
       } 
       return r
   }
   ```

6. 复杂度分析

    - 时间复杂度 O(N)：计算 f(n) 需循环 n 次，每轮循环内计算操作使用 O(1)
    - 空间复杂度 O(1)：几个标志变量使用常数大小的额外空间

#### 方法二：矩阵快速幂

1. 思路

    - 方法一的时间复杂度是 O(N) ，使用矩阵快速幂的方法可以降低时间复杂度

2. 算法流程

    - 首先我们可以构建这样一个递推关系：
      <div>
      <img src="http://latex.codecogs.com/svg.latex? \begin{bmatrix}1&1\\\\1&0\\\\\end{bmatrix}\begin{bmatrix}{F(n)}\\\\{F(n-1)}\end{bmatrix}=\begin{bmatrix}{F(n)+F(n-1)}\\\\{F(n)}\\\\\end{bmatrix}=\begin{bmatrix}{F(n+1)}\\\\F(n)\\\\\end{bmatrix}">
      </div>
    - 因此：
      <div>
      <img src="http://latex.codecogs.com/svg.latex? \begin{bmatrix}{F(n+1)}\\\\{F(n)}\\\\\end{bmatrix}={\begin{bmatrix}1&1\\\\1&0\\\\\end{bmatrix}^{n}}\begin{bmatrix}{F(1)}\\\\{F(0)}\\\\\end{bmatrix}">
      </div>
    - 令：
      <div>
      <img src="http://latex.codecogs.com/svg.latex? M={\begin{bmatrix}1&1\\\\1&0\\\\\end{bmatrix}">
      </div>
    - 因此只要我们能快速计算矩阵 M 的 n 次幂，就可以得到 F(n) 的值。如果直接求取 <img src="http://latex.codecogs.com/svg.latex? M^n"> ，时间复杂度是 O(N) ，可以定义矩阵乘法，然后用快速幂算法来加速这里 <img src="http://latex.codecogs.com/svg.latex? M^n"> 的求取

3. Python 3 版本

   ```python
   class Solution:
       def fib(self, n: int) -> int:
           MOD = 10 ** 9 + 7
           if n < 2:
               return n
   
           def multiply(a: List[List[int]], b: List[List[int]]) -> List[List[int]]:
               c = [[0, 0], [0, 0]]
               for i in range(2):
                   for j in range(2):
                       c[i][j] = (a[i][0] * b[0][j] + a[i][1] * b[1][j]) % MOD
               return c
   
           def matrix_pow(a: List[List[int]], n: int) -> List[List[int]]:
               ret = [[1, 0], [0, 1]]
               while n > 0:
                   if n & 1:
                       ret = multiply(ret, a)
                   n >>= 1
                   a = multiply(a, a)
               return ret
   
           res = matrix_pow([[1, 1], [1, 0]], n - 1)
           return res[0][0]
   ```

4. Golang 版本

   ```go
   const mod int = 1e9+7
   
   func multiply(a, b [2][2]int) [2][2]int {
       c := [2][2]int{
           {0, 0},
           {0, 0},
       }
       for i := 0; i < 2; i++ {
           for j := 0; j < 2; j++ {
               c[i][j] = (a[i][0] * b[0][j] + a[i][1] * b[1][j]) % mod
           }
       }
       return c
   }
   
   func martixPow(a [2][2]int, n int) [2][2]int {
       ret := [2][2]int{
           {1, 0},
           {0, 1},
       }
       for n > 0 {
           if (n & 1) == 1 {
               ret = multiply(ret, a)
           }
           n >>= 1
           a = multiply(a, a)
       }
       return ret
   }
   
   
   func fib(n int) int {
       if n < 2 {
           return n
       }
       paramsMartix := [2][2]int{
           {1, 1},
           {1, 0},
       }
       res := martixPow(paramsMartix, n-1)
       return res[0][0]
   }
   ```

5. 复杂度分析

    - 时间复杂度：O(logN)
    - 空间复杂度：O(1)




------

### [转载文章](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/solution/fei-bo-na-qi-shu-lie-by-leetcode-solutio-hbss/)




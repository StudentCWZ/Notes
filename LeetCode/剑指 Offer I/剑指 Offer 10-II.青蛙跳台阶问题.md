## 剑指 Offer 10-II.青蛙跳台阶问题

### [题目传送门](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

#### 方法一：动态规划

1. 思路

    - 此类求多少种可能性的题目一般都有递推性质，即 f(n) 和 f(n-1)…f(1) 之间是有联系的
    - 设跳上 n 级台阶有 f(n) 种跳法。在所有跳法中，青蛙的最后一步只有两种情况：跳上 1 级或 2 级台阶
        - 当为 1 级台阶：剩 n-1 个台阶，此情况共有 f(n-1) 种跳法
        - 当为 2 级台阶：剩 n-2 个台阶，此情况共有 f(n-2) 种跳法
    - f(n) 为以上两种情况之和，即 f(n) = f(n-1) + f(n-2) ，以上递推性质为斐波那契数列
    - 本题可转化为求斐波那契数列第 n 项的值 ，与 [面试题10-I.斐波那契数列](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/) 等价，唯一的不同在于起始数字不同
        - 青蛙跳台阶问题：f(0) = 1 , f(1) = 1 , f(2) = 2
        - 斐波那契数列问题：f(0) = 0 , f(1) = 1 , f(2) = 1

2. Python 3 版本

   ```python
   class Solution:
       def numWays(self, n: int) -> int:
           if n == 0: return 1
           if n == 1: return 1
           p, q, r = 0, 1, 1
           for _ in range(2, n+1):
               p = q
               q = r
               r = (p + q) % 1000000007
           return r
   ```

3. Golang 版本

   ```go
   func numWays(n int) int {
       const mod int = 1e9+7
       if n == 0 {
           return 1
       }
       if n == 1 {
           return 1
       }
       p, q, r := 0, 1, 1
       for i := 2;i <= n;i++ {
           p = q
           q = r
           r = (p + q) % mod
       }
       return r
   }
   ```

4. 复杂度分析

    - 时间复杂度 O(N)：计算 f(n) 需循环 n 次，每轮循环内计算操作使用 O(1)
    - 空间复杂度 O(1)：几个标志变量使用常数大小的额外空间



------

### [转载文章](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/solution/mian-shi-ti-10-ii-qing-wa-tiao-tai-jie-wen-ti-dong/)




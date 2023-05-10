## 剑指 Offer 05.替换空格

### [题目传送门](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/)

#### 方法一：遍历添加

1. 思路

    - 在 Python 和 Java 等语言中，字符串都被设计成不可变的类型，即无法直接修改字符串的某一位字符，需要新建一个字符串实现

2. 算法流程

    - 初始化一个 list (Python) / StringBuilder (Java) ，记为 res
    - 遍历列表 s 中的每个字符 c
        - 当 c 为空格时：向 res 后添加字符串 "%20"
        - 当 c 不为空格时：向 res 后添加字符 c

    - 将列表 res 转化为字符串并返回

3. Python 3 版本(API)

   ```python
   class Solution:
       def replaceSpace(self, s: str) -> str:
           s = s.replace(' ', '%20')
           return s
   ```

4. Python 3 版本

   ```python
   class Solution:
       def replaceSpace(self, s: str) -> str:
           res = []
           for _, v in enumerate(s):
               if v == " ":
                   v = "%20"
               res.append(v)
           s = ''.join(res)
           return s
   ```

5. Golang 版本(API)

   ```go
   func replaceSpace(s string) string{
       return strings.Replace(s, " ", "%20", -1)
   }
   ```

6. Golang 版本

   ```go
   func replaceSpace(s string) string {
      resultSlice := make([]string, 0)
      for i := 0; i < len(s); i++ {
         vStr := string(s[i])
         if vStr == " " {
            vStr = "%20"
         }
         resultSlice = append(resultSlice, vStr)
      }
      resultStr := strings.Join(resultSlice, "")
      return resultStr
   }
   ```

7. 复杂度分析

   - 时间复杂度 O(N)：遍历使用 O(N)，每轮添加(修改)字符操作使用 O(1)
   - 空间复杂度 O(N)：新建的 list 都使用了线性大小的额外空间



------

### [转载文章](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/solution/mian-shi-ti-05-ti-huan-kong-ge-ji-jian-qing-xi-tu-/)




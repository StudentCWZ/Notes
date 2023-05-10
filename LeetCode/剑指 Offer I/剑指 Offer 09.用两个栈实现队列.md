## 剑指 Offer 09.用两个栈实现队列

### [题目传送门](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

#### 方法一：双栈

1. 思路

    - 栈无法实现队列功能：栈底元素（对应队首元素）无法直接删除，需要将上方所有元素出栈
    - 双栈可实现列表倒序：设有含三个元素的栈 A = [1, 2, 3] 和空栈 B = [] 。若循环执行 A 元素出栈
    - 添加入栈 B ，直到栈 A 为空，则 A = [] , B = [3, 2, 1]，即栈 B 元素实现栈 A 元素倒序 。
      利用栈 B 删除队首元素： 倒序后，B 执行出栈则相当于删除了 A 的栈底元素，即对应队首元素

2. Python 3 版本

   ```python
   class CQueue:
   
       def __init__(self):
           self.stack_A = []
           self.stack_B = []
   
   
       def appendTail(self, value: int) -> None:
           self.stack_A.append(value)
   
   
       def deleteHead(self) -> int:
           if self.stack_B:
               return self.stack_B.pop()
           if not self.stack_A:
               return -1
           while self.stack_A:
               self.stack_B.append(self.stack_A.pop())
           return self.stack_B.pop()
   ```

3. Golang 版本

   ```go
   type CQueue struct {
       inStack, outStack []int
   
   }
   
   
   func Constructor() CQueue {
       return CQueue{}
   }
   
   
   func (this *CQueue) AppendTail(value int)  {
       this.inStack = append(this.inStack, value)
   }
   
   
   func (this *CQueue) DeleteHead() int {
       if len(this.outStack) == 0 {
           if len(this.inStack) == 0 {
               return -1
           }
           this.inAndOut()
       }
       value := this.outStack[len(this.outStack) - 1]
       this.outStack = this.outStack[:len(this.outStack)-1]
       return value
   }
   
   
   func (this *CQueue) inAndOut() {
       for len(this.inStack) > 0 {
           this.outStack = append(this.outStack, this.inStack[len(this.inStack) - 1])
           this.inStack = this.inStack[:len(this.inStack) - 1]
       }
   }
   ```

    4. 复杂度分析
        - 时间复杂度：appendTail 为 O(1) ，deleteHead 为均摊 O(1) 。对于每个元素，至多入栈和出栈各两次，故均摊复杂度为 O(1)
        - 空间复杂度：O(n) 。其中 n 是操作总数。对于有 n 次 appendTail 操作的情况，队列中会有 n 个元素，故空间复杂度为 O(n)



------

### [转载文章](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/solution/mian-shi-ti-09-yong-liang-ge-zhan-shi-xian-dui-l-3/)




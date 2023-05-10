## 剑指 Offer 06.从尾到头打印链表

### [题目传送门](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

#### 方法一：递归法

1. 思路

    - Python 算法流程
        - 递推阶段：每次传入 head.next ，以 head == None(即走过链表尾部节点)为递归终止条件，此时返回空列表 []
        - 回溯阶段：利用 Python 语言特性，递归回溯时每次返回 当前 list + 当前节点值 [head.val] ，即可实现节点的倒序输出

2. Python 3 版本

   ```python
   class ListNode:
       def __init__(self, x):
           self.val = x
           self.next = None
   
   
   class Solution:
       def reversePrint(self, head: ListNode) -> List[int]:
           if not head:
               return []
           return self.reversePrint(head.next) + [head.val]
   ```

3. 复杂度分析

    - 时间复杂度 O(N)：遍历链表，递归 N 次
    - 空间复杂度度 O(N)：系统递归需要使用 O(N) 的栈空间

#### 方法二：辅助栈法

1. 思路

    - 入栈：遍历链表，将各节点值 push 入栈
    - 出栈：将各节点值 pop 出栈，存储于数组并返回

2. Python 3 版本

   ```python
   class ListNode:
       def __init__(self, x):
           self.val = x
           self.next = None
   
   
   class Solution:
       def reversePrint(self, head: ListNode) -> List[int]:
           stack = []
           while head:
               stack.append(head.val)
               head = head.next
           return stack[::-1]
   ```

3. Golang 版本

   ```go
   type ListNode struct {
       Val int
       Next *ListNode
   }
   
   
   func reversePrint(head *ListNode) []int {
       /*1.遍历链表存入数组 2.数组反转*/
       var resultSlice []int
       if head == nil {
           return nil
       }
       for head != nil {
           resultSlice = append(resultSlice, head.Val)
           head = head.Next
       }
       reverse(resultSlice)
       return resultSlice
   }
   
   func reverse(list []int)[]int{
       n:=len(list)
       for i:= 0; i < n / 2; i++ {
           list[i], list[n-i-1] = list[n-i-1], list[i]
       }
       return list
   }
   ```

4. Golang 另一版本

   ```go
   type ListNode struct {
       Val int
       Next *ListNode
   }
   
   func reversePrint(head *ListNode) []int {
       /*双栈*/
       inStack := make([]int, 0)
       reverseStack := make([]int, 0)
       for head != nil {
           inStack = append(inStack, head.Val)
           head = head.Next
       } 
       for len(inStack) > 0 {
           val := inStack[len(inStack) - 1]
           reverseStack = append(reverseStack, val)
           inStack = inStack[:len(inStack)-1]
       }
       return reverseStack
   }
   ```

5. 复杂度分析
    - 时间复杂度 O(N)：入栈和出栈共使用 O(N) 时间
    - 空间复杂度 O(N)：辅助栈 stack 使用 O(N) 的额外空间



   ------

### [转载文章](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/solution/mian-shi-ti-06-cong-wei-dao-tou-da-yin-lian-biao-d/)

   


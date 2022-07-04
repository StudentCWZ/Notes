## 剑指 Offer 07.重建二叉树

### [题目传送门](https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/)

#### 方法一：分治算法

1. 思路

    - 前序遍历性质：节点按照 [ 根节点 | 左子树 | 右子树 ] 排序
    - 中序遍历性质：节点按照 [ 左子树 | 根节点 | 右子树 ] 排序
    - 根据以上性质，可得出以下推论：
        - 前序遍历的首元素 为 树的根节点 node 的值
        - 在中序遍历中搜索根节点 node 的索引 ，可将中序遍历划分为 [ 左子树 | 根节点 | 右子树 ]
        - 根据中序遍历中的左(右)子树的节点数量，可将前序遍历划分为 [ 根节点 | 左子树 | 右子树 ]
    - 通过以上三步，可确定三个节点：1.树的根节点、2.左子树根节点、3.右子树根节点
    - 根据分治算法思想，对于树的左、右子树，仍可复用以上方法划分子树的左右子树

2. 算法流程

    - 递推参数：根节点在前序遍历的索引 root 、子树在中序遍历的左边界 left 、子树在中序遍历的右边界 right

    - 终止条件：当 left > right ，代表已经越过叶节点，此时返回 null

    - 递推工作：

        - 建立根节点 node ：节点值为 preorder[root]

        - 划分左右子树：查找根节点在中序遍历 inorder 中的索引 i

        - 构建左右子树：开启左右子树递归

          |        |     根节点索引      | 中序遍历左边界 | 中序遍历右边界 |
                 | :----: | :-----------------: | :------------: | :------------: |
          | 左子树 |      root + 1       |      left      |     i - 1      |
          | 右子树 | root + 1 + i - left |     i + 1      |     right      |

    - 返回值：回溯返回 node ，作为上一层递归中根节点的左/右子节点

3. Python 3 版本

   ```python
   # Definition for a binary tree node.
   class TreeNode:
       def __init__(self, x):
           self.val = x
           self.left = None
           self.right = None
   
           
   class Solution:
       def buildTree(self, preorder: List[int], inorder: List[int]) -> TreeNode:
           dic = {}
           for i in range(len(inorder)):
               dic[inorder[i]] = i
   
           def recur(root: TreeNode, left: int, right: int) -> TreeNode:
               if left > right: return
               node = TreeNode(preorder[root])
               i = dic[preorder[root]]
               node.left = recur(root + 1, left, i-1)
               node.right = recur(root + 1 + i - left, i+1, right)
               return node
       
           return recur(0, 0, len(inorder) - 1)
   ```

4. Golang 版本

   ```go
   type TreeNode struct {
       Val int
       Left *TreeNode
       Right *TreeNode
   }
    
   
   func buildTree(preorder []int, inorder []int) *TreeNode {
       hashMap := make(map[int]int)
       for i := 0; i < len(inorder); i++ {
           hashMap[inorder[i]] = i
       }
       var recur func(root, left, right int) *TreeNode 
       recur = func(root, left, right int) *TreeNode {
           if left > right {
               return nil
           }
           node := &TreeNode{
               Val: preorder[root],
           }
           j := hashMap[preorder[root]]
           node.Left = recur(root + 1, left, j - 1)
           node.Right = recur(root + 1 + j - left, j + 1, right)
           return node
       }
       return recur(0, 0, len(inorder) - 1)
   }
   ```

5. 复杂度分析

    - 时间复杂度 O(N)：其中 N 为树的节点数量。初始化 HashMap 需遍历 inorder ，占用 O(N) 。递归共建立 N 个节点，每层递归中的节点建立、搜索操作占用 O(1) ，因此使用 O(N) 时间
    - 空间复杂度 O(N)：HashMap 使用 O(N) 额外空间；最差情况下(输入二叉树为链表时)，递归深度达到 N ，占用 O(N) 的栈帧空间；因此总共使用 O(N) 空间



------

### [转载文章](https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/solution/mian-shi-ti-07-zhong-jian-er-cha-shu-di-gui-fa-qin/)




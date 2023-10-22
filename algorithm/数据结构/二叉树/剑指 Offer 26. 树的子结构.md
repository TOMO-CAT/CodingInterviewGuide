# 剑指 Offer 26. 树的子结构
 ## 题目 
难度: 中等

输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)

B是A的子结构， 即 A中有出现和B相同的结构和节点值。

例如:<br>
给定的树 A:

     3<br>
    / \<br>
   4   5<br>
  / \<br>
 1   2<br>
给定的树 B：

   4 <br>
  /<br>
 1<br>
返回 true，因为 B 与 A 的一个子树拥有相同的结构和节点值。

**示例 1：**

```
输入：A = [1,2,3], B = [3,1]
输出：false

```


**示例 2：**

```
输入：A = [3,4,5,1,2], B = [4,1]
输出：true
```


**限制：**

0 <= 节点个数 <= 10000
来源: 力扣（LeetCode）
链接: https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 答案

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    // 判断B是否是A同根的子结构
    bool helper(TreeNode* A, TreeNode* B) {
        // 处理特殊值
        if (!A && B) {
            return false;
        }
        if (!B) {
            return true;
        }

        if (A->val != B->val) {
            return false;
        }

        return helper(A->left, B->left) && helper(A->right, B->right);
    }

    bool isSubStructure(TreeNode* A, TreeNode* B) {
        if (!A || !B) {
            return false;
        }

        if (helper(A, B)) {
            return true;
        }

        return isSubStructure(A->left, B) || isSubStructure(A->right, B);
    }
};
```


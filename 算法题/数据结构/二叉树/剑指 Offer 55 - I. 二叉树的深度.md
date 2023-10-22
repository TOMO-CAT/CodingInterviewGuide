# 剑指 Offer 55 - I. 二叉树的深度

## 题目

难度: 简单

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

例如：

给定二叉树 [3,9,20,null,null,15,7]，

```bash
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度 3 。

**提示：**

注意：本题与主站 104 题相同：<https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/>

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/>  
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

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
    int res = 0;

    void helper(TreeNode* root, int depth) {
        if (!root) {
            res = std::max(res, depth);
            return;
        }

        helper(root->left, depth + 1);
        helper(root->right, depth + 1);
    }

    int maxDepth(TreeNode* root) {
        helper(root, 0);
        return res;
    }
};
```

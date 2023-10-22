# 剑指 Offer 55 - II. 平衡二叉树

## 题目

难度: 简单

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

**示例 1:**

给定二叉树 [3,9,20,null,null,15,7]

```
    3
   / \
  9  20
    /  \
   15   7
```

返回 true 。

**示例 2:**

给定二叉树 [1,2,2,3,3,null,null,4,4]

```
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4

```

返回 false 。

注意：本题与主站 110 题相同：<https://leetcode-cn.com/problems/balanced-binary-tree/>

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/>  
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 答案

### 1. 自顶向下的递归

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
    int height(TreeNode* root) {
        if (!root) {
            return 0;
        }
        return std::max(height(root->left), height(root->right)) + 1;
    }

    bool isBalanced(TreeNode* root) {
        if (!root) {
            return true;
        }

        return std::abs(height(root->left) - height(root->right)) <= 1 && isBalanced(root->left) && isBalanced(root->right);
    }
};
```

### 2. 自底向上的递归

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
    int height(TreeNode* root) {
        if (!root) {
            return 0;
        }
        int left_height = height(root->left);
        int right_height = height(root->right);
        if (left_height == -1 || right_height == -1 || std::abs(left_height - right_height) > 1) {
            return -1;
        }
        return std::max(left_height, right_height) + 1;
    }

    bool isBalanced(TreeNode* root) {
        return height(root) != -1;
    }
};
```

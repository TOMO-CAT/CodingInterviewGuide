# 剑指 Offer 68 - I. 二叉搜索树的最近公共祖先

## 题目

难度: 简单

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

例如，给定如下二叉搜索树:  root = [6,2,8,0,4,7,9,null,null,3,5]

![](image/image-20231022212357922.png)

**示例 1:**

```
输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
输出: 6 
解释: 节点 2 和节点 8 的最近公共祖先是 6。

```

**示例 2:**

```
输入: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
输出: 2
解释: 节点 2 和节点 4 的最近公共祖先是 2, 因为根据定义最近公共祖先节点可以为节点本身。
```

**说明:**

- 所有节点的值都是唯一的。
- p、q 为不同节点且均存在于给定的二叉搜索树中。

**注意**：本题与主站 235 题相同：<https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/>

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/>  
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
    // 注意这是二叉搜索树
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        while (root) {
            // 1) 当两个结点都比root大时说明在右子树
            if (p->val > root->val && q->val > root->val) {
                root = root->right;
                continue;
            }

            // 2) 当两个结点都比root小时说明在左子树
            if (p->val < root->val && q->val < root->val) {
                root = root->left;
                continue;
            }

            // 3) 一个在左侧一个在右侧
            break;
        }

        return root;
    }
};
```

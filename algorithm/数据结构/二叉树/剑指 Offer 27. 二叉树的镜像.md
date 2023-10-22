# 剑指 Offer 27. 二叉树的镜像
 ## 题目 
难度: 简单

请完成一个函数，输入一个二叉树，该函数输出它的镜像。

例如输入：

     4<br>
   /   \<br>
  2     7<br>
 / \   / \<br>
1   3 6   9<br>
镜像输出：

     4<br>
   /   \<br>
  7     2<br>
 / \   / \<br>
9   6 3   1

 

**示例 1：**

```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]

```




**限制：**

0 <= 节点个数 <= 1000

注意：本题与主站 226 题相同：<a href="https://leetcode-cn.com/problems/invert-binary-tree/">https://leetcode-cn.com/problems/invert-binary-tree/</a>
来源: 力扣（LeetCode）
链接: https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/
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
    TreeNode* mirrorTree(TreeNode* root) {
        // 递归呀
        if (!root) {
            return nullptr;
        }

        TreeNode* left = root->left;

        root->left = mirrorTree(root->right);
        root->right = mirrorTree(left);
        return root;
    }
};
```


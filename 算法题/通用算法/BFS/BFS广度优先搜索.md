# BFS广度优先搜索

## 简介

## 用途

广度优先搜索（BFS）最常用的应用是求根结点到目标结点的最短路径。

## BFS 与 DFS 区别

### 1. 二叉树遍历

DFS 遍历二叉树：

```c++
void dfs(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    dfs(root->left);
    dfs(root->right);
}
```

BFS 遍历二叉树：

```c++
void bfs(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    
    std::deque<TreeNode*> queue;
    queue.push_back(p);
    while (!queue.empty()) {
        TreeNode* node = queue.front();
        queue.pop_front();
        if (node->left != nullptr) {
            queue.push_back(node->left);
        }
        if (node->right != nullptr) {
            queue.push_back(node->right);
        }
    }
}
```

## Reference

[1] <https://leetcode-cn.com/problems/binary-tree-level-order-traversal/solution/bfs-de-shi-yong-chang-jing-zong-jie-ceng-xu-bian-l/>

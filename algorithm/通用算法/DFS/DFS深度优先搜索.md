# DFS 深度优先搜索

## DFS 与动态规划关系

以 LeetCode279 完全平方数为例，注意动态规划超时的题目可能可以转化成动态规划。

因为 DFS 本质上就是暴力搜索，复杂度较高。

## 回溯与深度优先搜索的关系

严格来说，搜索也是一种暴力枚举策略，传统的枚举需要固定 for 循环的层数，但是这样不能随意增减枚举层数，本文将介绍一种新的利用递归的方式枚举每个可能的选项，如果合法就继续下一个，如果所有选项都不合法就退回并尝试更换上一个的选项，继续枚举。这种方式就是回溯算法，常用深度优先搜索实现。

## 递归和 DFS 关系

> N 路 DFS 就是一棵完全生长的 N 叉树，而递归就是一叉树。
>
> 因此递归就是特殊的 DFS。

DFS 是用递归实现的，但是 DFS 一条路走到底沿原路返回的时候需要回溯中间变量。

注意区分两路 DFS（二叉树）和多路 DFS（多叉树）：

### 1. 两路 DFS

```c++
class Solution {
private:
    vector<string> res_;  // 存储结果
    int n_;               // 括号对数

    // DFS: 完全生长的二叉树 + 剪枝
    // 1) 初始条件: cur_str空的, left为0, right为0
    // 2) 退出条件: cur_str 长度达到2n
    // 3) 中间变量: cur_str

    // cur_str: 当前字符串
    // left: 当前用的左括号数量 0 <= left <= n
    // right: 当前用的右括号数量 o <= right <= n
    void DFS(string cur_str, int left, int right) {
        if (cur_str.size() == 2 * n_) {
            res_.push_back(cur_str);
        }

        if (left < n_) {
            DFS(cur_str + '(', left+1, right);
        }
        if (right < left) {
            DFS(cur_str + ')', left, right+1);
        }
    }
public:
    vector<string> generateParenthesis(int n) {
        n_ = n;
        DFS("", 0, 0);
        return res_;
    }
};
```

### 2. 多路 DFS

```c++
class Solution {
public:
    unordered_map<char, string> num2letter = {
        {'2', "abc"},
        {'3', "def"},
        {'4', "ghi"},
        {'5', "jkl"},
        {'6', "mno"},
        {'7', "pqrs"},
        {'8', "tuv"},
        {'9', "wxyz"}
    };

    // DFS深度优先搜索
    // 1) 退出递归的条件: 深度达到一定条件
    // 2) 更改条件变量, 比如posit++
    void DFS(int posit, const string &digits, const string &cur_str, vector<string> &res) {
        if (posit == digits.size()) {
            res.push_back(cur_str);
            return;
        }

        char cur_num = digits[posit];
        for (char s : num2letter[cur_num]) {
            DFS(posit+1, digits, cur_str + s, res);
        }
    }

    vector<string> letterCombinations(string digits) {
        if (digits == "") {
            return {};
        } 

        vector<string> res;
        DFS(0, digits, "", res);
        return res;
    }
};
```

## 原理

DFS（Deep First Search）深度优先搜索包括两个步骤：递归和回溯。深度优先指的是以深度为准则，先一条路直接走到底（即递归），再穷举所有可能的路。

> Tips：抽象一下 DSF 解题过程，就是让一棵 N 叉树完全生长，优化就是在非叶子节点提前剪枝。

## 注意事项

* 注意如果不是尾递归（比如 LeetCode37 是非尾递归，而 LeetCode22 是尾递归），那么最好让 `dfs()` 函数返回一个布尔值，如果返回 true 则提前结束递归
* 注意 DFS 本质还是暴力搜索，优化的手段在于提前剪枝（参考 LeetCode22 括号生成）

## 真题

* 17. 电话号码的字母组合
* 22. 括号生成
* 37. 解数独
* 39. 组合总和
* 40. 组合总和Ⅱ
* 46. 全排列
* 78. 子集
* 79. 单词搜索

## Reference

[1] <https://zhuanlan.zhihu.com/p/24986203>

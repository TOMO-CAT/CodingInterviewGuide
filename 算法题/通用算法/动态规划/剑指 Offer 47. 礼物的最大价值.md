# 剑指 Offer 47. 礼物的最大价值

## 题目

难度: 中等

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

**示例 1:**

```
输入: 
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
```

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/>  
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 答案

### 1. BFS

==超时了==，注意BFS超时的时候可以选择动态规划。

```c++
class Solution {
public:
    int ans = 0;

    // x, y: 坐标
    // cur: 当前的礼物价值
    void helper(vector<vector<int>>& grid, int x, int y, int cur) {
        int row_num = grid.size();
        int col_num = grid[0].size();

        if (x == row_num - 1 && y == col_num - 1) {
            ans = std::max(ans, cur);
        }

        // 向右走
        if (y < col_num - 1) {
            helper(grid, x, y + 1, cur + grid[x][y + 1]);
        }
        // 向下走
        if (x < row_num - 1) {
            helper(grid, x + 1, y, cur + grid[x + 1][y]);
        }
    }

    int maxValue(vector<vector<int>>& grid) {
        helper(grid, 0, 0, grid[0][0]);
        return ans;
    }
};
```

### 2. 动态规划

```c++
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();

        // dp[x][y] 表示
        std::vector<std::vector<int>> dp(m, std::vector<int>(n));

        // 初始状态设置
        dp[0][0] = grid[0][0];
        for (int i = 1; i < m; i++) {
            dp[i][0] = dp[i - 1][0] + grid[i][0];
        }
        for (int j = 1; j < n; j++) {
            dp[0][j] = dp[0][j - 1] + grid[0][j];
        }

        // 状态转移
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = std::max(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
        }

        return dp[m - 1][n - 1];
    }
};
```

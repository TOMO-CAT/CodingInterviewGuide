# 剑指 Offer 66. 构建乘积数组

## 题目

难度: 中等

给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B[i] 的值是数组 A 中除了下标 i 以外的元素的积, 即 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]。不能使用除法。

**示例:**

```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/>  
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 答案

### 1. 动态规划

两个 dp 数组，一个维护左边乘积，一个维护右边乘积：

```c++
class Solution {
public:
    vector<int> constructArr(vector<int>& a) {
        int n = a.size();
        if (n <= 0) {
            return {};
        }

        // 两个 dp 数组, 一个维护左边乘积, 一个维护右边乘积
        vector<int> dp_left(n);
        vector<int> dp_right(n);

        dp_left[0] = 1;
        dp_right[n - 1] = 1;

        for (int i = 1; i < n; i++) {
            dp_left[i] = dp_left[i - 1] * a[i - 1];
            dp_right[n - 1 - i] = dp_right[n - i] * a[n - i];
        }

        vector<int> res(n);
        for (int i = 0; i < n; i++) {
            res[i] = dp_left[i] * dp_right[i];
        }

        return res;
    }
};
```

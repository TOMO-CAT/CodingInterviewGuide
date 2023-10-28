# 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面

## 题目

难度: 简单

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数在数组的前半部分，所有偶数在数组的后半部分。

**示例：**

```
输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。
```

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/>  
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 答案

```c++
class Solution {
public:
    vector<int> exchange(vector<int>& nums) {
        // 双指针
        int n = nums.size();
        if (n <= 1) return nums;
        int left = 0;
        int right = n - 1;

        while (left < right) {
            while (left < n && nums[left] % 2 == 1) {
                left++;
            }
            while (right >= 0 && nums[right] % 2 == 0) {
                right--;
            }

            if (left >= right) {
                break;
            }

            std::swap(nums[left], nums[right]);
            left++;
            right--;
        }

        return nums;
    }
};
```

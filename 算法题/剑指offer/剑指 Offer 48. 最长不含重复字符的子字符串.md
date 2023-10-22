# 剑指 Offer 48. 最长不含重复字符的子字符串

## 题目

难度: 中等

请从字符串中找出一个最长的不包含重复字符的子字符串，计算该最长子字符串的长度。

**示例 1:**

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。

```

**示例 2:**

```
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。

```

**示例 3:**

```
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。

```

**注意**：本题与主站 3 题相同：<https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/>

> 来源: 力扣（LeetCode）  
> 链接: <https://leetcode.cn/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/>  
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 答案

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int res = 0;

        // 存储元素出现过的下标
        std::unordered_map<char, int> num2idx;

        // 双指针
        // [left, right] 内的子串是不重复的, 而且
        int left = 0;
        int right = 0;

        while (right < s.size()) {
            auto iter = num2idx.find(s[right]);
            if (iter != num2idx.end() && iter->second >= left) {
                res = std::max(res, right - left);
                left = iter->second + 1;
            }
            num2idx[s[right]] = right;
            right++;
        }

        // 注意这最后一步不能漏掉
        res = std::max(res, right - left);

        return res;
    }
};
```

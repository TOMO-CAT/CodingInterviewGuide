# 算法题常用pair操作

## 使用pair的原因

可以使用`pair`的容器作为用户自定义顺序的hash表，因为`map`或者`unordered_map`都做不到（前者是按照`less<T>`排序的，后者是胡乱排序的），所以应该使用`pair`的容器作为用户自定义顺序的hash表：

```c++
// Leetcode12. 整数转罗马数字
const vector<pair<int, string>> value2symbol = {
    {1000, "M"},
    {900,  "CM"},
    {500,  "D"},
    {400,  "CD"},
    {100,  "C"},
    {90,   "XC"},
    {50,   "L"},
    {40,   "XL"},
    {10,   "X"},
    {9,    "IX"},
    {5,    "V"},
    {4,    "IV"},
    {1,    "I"},
};
```

## pair与vector结合

```c++
// 定义
vector<pair<int, int>> value2value;

// 插入元素
value2value.emplace(i, j);

// 获取元素
auto [i, j] = value2value[p];

// 排序
std::sort(value2value.begin(), value2value.end(), [](const std::pair<int, int>& a, const std::pair<int, int>& b) {
    return (a.first < b.first) || (a.first == b.first && a.second < b.second);
});
```

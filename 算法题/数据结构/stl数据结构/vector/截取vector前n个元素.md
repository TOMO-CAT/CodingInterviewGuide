# 截取 vector 前 n 个元素

利用 `std::vector` 的构造函数：

```c++
std::vector<int> orig = {1, 2, 3, 4};
// 截取 orig 前两个元素构造 v1
// v1 等价于 {1, 2}
std::vector<int> v1(orig.begin(), orig.begin() + 2);
```

# 算法题常用array操作

## 使用array的原因

C++中数组需要显式初始化，否则值是未定义的，使用array可以简化代码。

> 最好还是使用vector来替换array，减少心智负担，提前确定size即可。

## 用法

### 1. 定义及初始化

```c++
// 定义但值未初始化, 是未定义的值
std::array<double, 100> data;

// 定义且零初始化
std::array<double, 100> data = {};

// 初始化器列表中的 4 个值用于初始化前 4 个元素，其余的元素都将为 0
std::array<double, 10> values {0.5, 1.0, 1.5, 2.0};

// 将所有元素设成给定值
values.fill(3.1415926);
```

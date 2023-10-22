# 算法题常用vector操作

## 初始化

```c++
// 初始化包含10个int元素的vector, 每个元素初始化为0
vector<int> vi(10, 0);
```

## 指定大小或容量

```c++
vector.resize(10);
vector.reserve(20);
```

## 拷贝数组

```c++
vector<int> vi2(vi.begin(), vi.end());
```

## 获取尾元素

```c++
vi.back();
```

## 多维数组

```c++
// 构造二维数组, dp[m][n]
vector<vector<int>> dp(m, vector<int>(n));
```

## 反转容器

```c++
std::vector<int> vi = {1, 2, 3, 4};
std::reverse(vi.begin(), vi.end());
```

## 交换元素

```c++
std::swap(vi[i], vi[j]);
```

## 排序

```c++
std::sort(vi.begin(), vi.end());
```

## 求最值

```c++
int max = *std::max_element(vi.begin(), vi.end());
int min = *std::min_element(vi.begin(), vi.end());
```

## 交换vector

```c++
vi.swap(vi2);
```

# 合并两个 vector

## Insert

```c++
// 声明
std::vector<std::string> v1, v2, v3;

// 给 v1 和 v2 赋值
// ...

// 将 v1 和 v2 内容合并到 v3
v3.insert(v3.end(), v1.begin(), v1.end());
v3.insert(v3.end(), v2.begin(), v2.end());
```

## merge

```c++
// 声明
std::vector<std::string> v1, v2, v3;

// 给 v1 和 v2 赋值
// ...

std::sort(v1.begin(), v1.end());
std::sort(v2.begin(), v2.end());

v3.resize(v1.size() + v2.size());
merge(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());
```

需要注意的是使用 merge 时需要提前为 v3 申请好内存空间，否则会报错。

# 算法题常用priority_queue 操作

## 优先队列

普通的队列是一种先进先出的数据结构，元素在队列尾追加，而从队列头删除。在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （first in, largest out）的行为特征。通常采用堆数据结构来实现。

## 包含头文件

```c++
#include <queue>
using std::priority_queue;

// 升序队列
priority_queue <int, vector<int>, greater<int>> q;
// 降序队列
priority_queue <int, vector<int>, less<int>>q;
```

## 自定义比较器

```c++
bool cmp(int a, int b) {
    return a > b;
}

std::priority_queue<int, std::vector<int>, decltype(&cmp)> q(cmp);
```

## 常用操作

### 1. 获取并弹出元素

```c++
q.top()
q.pop()
```

### 2. 添加元素

```c++
q.push(10)
```

### 3. 判断是否为空

```c++
q.empty()
```

## Reference

[1] <https://baike.baidu.com/item/%E4%BC%98%E5%85%88%E9%98%9F%E5%88%97/9354754?fr=aladdin>

[2] <https://blog.csdn.net/weixin_36888577/article/details/79937886>

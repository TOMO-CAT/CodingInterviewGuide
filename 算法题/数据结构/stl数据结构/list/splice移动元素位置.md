# splice 移动元素位置

## 简介

`list::splice` 是 C++ STL 中的内置函数，用于将元素从一个列表传输到另一个列表。

## 使用方法

#### 1. 参数

* `position` 是要操作的 list 对象的迭代器
* `list<T Allocator>&x` 被剪的对象

#### 2. 用法一：移动整个 List

会在 `position` 后把 `list<T Allocator>&x` 所有的元素到剪接到要操作的 list 对象：

```c++
void splice ( iterator position, list<T,Allocator>& x );
```

#### 3. 用法二：移动一个元素

只会把 it 的值剪接到要操作的 list 对象中：

```c++
void splice ( iterator position, list<T,Allocator>& x, iterator it );
```

#### 4. 用法三：移动一个范围

把 first 到 last 剪接到要操作的 list 对象中：

```c++
void splice ( iterator position, list<T,Allocator>& x, iterator first, iterator last );
```

# 算法题常用set操作

## 应用场景

### 1. 判断死循环

Leetcode202：快乐数

## unordered_set与set

### 1. 区别

|                | `set`                                  | `unordered_set`                     |
| -------------- | -------------------------------------- | ----------------------------------- |
| Ordering       | increasing order(by default)           | no ordering                         |
| Implementation | Self balancing BST like Red-Black Tree | Hash Table                          |
| Search time    | log(n)                                 | O(1) -> Average<br>O(n)->Worst Case |
| Insertion time | log(n) + Rebalance                     | Same as search                      |
| Deletion time  | log(n) + Rebalance                     | Same as search                      |

### 2. 使用set的场景

* 我们需要有序的数据
* 我们将不得不按顺序打印/访问数据
* 我们需要元素的前任/后继
* 由于`set`是有序的，因此我们可以在`set`元素上使用`binary_search()`、`lower_bound()`和`upper_bound()`之类的函数，这些函数不能在`unordered_set`上使用

### 3. 使用unordered_set的场景

* 我们需要保留一组不同的元素，且不需要排序

### 4. 时间复杂度

`BST`的时间复杂度都是`O(Log(n))`，但是哈希表可以达到`O(1)`，由此可见在常见操作中使用哈希表更快，使用`BST`时有如下考虑：

* 只需执行`BST`的有序遍历，就可以按顺序获取所有键，哈希表要实现此操作需要额外的方法
* `BST`可以方便实现范围查询，这种操作对于哈希表而言也不是常规的操作
* 与散列相比，`BST`更容易实现，我们可以轻松实现自定义`BST`，但是为了实现散列通常需要依赖于编程语言提供的库
* 使用自平衡`BST`可以保证所有操作都在`O(Log(n))`时间内进行，但是使用散列时`O(1)`只是平均时间，某些特定操作可能很费时（尤其在调整表大小时）

## 常用操作

### 1. 增删

```c++
// 插入元素
unordered_set.insert(val);
// 删除元素
unordered_set.erase(val);
```

### 2. 转化

```c++
// 转成vector
vec.assign(set.begin(), set.end());
```

### 3. 判断元素是否存在

最简单的方式是使用count函数：

```c++
if (myset.count(x)) {
   // x is in the set, count is 1
} else {
   // count zero, i.e. x not in the set
}
```

当你不仅需要判断元素是否存在，而且要访问该元素时，那么最好使用迭代器：

```c++
set< X >::iterator it = myset.find(x);
if (it != myset.end()) {
   // do something with *it
}
```

## Reference

[1] <https://www.itranslater.com/qa/details/2113763860665598976>

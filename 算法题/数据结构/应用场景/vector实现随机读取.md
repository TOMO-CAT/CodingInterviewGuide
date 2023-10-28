# vector实现随机读取

## 问题

现在需要实现一个类，它提供如下三种接口：

* `add`：添加元素
* `del`：删除元素
* `random`：返回随机一个元素

## 实现

* `std::unordered_map<int, int>` 存储元素到元素索引的hash，**实现 $O(1)$ 复杂度**的 `add` 与 `del`
* `std::vector<int>` 存储全量元素，随机数引擎获取一个元素对 `vector.size()` 取余后直接访问即可

```c++
std::vector<int> data_;
std::unordered_map<int, int> num2idx_;

void add(int num) {
    data_.push_back(num);
    num2idx_[num] = data_.size() - 1;
}

void del(int num) {
    int idx = num2idx_[num];
    std::swap(data_[idx], data_[data_.size() - 1]);  // 关键代码
    data_.pop_back();
}

int random() {
    int random_idx;
    // 生成一个随机数
    return data_[random_idx % data_.size()];
}
```

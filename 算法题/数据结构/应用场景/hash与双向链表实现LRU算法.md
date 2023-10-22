# 146. LRU 缓存

## 题目

难度：中等

* LRUCache(int capacity) 以 **正整数** 作为容量 capacity 初始化 LRU 缓存
* int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
* void put(int key, int value) 如果关键字 key 已经存在，则变更其数据值 value ；如果不存在，则向缓存中插入该组 key-value 。如果插入操作导致关键字数量超过 capacity ，则应该 **逐出** 最久未使用的关键字。

函数 get 和 put 必须以 O(1) 的平均时间复杂度运行。

**示例：**

```
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4

```

> 来源：力扣（LeetCode）  
> 链接：<https://leetcode-cn.com/problems/lru-cache/>  
>著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 思路：哈希链表

要让 put 和 get 方法的时间复杂度为 O(1)，我们可以总结出 cache 这个数据结构必要的条件：

1、显然 cache 中的元素==必须有顺序==，以区分最近使用的和久未使用的数据，当容量满了之后要删除最久未使用的那个元素腾位置

2、我们要在 cache 中==快速找某个 key== 是否已存在并得到对应的 val

3、每次访问 cache 中的某个 key，需要将这个元素变为最近使用的，也就是说 cache 要==支持在任意位置快速插入和删除元素==

那么，什么数据结构同时符合上述条件呢？哈希表查找快，但是数据无固定顺序；链表有顺序之分，插入删除快，但是查找慢。所以结合一下，形成一种新的数据结构：哈希链表 LinkedHashMap。

LRU 缓存算法的核心数据结构就是哈希链表，双向链表和哈希表的结合体。这个数据结构长这样：

![img](image/1647580694-NAtygG-4.jpg)

==之所以要使用双向链表，是因为链表删除或者添加元素需要同时知道首尾元素。==

## 答案

### 1. 使用 C++ List

尤其注意：

1. `std::list` 中同时保存了哈希表的 key 和 value
2. 注意 `splice()` 和 `back()` 函数的使用

```c++
class LRUCache {
private:
    int capacity_;

    // C++11后是双向链表
    // 头部是最新使用的 key, 尾部是最久未使用的 key
    std::list<std::pair<int, int>> cache_;

    // 根据 key 找到对应的链表结点
    std::unordered_map<int, std::list<std::pair<int, int>>::iterator> key2node_;

public:
    LRUCache(int capacity) : capacity_(capacity) {}
    
    int get(int key) {
        // 1) 不存在该 key 返回 -1
        auto iter = key2node_.find(key);
        if (iter == key2node_.end()) {
            return -1;
        }
        // 2) 存在该 key 时, 将其移动到链表头部
        cache_.splice(cache_.begin(), cache_, iter->second);
        return iter->second->second;
    }
    
    void put(int key, int value) {
        // 1) 这个 key 已经存在, 则更新值并把该值移动到最前面
        auto iter = key2node_.find(key);
        if (iter != key2node_.end()) {
            iter->second->second = value;
            cache_.splice(cache_.begin(), cache_, iter->second);
            return;
        }

        // 2) 值不存在时将 key 添加到链表头部
        cache_.push_front({key, value});
        key2node_[key] = cache_.begin();
        // 如果超出 capacity_ 则移除尾部元素
        if (cache_.size() > capacity_) {
            key2node_.erase(cache_.back().first);
            cache_.pop_back();
        }
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

### 2. 自定义双向链表结构

```c++
// 双向链表
struct DLinkedNode {
    int key;
    int value;
    DLinkedNode* prev;
    DLinkedNode* next;
    DLinkedNode() : key(0), value(0), prev(nullptr), next(nullptr) {}
    DLinkedNode(int k, int v) : key(k), value(v), prev(nullptr), next(nullptr) {}
};

class LRUCache {
private:
    unordered_map<int, DLinkedNode*> cache;
    // 两个dummy节点构造双向链表
    DLinkedNode* head;
    DLinkedNode* tail;
    int capacity;

public:
    LRUCache(int c) : capacity(c) {
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head->next = tail;
        tail->prev = head;
    }
    
    int get(int key) {
        if (!cache.count(key)) {
            return -1;
        }

        // 如果key存在, 先通过hash表定位, 再移动到头部
        auto node = cache[key];
        // 移动到头部就是先从链表删掉该节点
        node->prev->next = node->next;
        node->next->prev = node->prev;
        // 然后加到链表头部
        head->next->prev = node;
        node->next = head->next;
        head->next = node;
        node->prev = head;

        return node->value;
    }
    
    void put(int key, int value) {
        // printf("put key:%d value:%d\n", key, value);
        // 如果数据已经存在就更新值, 并把元素移动到头部
        auto iter = cache.find(key);
        if (iter != cache.end()) {
            iter->second->value = value;

            auto node = iter->second;
            node->prev->next = node->next;
            node->next->prev = node->prev;
            head->next->prev = node;
            node->next = head->next;
            head->next = node;
            node->prev = head;
            // printf("old value, move to head\n");
            return;
        }

        // 如果数据不存在那么向缓存中插入该数组
        // 1) 插入到缓存中
        auto node = new DLinkedNode(key, value);
        cache[key] = node;
        // 2) 将节点插入到双向链表首部
        head->next->prev = node;
        node->next = head->next;
        head->next = node;
        node->prev = head;
        // printf("new value, move to head\n");

        // 如果此时缓存超过capacity, 那么需要丢掉尾部
        if (cache.size() > capacity) {
            // 找到尾部值对应的key, 删掉缓存
            int key = tail->prev->key;
            // printf("oversize, delete key:%d\n", key);
            cache.erase(key);

            // 删掉尾部结点
            tail->prev->prev->next = tail;
            tail->prev = tail->prev->prev;
        }
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

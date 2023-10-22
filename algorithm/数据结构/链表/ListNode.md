# 算法题链表常用操作

## 定义

```c++
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
 };
```

## 获取链表长度

```c++
// 获取链表长度, 遍历结束后head为nullptr, 所以需要封在函数中
int get_list_len(ListNode* head) {
    int len = 0;
    while (head) {
        ++len;
        head = head->next;
    }
    return len;
}
```

## 常用操作

### 1. 构造哑结点

```c++
ListNode *dummy = new ListNode();
ListNode *cur_node = dummy;

// 在cur_node后拼接节点

ListNode *ans = dummy->next;
delete dummy;
return ans;
```

### 2. 需要保证cur_node最后一个节点next指针为nullptr

```c++
// 当cur_node更新完毕之后
cur_node->next = nullptr;
```

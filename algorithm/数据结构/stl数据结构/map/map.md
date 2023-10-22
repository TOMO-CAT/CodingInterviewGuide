# 算法题常用 map 操作

## 基础操作

```c++
#include <map>
#include <string>
#include <iostream>

int main() {
    // 声明
    std::map<std::string, int> str2int;

    // 列表初始化
    str2int = {{"tomo", 1}, {"cat", 2}};

    // 查找元素是否存在
    if (str2int.find("tomo") != str2int.end()) {
        printf("key:tomo, value:%d\n", str2int["tomo"]);
    }
    
    // 清空map
    str2int.clear();
    
    // 获取键值对数量
    str2int.size();
    
    // 判断key对应值的数量, 非multi map只能是0或者1
    str2int.count("tomo");
}
```

## 删除 key

和 set 一样是 erase：

```c++
m.erase(key);
```

## 返回 map 最后一个元素

```c++
auto iter = m.end();
iter--;
```

## 遍历 map

第一种方式需要 C++11 支持，而且 iter 是值语义：

```c++
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_map<int, int> m = { {1, 2}, {3, 4} };

    // 遍历方式1
    std::cout << "==========================遍历方式1==========================" << std::endl;
    for (const auto& iter : m) {
        std::cout << "key: " << iter.first << std::endl;
        std::cout << "value: " << iter.second << std::endl;
    }

    // 遍历方式2
    std::cout << "==========================遍历方式2==========================" << std::endl;
    for (auto iter = m.begin(); iter != m.end(); iter++) {
        std::cout << "key: " << iter->first << std::endl;
        std::cout << "value: " << iter->second << std::endl;
    }

    return 0;
}
```

输出：

```bash
==========================遍历方式1==========================
key: 3
value: 4
key: 1
value: 2
==========================遍历方式2==========================
key: 3
value: 4
key: 1
value: 2
```

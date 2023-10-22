# string

## 删除元素

```c++
string str;
str.erase(10);    // 从下标为10处全部删除
str.erase(5, 1);  // 从下标为5处开始删除, 删除长度为1
```

## 获取子串

```c++
string str;
str.substr(1, 3);  // 从下标为1处开始获取子串, 子串长度为3
```

## 特殊操作

### 1. 判断字符是否是数字 0~9

```c++
if (c - '0' >= 0 && c - '0' <= 9) {
    // …
}
```

## 数值转换

```c++
// int转化为string
str = std::to_string(int);

// string转成int
i = std::stoi(str);

// string转成long
l = std::stol(str);
```

## 用字符串实现数字比较

注意如果出现了整数溢出的问题，理论上用字符串比较即可：

```c++
int a = 999999998;
int b = 999999997;

// 比较 999999997999999998 大还是 999999998999999997 大
return std::to_string(a) + std::to_string(b) > std::to_string(b) + std::to_string(a)
```

# 标准库function类型

## 可调用对象与调用形式

C++语言中有几种可调用的对象：函数、函数指针、`lambda`表达式、`bind`创建的对象以及重载了函数调用运算符的类。可调用的对象和其他对象一样都有类型（比如每个`lambda`有它自己唯一的（未命名）类类型，函数及函数指针的类型则由其返回值类型和实参类型决定），然而两个不同类型的可调用对象缺可能共享同一种调用形式`call signature`。

调用形式指明了调用返回的类型以及传递给调用的实参类型，一种调用形式对应一个函数类型，例如：

```c++
// 函数类型: 接受两个int并返回一个int
int(int, int);
```

下面三种不同的可调用对象都实现了该函数类型：

```c++
// 普通函数
int add(int i, int j) { return i + j; }
// lambda表达式
auto mod = [](int i, int j) { return i % j; };
// 重载了函数调用运算符的类
struct divide {
    int operator()(int denominator, int divisor) {
        return denominator / divisor;
    }
};
```

## 标准库function类型

`function`定义在`functional`头文件中，可以将可调用对象按照调用形式进行分类。

### 1. 操作

| 操作                     | 功能                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `function<T> f`          | `f`是一个用来存储可调用对象的空`function`，这些可调用对象的调用形式应该与函数类型`T`相同 |
| `function<T> f(nullptr)` | 显式地构造一个空`funciton`                                   |
| `function<T> f(obj)`     | 在`f`中存储可调用对象`obj`的副本                             |
| `f`                      | 将`f`作为条件：当`f`含有一个可调用对象时为真；否则为假       |
| `f(args)`                | 调用`f`中的对象，参数是`args`                                |

### 2. 类型成员

| 类型成员                                                     | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `result_type`                                                | 该`function`类型的可调用对象返回的类型                       |
| `argument_type` `first_argument_type` `second_argument_type` | 当`T`有一个或两个实参时定义的类型，如果`T`只有一个实参，则`argument_type`是该类型的同义词；如果`T`有两个实参，则`first_argument_type`和`second_argument_type`分别代表两个实参的类型 |

### 3. 例子

```c++
#include<iostream>
#include<functional>

// 普通函数
int add(int i, int j) { return i + j; }
// lambda表达式
auto mod = [](int i, int j) { return i % j; };
// 重载了函数调用运算符的类
struct divide {
    int operator()(int denominator, int divisor) {
        return denominator / divisor;
    }
};

int main() {
    std::function<int(int, int)> f1 = add;                                 // 函数指针
    std::function<int(int, int)> f2 = divide();                            // 函数对象类的对象
    std::function<int(int, int)> f3 = [](int i, int j) { return i * j; };  // lambda
    std::cout << f1(4, 2) << std::endl;  // 打印6
    std::cout << f2(4, 2) << std::endl;  // 打印2
    std::cout << f3(4, 2) << std::endl;  // 打印8
}
```

我们把所有可调用对象，包括函数指针、`lambda`或者函数对象都添加到`map`中：

```c++
// 列举了可调用对象与二元运算符对应关系的表格
// 可调用对象需要接收两个int，返回一个int
// 其中的元素可以是函数指针、函数对象或者lambda
std::map<std::string, std::function<int(int, int)>> binops = {
    {"+", add},                                // 函数指针
    {"-", std::minus<int>()},                  // 标准库函数对象
    {"/", divide()},                           // 用户定义的函数对象
    {"*", [](int i, int j) {return i * j;} },  // 未命名的lambda
    {"%", mod}};                               // 命名了的mod对象

// 调用方式
binops["+"](10, 5);  // 调用add(10, 5)
binops["-"](10, 5);  // 调用minus<int>对象的调用运算符
binops["/"](10, 5);  // 调用divide对象的调用运算符
binops["*"](10, 5);  // 调用lambda对象
binops["%"](10, 5);  // 调用lambda对象
```

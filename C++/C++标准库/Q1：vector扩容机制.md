# C++面试：vector 扩容机制

## 1. vector 添加元素时的扩容过程

### 1.1 问题

请问下面这串代码的输出内容是什么？

```c++
#include <vector>
#include <iostream>

struct Cat {
    // 默认构造函数
    Cat() {
        printf("Cat()\n");
    }

    // 析构函数
    ~Cat() {
        printf("~Cat()\n");
    }

    // 拷贝构造函数
    Cat(const Cat &cat) {
        printf("Cat(const Cat &cat)\n");
    }

    // 移动构造函数
    Cat(Cat && cat) {
        printf("Cat(Cat && cat)\n");
    }

    // 拷贝赋值运算符
    Cat& operator=(const Cat &cat) {
        printf("Cat& operator=(const Cat &cat)\n");
        return *this;
    }

    // 移动赋值运算符
    Cat& operator=(Cat &&cat) {
        printf("Cat& operator=(Cat &&cat)\n");
        return *this;
    }
};

int main(void) {
    std::vector<Cat> cat_list;
    cat_list.reserve(2);
    for (int i = 0; i < 4; i++) {
        printf("====================%d====================\n", i);
        cat_list.push_back(Cat());
    }

    printf("====================done====================\n");
    return 0;
}


```

### 1.2 答案

输出内容如下：

```bash
====================0====================
Cat()
Cat(Cat && cat)
~Cat()
====================1====================
Cat()
Cat(Cat && cat)
~Cat()
====================2====================
Cat()
Cat(Cat && cat)
Cat(const Cat &cat)
Cat(const Cat &cat)
~Cat()
~Cat()
~Cat()
====================3====================
Cat()
Cat(Cat && cat)
~Cat()
====================done====================
~Cat()
~Cat()
~Cat()
~Cat()
```

解析：

```bash
# 第一次 push
Cat()            # 1) 调用默认构造函数创建临时 Cat 对象
Cat(Cat && cat)  # 2) 移动构造函数插入临时对象到 vector 中
~Cat()           # 3) 析构第一次产生的临时 Cat 对象

# 第二次 push: 同上
Cat()
Cat(Cat && cat)
~Cat()

# 第三次 push
Cat()                # 1) emplayce_back()函数内调用默认构造函数在栈上创建临时 Cat 对象      
Cat(Cat && cat)      # 2) 触发 vector 扩容，调用移动构造函数插入临时对象到 vector 中最后一个位置
Cat(const Cat &cat)  # 3) 调用拷贝构造函数复制原有的缓存
Cat(const Cat &cat)  # 4) 同上
~Cat()               # 5) 析构原有的第一个元素
~Cat()               # 6) 析构原有的第二个元素
~Cat()               # 7) 析构第三次塞的临时变量

# 第四次 push: 同第一次
====================3====================
Cat()
Cat(Cat && cat)
~Cat()

# 循环结束到 main 函数退出
====================done====================
~Cat()  # 1) 析构第一个元素
~Cat()
~Cat()
~Cat()
```

细节：

> Q1：析构时的析构顺序？
>
> A1：
>
>
>
> Q2：扩容期间复制缓存为什么使用拷贝构造函数而不使用效率更高的移动构造函数？
>
> A2：标准库要求`vector`的`push_back`函数是强异常保证的（若函数抛出异常，则程序的状态会恰好被回滚到该函数调用之前的状态），移动构造函数将对象从旧内存移动到新内存中抛出异常时我们无法保证`vector`能回退到扩容前的状态。如果我们给移动构造函数加上`noexcept`声明保证其不会抛出异常，那么扩容期间复制缓存就会调用效率更高的移动构造函数。

### 1.3 变式题：push 与 emplace 的差别

将前面代码中的`push_back()`修改成`emplace_back()`：

```c++
#include <vector>
#include <iostream>

struct Cat {
    // 默认构造函数
    Cat() {
        printf("Cat()\n");
    }

    // 析构函数
    ~Cat() {
        printf("~Cat()\n");
    }

    // 拷贝构造函数
    Cat(const Cat &cat) {
        printf("Cat(const Cat &cat)\n");
    }

    // 移动构造函数
    Cat(Cat && cat) {
        printf("Cat(Cat && cat)\n");
    }

    // 拷贝赋值运算符
    Cat& operator=(const Cat &cat) {
        printf("Cat& operator=(const Cat &cat)\n");
        return *this;
    }

    // 移动赋值运算符
    Cat& operator=(Cat &&cat) {
        printf("Cat& operator=(Cat &&cat)\n");
        return *this;
    }
};

int main(void) {
    std::vector<Cat> cat_list;
    cat_list.reserve(2);
    for (int i = 0; i < 4; i++) {
        printf("====================%d====================\n", i);
        cat_list.emplace_back();
    }

    printf("====================done====================\n");
    return 0;
}
```

输出：

```c++
====================0====================
Cat()
====================1====================
Cat()
====================2====================
Cat()
Cat(const Cat &cat)
Cat(const Cat &cat)
~Cat()
~Cat()
====================3====================
Cat()
====================done====================
~Cat()
~Cat()
~Cat()
~Cat()
```

解析：

```bash
# 第一次 push: 直接在 vector 的堆上内存原地构造对象
Cat()

# 第二次 push: 同上
Cat()

# 第三次 push
Cat()                # 1) 触发 vector 扩容申请新的堆内存: 先在 cat_list[2]上原地构造第三个元素
Cat(const Cat &cat)  # 2) 调用拷贝构造函数复制原有的缓存
Cat(const Cat &cat)  # 3) 同上
~Cat()               # 4) 析构原有的第一个元素
~Cat()               # 5) 析构原有的第二个元素

# 第四次 push: 同第一次
Cat()

# 循环结束到 main 函数退出
~Cat()  # 1) 析构第一个元素
~Cat()
~Cat()
~Cat()
```

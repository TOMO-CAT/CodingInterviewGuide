# extern

## 功能

修饰符`extern`用在变量和函数的声明前，表示“此变量/函数是在别处定义的”，提示编译器在遇到此函数或者变量时在其他模块中寻找它们的定义。

### 1. extern 修饰变量

> `extern`修饰变量时经常用于：定义全局变量时，在`.h`文件中通过`extern`声明，然后在`.cpp`中定义它。

如果想声明一个变量而非定义它，就使用关键字`extern`并且不要显式地初始化变量：

```c++
/*
 * 在所有函数外声明/定义全局变量 a
 * */

extern int a;      // 声明一个全局变量
int a;             // 定义一个全局变量
extern int a = 0;  // 定义一个全局变量并初始化, 抵消了 extern 的作用
int a = 0;         // 定义一个全局变量并初始化
```

声明而非定义全局变量时必须加 extern 关键字修饰，否则即使没有显式的初始化它也会被加载进 BSS 区初始化为 0。

### 2. extern 修饰函数

对于函数而言，声明和定义本来就是有区别的（定义函数要有函数体，声明函数不需要函数体），因此声明函数时 extern 关键字是可有可无的（或者函数声明本身就是 extern 的）。

假设我们在`foo.h`中声明了`foo_func()`函数，然后在`foo.cpp`中定义它，当我们想要在`bar.cpp`中引用该函数时，使用`extern`声明和直接`#include foo.h`还是有一些区别的。

假设我们有`foo.h`和`foo.cpp`文件，并且在其中定义了`foo()`等函数：

```c++
/*
 * foo.h
 */
#ifndef FOO_H
#define FOO_H

void foo();

#endif

/*
 * foo.cpp
 */
#include <iostream>

void foo() {
    std::cout << "foo()" << std::endl;
}

```

#### 2.1 使用#include 引入函数声明

```c++

/*
 * main.cpp
 */
#include "foo.h"

int main() {
    foo();
    return 0;
}
```

#### 2.2 使用 extern 引入函数声明

在前面的例子中，我们在`mian.cpp`中`#include "foo.h"`从而可以使用声明在`foo.h`中的函数。但是这样的做法会将`foo.h`中所有的声明都引入到`main.cpp`中，假设我们只需要`foo()`函数就显得很没有必要。我们可以通过在函数声明前加上`extern`表明这个函数是定义在其他`.cpp`文件中的，这样一方面可以使得代码更佳清晰简洁，另一方面也会加快编译期间预处理的速度。

```c++
extern void foo();

int main() {
    foo();
    return 0;
}
```

## 重定义问题

C++中重定义错误包含如下两种情况：

* 情况一：多个源文件包含了同个头文件，且头文件中包含了某个全局变量或者非内联函数的定义
* 情况二：某个源文件多次包含同一个头文件

对于情况一而言，我们应当尽量避免在头文件中定义全局变量或者非内联函数，全局变量可以在头文件中用 extern 声明然后在源文件中定义，函数定义可以加上 inline 关键字变成内联函数。由于编译器会将类、内联函数以及 const 变量默认视为定义它们的源文件所私有，因此这些类型可以定义在头文件中。

对于情况二而言，我们可以在头文件中添加头文件保护符：

```c++
// 法一
#ifndef FOO_H
#define FOO_H
/* foo.h 的内容 */
#endif

// 法二: Google 编码规范不推荐
#pragma once
```

## Reference

[1] <https://www.cnblogs.com/renyuan/archive/2012/11/30/2796801.html>

[2] <https://zhuanlan.zhihu.com/p/141833130>

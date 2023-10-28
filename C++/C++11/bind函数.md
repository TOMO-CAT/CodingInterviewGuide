# bind函数

## 简介

标准库`bind`函数定义在头文件`functional`中，我们可以将`bind`看做是一个通用的函数适配器：**它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表**：

```c++
/*
 * 功能: 当我们调用newCallable时, 相当于调用callbale, 并传递给它arg_list指定的参数
 *
 * newCallable: 新生成的可调用对象
 * callable: 原有的可调用对象
 * arg_list: 逗号分隔的参数列表, _n表示newCallable中第n个参数
 */
auto newCallable = bind(callable, arg_list);
```

## bind的参数

假设`f`是一个有`5`个参数的可调用对象，`g`是一个有`2`的参数的可调用对象，我们构造下述`bind`的调用：

```c++
using namespace std::placeholders;
auto g = bind(f, a, b, _2, c, _1);
```

- 新生成的`g`可调用对象有两个参数，分别用占位符`_2`和`_1`表示
- 新的可调用对象`g`将它自己的参数作为第三个和第五个参数传递给`f`
- `f`的第一个、第二个和第四个参数分别被绑定到给定的值`a`、`b`和`c`上

总结一下：`bind`调用会将`g(_1, _2)`映射成`f(a, b, _2, c, _1)`，通过占位符我们也可以**重排参数顺序**。

## 绑定引用参数

默认情况下`bind`那些不是占位符的参数被拷贝到`bind`返回的可调用对象中。与`lambda`表达式类似，有时候我们希望以引用的方式传递绑定的参数，或是绑定的参数无法拷贝，比如构造一个以引用方式捕获`ostream`的`lambda`：

```c++
// os是一个局部变量, 引用一个输出流
// c是一个局部变量, 类型为char
for_each(words.begin(), words.end(), 
    [&os, c](const string &s) { os << s << c; });
```

我们可以编写一个函数完成这个功能：

```c++
ostream &print(ostream &os, const string &s, char c) {
    return os << s << c;
}
```

不过需要注意`bind`不能拷贝一个`ostream`，当我们希望传递给`bind`一个对象而又不拷贝它，就必须用标准库函数`ref`：

```c++
// 错误用法：os无法被拷贝
for_each(words.begin(), words.end(), bind(print, os, _1, ' '));

// 正确用法：使用ref
for_each(words.begin(), words.end(), bind(print, ref(os), _1, ' '));

```

> Tips：标准库中还有一个`cref`函数可以生成一个保存`const`引用的类。

# 什么时候使用 virtual 析构函数

## 答案

* 类的设计目的如果不是为了作为基类使用，或不是为了具备多态性，就不应该声明 virtual 析构函数。因为这会导致对象携带 vptr 等额外信息。
* 任何类只要带有 virtual 函数几乎确定也应该有一个 virtual 析构函数。否则通过基类指针 delete 派生类对象时可能出现资源泄漏问题。

## 解析

C++ 中明确指出，当派生类对象经由一个基类指针 delete 且基类中带有 non-virtual 析构函数，其结果是未定义的（实际执行时通常是对象的 derived 成分没被销毁）。

消除这个问题的做法很简单：给基类定义一个 virtual 析构函数。

另外如果一个类设计目的不是作为 base class 使用时，令其析构函数为 virtual 往往是一个馊主意。这是因为如果要实现 virtual 函数，对象必须携带某些信息（virtual table pointer，vptr）以用于在运行期决定哪一个 virtual 函数应该被调用，这会导致对象体积增加。

即使一个类完全不带 virtual 函数，但也可能遇到“non-virtual 析构函数”问题。如果你企图继承一个 STL 容器或其他带有“non-virtual 析构函数”的类，那么使用基类指针 delete 派生类对象时就可能出现资源泄漏等问题。

需要注意的是 Uncopyable 和标准库的`input_iterator_tag`这些类的设计目的是作为基类使用，但是不是为了多态用途，因此它们不需要 virtual 析构函数。

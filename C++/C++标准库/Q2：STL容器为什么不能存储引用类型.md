# STL 为什么不能存储引用类型

## 例子

假如我们在 STL 类型（比如`std::vector`）中存储引用类型，那么会报编译错误：

```c++
#include <vector>

int main() {
    std::vector<int&> vi;
    return 0;
}
```

编译运行：

```bash
$g++ -g test.cpp -o test
In file included from /usr/include/c++/4.8.2/x86_64-redhat-linux/bits/c++allocator.h:33:0,
                 from /usr/include/c++/4.8.2/bits/allocator.h:46,
                 from /usr/include/c++/4.8.2/vector:61,
                 from test.cpp:1:
/usr/include/c++/4.8.2/ext/new_allocator.h: In instantiation of ‘class __gnu_cxx::new_allocator<int&>’:
/usr/include/c++/4.8.2/bits/allocator.h:92:11:   required from ‘class std::allocator<int&>’
/usr/include/c++/4.8.2/ext/alloc_traits.h:199:53:   required from ‘struct __gnu_cxx::__alloc_traits<std::allocator<int&> >’
/usr/include/c++/4.8.2/bits/stl_vector.h:75:28:   required from ‘struct std::_Vector_base<int&, std::allocator<int&> >’
/usr/include/c++/4.8.2/bits/stl_vector.h:210:11:   required from ‘class std::vector<int&>’
test.cpp:4:23:   required from here
/usr/include/c++/4.8.2/ext/new_allocator.h:63:26: error: forming pointer to reference type ‘int&’
       typedef _Tp*       pointer;
                          ^
/usr/include/c++/4.8.2/ext/new_allocator.h:64:26: error: forming pointer to reference type ‘int&’
       typedef const _Tp* const_pointer;
...
```

## C++不能声明或定义引用的指针

C++中不允许声明或定义指向引用的指针，这是从语法层面规定的，原因如下：

* 从设计目的上来讲，设计引用的目的是为了简化指针操作，避免指针造成的一系列问题，如果对引用再取地址的话，有违背设计初衷之意。
* 从实现角度来讲，引用常常为指针常量，对指针常量取地址（指向指针常量的二重指针）本身便没有意义。C/C++ 中定义二重指针往往是要对一重指针进行修改，而指针常量本身就不能修改，且倘若要访问引用所指变量的地址或者访问引用所指变量本身，直接可以通过引用完成，何必大费周折。

```c++
#include <iostream>

int main() {
    int value = 99;
    int& ref = value;  // ref 本质为指针常量(即指针的值是一个常量, 在定义时必须被初始化)

    // int&* ptr = ref;  // 错误: C++中不允许定义指向引用的指针

    std::cout << "value address by reference = " << &ref << std::endl;  // 编译器将其优化为&(*ref)
    std::cout << "value address = " << &value << std::endl;
    std::cout << "value = " << ref << std::endl;  // 编译器将引用优化为(*ref)
    return 0;
}
```

编译运行：

```bash
$g++ -g test.cpp -o test
$./test 
value address by reference = 0x7ffe75480434
value address = 0x7ffe75480434
value = 99
```

## STL 容器中不能存储类型

以`std::vector`为例，其简化代码如下：

```c++
template<typename T, typename Alloc = alloc>
class vector
{
public: 
   // traits
   using value_type = T;
   using pointer = value_type*; // notice
   using iterator = value_type*; // notice
   using reference = value_type&;
   using size_type = size_t;
   using difference_type = ptrdiff_t;

private:
   iterator start;
   iterator finish;
   iterator end;

   // otehr code
};
```

当定义一个存储引用的`std::vector`时，例如：

```c++
vector<int&> vi;
```

根据传入`std::vector`模板中的参数为`int&`，因此`T`和`value_type`为`int&`，`pointer`和`iterator`就会变成`int&*`，当用`iterator`声明`start/finish/end`时，就会报错。因为 C++中不允许声明或定义指向引用的指针。

## Reference

[1] <https://blog.csdn.net/qq_38988226/article/details/110733006>

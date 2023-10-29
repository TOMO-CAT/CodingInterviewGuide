# 栈被破坏的coredump调试

## 代码

```c++
// stack_broken.cpp
#include <stdio.h>
#include <string.h>

void foo(int n) {
    printf("The %d step begin.\n", n);
    int a[10];
    for (int i = 0; i< n; i++) {  // n > 10时会导致写入的空间超过了栈上的“合法”内存
        a[i] = i;
    }
    if (n < 20) {
        foo(n +1);
    }
    printf("The %d step end\n", n);
}

int main(void) {
    foo(8);
    return 0;
}
```

## coredump原因

上述这段代码关键在于`foo()`函数会有递归调用，在`n>10`会导致数组月结，写入空间超过了栈上的合法内存。

清楚如下几点：

* **EBP寄存器和栈上存储的「EBP of Pervious Stack Frame」构成了一个单向链表，链表头是当前执行的函数栈底，其余节点存储了各个层次的调用者函数栈底。**
* 所有栈上地址都是合法的（此处的“合法”指的是操作系统允许写入），也就是说即使是写入到`a[19]`这样的越界地址也是合法的，并不会触发crash。
* 由于栈的增长方向是从高地址向低地址，而**写坏的是“高地址”，即已经申请过的地址空间**。
* 栈上与函数调用最相关的数据是返回地址（Return Address）和上一个栈帧的EBP（EBP of Previous Stack Frame），**而直接导入crash的原因是因为返回地址被写入了脏数据**，导致执行对应的指令出现问题

## 结论

* 栈被破坏导致crash的本质是「Return Address」和「EBP of Previous Stack Frame」的数据被”写坏“导致的。
* 栈上数据错乱不一定会导致crash，有可能是仅仅把应该写入变量a的数据写到了变量b。
* **栈crash时函数的执行已经脱离了出问题的函数**。假设A调用B，然后B函数发生了栈上空间的错误写入，那么crash往往发生在A函数之中。这是因为只有B函数对应的汇编代码最后一句「retq」执行完毕之后才会发生crash，此时程序的控制权还在函数A中。

## 恢复函数调用栈

* **需要指出的是被破坏的函数调用栈部分已经无法恢复了**，我们仅仅能恢复没有被破坏的部分。恢复函数栈的原理也很简单，**那就是根据栈空间中的内存内容，找到那个“rbp链表”即可**。
* `rbp`寄存器是有效的，我们可以根据`rbp`打印出当前栈空间地址

```bash
# 查看rsp寄存器的值, 它原来应该存储foo(19)栈帧的rbp值, 然而被我们越界的数据写坏了
(gdb) info reg rsp
rsp            0x7ffc1f4a6f90   0x7ffc1f4a6f90

# 以rsp为起点, 查看栈空间的数据
(gdb) x/30xg 0x7ffc1f4a6f90
0x7ffc1f4a6f90: 0x0000001100000010   
```

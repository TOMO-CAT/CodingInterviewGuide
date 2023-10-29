# Golang

## 服务死锁

### 1. go1.14.1使用timer导致死锁

**现象**：服务不响应对外接口，上游请求全部超时。而且占用CPU较高（后来验证出现了自旋锁）。

**背景知识**：

* goroutine栈扩容
  * 以前是分段栈，现在是连续栈
  * ==编译器在有明显栈消耗的**函数头部插入一些检测代码**，通过`stackguard0`来决定是否触发`runtime.morestack`函数。将`stackguard0`设置为`StackPreempt`作用是进入函数时必定触发`runtime.morestack`，然后再调用`runtime.newstack`==
* go调度机制
  * **go1.12之前是协作式调度**，goroutine主动让出CPU才能触发调度，这意味着死循环goroutine可以长时间占用线程，导致其他goroutine饥饿。
  * **go1.13是基于协作的抢占式调度**：为goroutine引入了`stackguard0`，依赖调用函数栈增长检测代码的方式实现了基于协作的抢占式调度的工作机制：
    * 编译器会在有明显栈消耗的函数前插入一段代码，比较当前栈寄存器值和`stackguard0`来决定是否触发`morestack`函数
    * ==go运行时代码会在垃圾STW、sysmon监控发现goroutine运行超过10ms时，将当前`stackguard0`字段设置为`StackPreempt`（这个值大于任何实际上寄存器的值）==
    * ==在`morestack`代码中有一段，就是如果`stackguard0`值为`StackPreempt`就调用`runtime.Gosched`出让CPU==
  * **go1.14.1非协作的抢占式调度**：基于信号实现的，也被称为”异步抢占“。
    * M （线程）注册一个 SIGURG 信号的处理函数：sighandler
    * ==sysmon 线程检测到执行时间过长的 goroutine、GC stw 时，会向相应的 M（或者说线程，每个线程对应一个 M）发送 SIGURG 信号==
    * 收到信号后，内核执行 sighandler 函数，通过 pushCall 插入 asyncPreempt 函数调用
    * 回到当前 goroutine 执行 asyncPreempt 函数，通过 mcall 切到 g0 栈执行 gopreempt_m
    * 将当前 goroutine 插入到全局可运行队列，M 则继续寻找其他 goroutine 来运行
    * 被抢占的 goroutine 再次调度过来执行时，会继续原来的执行流
* mcall：
  * ==goroutine主动出让CPU（park或者gosched）时都会调用`runtime.mcall`==
  * `mcall`主要流程是保存当前goroutine的上下文，切换到g0上下文，传入函数参数，跳转到函数代码处执行
  * 与此相对的是`runtime.gogocall`函数用于恢复现场，主要是在`execute`中调用，即在执行goroutine前重新装载对应的寄存器
* goexit：
  * 每个goroutine创建时都会在栈上塞上`runtime.goexit`函数，这样当goroutine执行完之后就会返回到`runtime.goexit`去执行一些清理工作，这也是为什么每个goroutine栈底都是goexit函数的原因
* `runtime.stopTheWorldWithSema`如何STW：
  * ==未运行的P：idle队列或处于系统调用中的P，直接设置状态为`_Pgcstop`来阻止工作线程绑定它们==
  * 正在运行的P：不能简单修改状态，只能通过`preemptall`设置抢占标记来请求它们停下来
* goroutine栈底：
  * `goexit`
  * `mcall`或者`runtime.clone`：主动出让CPU
  * `runtime.morestack`：被信号抢占

**排查过程**：

* top命令查看占用CPU的进程和线程id
* 由于CPU占用很高，先perf和火焰图定位到热点函数，几乎都是`runtime.osyield()`在出让时间片
* `dlv`中通过`goroutine -with running`找到还在运行的goroutine，可以看到栈底是`runtime.mcall`，这说明goroutine正在进行协程切换，`mcall`会保存goroutine的上下文并在g0执行新的函数，一般这个新函数都会执行`schedule()`来挑选新的goroutine执行
* 正在运行的goroutine栈底是`gcBgMarkWorker`，栈顶是`runtime.futex`函数，现在需要找到丢失的从`systemstack_switch`到`runtime.futex`的g0栈信息
* 发现goroutine卡在了`runtime.stopTheWorldWithSema`，即STW 阶段，正在等待所有的P停下来，但是有一个P停不下来，打印出内存中所有的P信息，找到未停下来的P对应M线程
* **问题回到了为什么`preemptall()`发送抢占信号后仍然有P没停下来呢？**
* dlv打印所有P的信息，找到P对应的M地址，找到对应的操作系统线程ID，而且该M正处于执行`runtime.newstack`的g0栈，因此`m.curg == nil`，STW设置抢占信号失败（**g0是不能被抢占的呀**）
* 问题回到了为什么这个线程在卡在了g0栈出不来，而且还在大量占用CPU，猜测是有一个死循环
* 通过dlv命令`thread`切换到该线程并`n`进行单步调试，找到了死循环，有人将`t.status`修改成了`timeModifying`，且其他所有P都被停下来了，自然就会无法更新status

**关键信息**：

* 调用`systemstack_switch`切换到g0后丢失的栈信息如何查。直接就跳到了`runtime.futex`
  * 使用dlv下的`si`单步单核调试，让`futex`执行完返回到caller
  * 但是这个方法并不总是行得通，有时候会出现循环调用的问题，比如`runtime.futexsleep`
  * 解决方法，反汇编停在`runtime.futexsleep`的第一条指令，只要发生函数调用，**rsp寄存器**一定执行函数执行完后的返回地址，打印寄存器信息，再打印出地址就是返回地址，直到找到整个调用链

**死锁原因**

* ==`goroutine1`使用了`t.Reset()`将`t.status`修改成了`timerModifying`，然后在调用`wakeNetPooler`方法时触发goroutine的抢占切换到g0执行`runtime.morestack`。g0会执行`runtime.schedule`进行goroutine的调度，此时需要等到所有timer都修改完成，即（t.status状态不能时timeModifying）==
* 恰好GC的STW请求所有的P停下来，而g0是不能被抢占的，它一直卡在检查`t.status`，而其他所有goroutine都被停止了无法修改`t.status`

**复现bug**”

* go1.14.1
* 大量使用timer

## 内存泄漏

### 1. cgo内存泄漏

**排查过程**：

* 判断是否是协程数量过多导致的协程泄漏
* pprof分析内存使用：内存正常
* golang堆内存分析：没有异常增长
* 查找资料后发现cgo代码申请的堆内存是不会被pprof记录的，因此搜索CGO关键字，发现有个库近期有修复过内存泄漏的问题，更新库后发现内存泄漏问题解决

## 性能优化

### 1. 基于atomic.Value全局配置热更新

**背景**：pprof发现某个配置每个请求都反序列化一个特别大的apollo在线配置的json配置项。

**解决方法**：基于atomic.Value实现配置热更新，异步线程每隔10s热更新一次，所有请求读取对应的缓存配置即可。

**atomic.Value源码阅读**：

* `atomic.Value`就是一个`interface{}`类型
* `Store()`方法：
  * 将`interface{}`解析成`ifaceWords`，它包含一个`type`指针和一个`data`指针
  * 判断`typ`
    * 如果是初始态`nil`，CAS锁将其设置为`^uintptr(0)`中间态，抢锁成功就把旧值原子更新成新值
    * 如果是中间态：忙等到第一次Store完成
    * 如果不是第一次Store则直接调用`StorePointer`进行更新`typ`和`value`即可
* `Load()`方法：
  * `typ`为`nil`或中间态时返回nil，其余调用`LoadPointer`进行加载

> 优势：只有第一次写入加锁了，其他都是指针读写原子操作。

### 2. json库调研

**选择标准**：

* 少用反射，提高性能和降低GC压力
* 提供良好定义的接口，支持反序列化到`map`

### 3. GC调优

* 根本方法：减少对象分配，复用对象
* 降低GC频次：
  * 设置GOGC：存在OOM风险
  * 压舱石：
    * 不会浪费内存（知识再虚拟内存上，没有使用就不会映射到物理内存）
    * 会不会影响GC速度：Mark算法只会遍历所有存货的对象，和存活对象数量相关，和垃圾对象无关
    * 只有频繁GC的才考虑使用压舱石（比如内存占用低但是堆内存大小波动较大）
* 避免GC：
  * 大数组切片
  * CGO申请堆外内存

### 4. go1.12对栈对象精确扫描

**现象**：go1.9.2升级到go1.13.5后发现gc耗时和CPU使用率大幅度上涨。

**排查**：

* pprof发现`runtime.(*lfstack).pop`在go1.13.5中十分耗费CPU资源
* 通过`go tool trace`发现gc时进入长时间的辅助标记`mark assist`
* 查看`runtime.(*lfstack).pop`中每一行代码的时间消耗，发现大部分消耗在CAS方法实现的无锁栈`pop`方法
* **猜想是因为线上高峰期对`runtime.(*lfstack).pop`并发严重，导致改行代码大概率返回false。**
* 找到父函数和对应的commit和解决的issue

**结论**：go1.12引入的栈对象扫描是为了解决栈空间释放不及时的问题。go 1.12前的gc在扫描时如果发现对象是分配在栈上的，就会假定对象一直存活而不去进一步扫描它，从而导致程序OOM。go 1.12后增加了对栈对象的扫描，发现指针变量指向占空间地址后会将该地址放入`ifstack`中，用于后续扫描。

**解决方法**：通过debug遍历控制了栈相关的debug输出，打印出可能存在问题的栈对象指针，处理该部分代码。

**复现**：

* 有较多指向栈对象的指针
* 并发goroutine多（并发进入`runtime.(*lfstack).pop`函数）
* CPU核数较多

## panic

### 1. 并发读写内置map

细节：Fatal错误无法通过recover捕获。

# Go

### 核心特性

goroutine

channel

defer

切片

垃圾回收机制（内置runtime？）

int与int32

interface代替继承多态

字母大小设置可见性

没有模板泛型

所有类型初始化默认为0

反射



### 切片Slice与数组

在底层实现上：

* 切片，是底层实现数组每个数据的引用，自身是结构体，
  * 初始化时，可以不进行长度、容量的声明，长度可以不断增加，但是不会减少
    * 长度为slice的元素个数
    * 容量是slice第一个元素到底层数组的最后一个最后一个元素的距离
  * 长度可变
  * 底层连续地址空间的**引用**、指针
  * 作为函数参数而言，和数组也是不一样的
* 数组，为底层数组的值传递，在for循环的时候会有一个临时对象
  * 初始化的时候，需要确定数组的长度
  * 长度固定
  * **值传递**
  * 作为参数传递的时候，函数参数类型也需要制定数组长度

初始化方式：

数组：

```go
var test [len]type
var test [...]type{value0, value1, ......}
var test [N]type{value0,......,valueN-1}
// 以及上述两种的组合
```

切片：

```go
test := make([]type, len, cap)
test := make([]type, len)
test := []type{}
test := []type{value0, ......}
```



### make与new

* new用于各种类型的内存分配，返回指针，指向对应类型的零值
  * 类型的指针需要分配内存才能赋值，与c中一样
* make只用于给slice、map、channel类（引用类型）分配内存，返回对应的类型，并非指针



### 结构体细节



### 并发编程

#### runtime

go运行的基础设施

* 协调多线程调度、内存分配、GC
* 操作系统、CPU级别操作的封装（信号处理，系统调用、寄存器操作、原子操作）、CGO
* 性能分析等，pprof、trace、race
* 反射实现

与python、java的runtime不同的是，在运行时与用户代码没有明显的界限，一起打包

最主要的功能是两个方面：调度、GC

#### 调度

众所周知，操作系统内部有着线程、进程调度器，当触发阻塞、时间片用尽、硬件中断的时候，都会涉及到切换的问题

Go在此基础上实现了自己的调度器（调度Goroutine，Go内部最基本的执行单元）



##### 线程与协程

线程：共享堆，不共享栈，其切换由操作系统控制

协程：共享堆，不共享栈，切换由程序员显式控制，可以避免上下文切换的额外消耗，可以运行在一个或多个线程上

协程的切换调度在用户空间完成，不涉及到用户空间到内核空间的切换（寄存器切换、内存数据切换、栈切换、安全检查），线程调度里面的taskstructure除了CPU信息之外，还会保存线程的私有栈以及寄存器，上下文会多一点，在POSIX中线程获得了许多进程拥有的功能，这些功能在go的调度中都是用不到的，同时也增加了开销

由于线程拥有协程，而一个线程只在一个CPU上执行，导致协程没有办法利用多核

Goroutine与协程类似，且可以实现并行

Go实现了调度器之后

* 可以自己管理上下文切换等，减少切换成本（协程切换）
* GC的时候需要暂停所有运行的goroutine，使得内存状态一致的时候进行垃圾回收

概念参考以及切换过程：https://blog.csdn.net/heiyeshuwu/article/details/51178268、https://zhuanlan.zhihu.com/p/27328476



##### 调度模型

![preview](https://pic3.zhimg.com/v2-a06db1f245421b17c64d7bc4f338b71e_r.jpg)

在这里P记录了G队列以及对应线程的上下文信息、mcache，这些东西没有强制与M绑定

每个P都有独立的队列的同时，同样存在全局的G队列，会定时进行扫描分配，以防饿死

当前M产生的G会进入当前P的队列中

* 队列的独立可以减少全局队列的锁竞争，提高效率，不会被原子操作等降低效率
* 产生的G进入当前队列，不会进入其他的M，没有上下文切换的代价，提高亲和性
* P与M独立，可以提高mecache的利用率，让阻塞G占用的mecache让出来



##### G状态流转

![preview](https://pic3.zhimg.com/v2-95c62d2ff20b8a75e0ec2eddddaf4bd2_r.jpg)

实际上并没有一个专门的线程或者进程作为一直运行的调度器实体，只有在协程状态变更、m操作等主动变化的时候才会进行调度



##### sysmon协程

类似linux内部的系统线程，作为一个常态协程，执行一些定时任务

![preview](https://pic2.zhimg.com/v2-28f751cb1c56fcc275bc545d7f82d869_r.jpg)

* 在网络交互netpoll中获取ready的协程G
* 通知抢占等
* GC
* debug信息追踪



##### 网络交互

![img](https://pic3.zhimg.com/80/v2-7bed4181eb6bbcba7be4a6b43a604c3e_720w.jpg)



#### GC

三大算法：Mark-Sweep、Mark-Sweep-Compact、Mark-Copy，都需要STW（stop the world）

Go GC特征：三色标记、并发标记和清扫、非分代、非紧缩、混合写屏障

##### 三色标记法

![preview](https://pic2.zhimg.com/v2-652acc8ca5d0f04c455c68d1084cd309_r.jpg)
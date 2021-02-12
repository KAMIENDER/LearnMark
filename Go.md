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

#### Goroutine


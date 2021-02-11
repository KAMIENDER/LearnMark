# C与C++

c++的新功能：模板、面向对象

### inline与defind区别

inline是声明内联函数，在编译的时候会直接把调用该函数的地方直接编译替换掉 ，避免频繁使用栈

defind声明的变量在使用的地方相当于字符串替换



### new、delete、malloc、free

new、delete、free：

* free，只是会进行内存的操作，即释放内存。new会调用构造函数，delete会调用析构函数

* new、delete是c++运算符，会执行对应的构造与析构函数
* malloc、free为库函数，只涉及内存的分配问题

构造函数调用顺序：基类-》子类

析构函数调用顺序：子类-》基类



### 多态、虚函数、纯虚函数

参考：https://zhuanlan.zhihu.com/p/41309205

* 虚函数：声明时前面加virtual，子类可以（或不）进行overwrite
* 纯虚函数：虚函数声明时在结尾加上=0，子类一定要进行overwrite
* 子类和父类都实现某个相同的成员函数的时候，默认调用的是父类的成员函数
* 多态，调用的虚函数取决于指针对象指向的类型，即使向下类型转换后也是一样
* 析构函数最好在基类中声明为虚函数，否则在进行析构的时候只会调用基类的析构函数

函数、虚表是放在一个全局的代码区，为类公有

成员变量、虚表指针是在堆区，属于每一个对象，因此

```c++
int main() 
{
    B bObject;
    A *p = & bObject;
}
```

时，p对象的虚表指针指向的是b的虚表，如果在函数调用中使用这种机制，只有在运行时才能够知道当前对象是什么类，在运行时进行的动态绑定



### epoll

https://zhuanlan.zhihu.com/p/64746509



### 智能指针

https://www.zhihu.com/question/319277442/answer/1517987598

传统指针在堆上分配内存使用之后，整个的生命周期就交给了用户，需要用户在恰当的时候进行内存的释放

智能指针为用户提供了自动销毁机制

使用计数的方式来判断释放的时机，当计数为时进行资源的释放

#### 基础

* 为一个模板类，同时具有指针的基本功能
* 目的是管理堆对象，那些不能够自己释放资源的对象

#### 类别

* auto_ptr

在拷贝、赋值构造的时候，进行对象的传递，原来的智能指针变成null

* unique_ptr

有唯一拥有权，计数永远为1

禁止了复制、赋值语义

使用move函数将一个对象的堆内容转给另外一个

可以指定析构内容，默认是调用对象的析构函数，在初始化指针的时候可以传对应的释放规则进去

* share_ptr

多个智能指针之间可以共享对象，相应的共享的时候，计数值会+1，每个指向该对象的指针析构的时候，计数-1，最后一个对象析构的时候会全部释放

跳出作用域的时候会进行相关对象的析构

赋值、初始化使用到的对象会对应的+1计数

enable_shared_from_this可能会造成循环引用的情况，它会返回一个share_ptr

* weak_ptr

weak_ptr不控制对象的生命周期

不会引起计数器的变化

可以由weak_ptr以及share_ptr进行初始化

不具有普通指针的访问操作功能



### stack overflow

堆栈溢出，在进行递归调用的时候，函数内部的局部变量会一直保存直到这个过程结束



### static

* 隐藏多个文件里面声明的变量，变为非全局变量，只有当前文件可见
* 修饰局部变量的时候，该局部变量只初始化一次，默认值为0，分配在全局数据区



### STL

* vector：底层为数组
* list：底层为链表
* map：红黑树，
* unordered_map：hash table

实现



### 模板

https://www.zhihu.com/question/309183344/answer/576192093



### ++

i++与++i

i++：先返回引用再增加

++i：先增加，再返回引用

i++优先级更高

#### 效率

对于内建类型（int、float、double）而言，效率相同转换成的汇编代码数量、操作命令都相同，只是执行顺序不一样

对于自建类型，包括STL，i++效率更低，++i可以直接返回自己本身，而i++需要返回中间的临时变量（是增加之前的一个备份，右值）



### 左值与右值

所有表达式非左值即右值

* 右值只能放在等号右边，左值则是两者皆可
* 左值可以取地址，代表内存中的某个位置（如变量）
* 右值只是一个值，不能够去取地址，如常量值（数字、字符串）、返回的函数内部的变量（临时变量，在函数结束后会进行释放操作，将值压入栈中返回）

#### 引用折叠

在c++中引用的引用是不合法的，任何含有一个&（左值引用的多次引用）都会坍缩为左值引用，除非多次引用都为&&（右值引用）

#### 左值引用、常引用与右值引用

左值引用：简单地对左值进行获取存储位置的操作（别名引用）

```c++
int a = 22;
int& j = a;
j = 4;
```

常量引用：会在内存中产生右值的临时对象，对该临时对象进行操作

```c++
const int& j = 22;
```

右值引用（在c++11引入）：常量引用无法进行修改等操作，可以通过右值引用的方式进行修改

```c++
int&& j = 22;
j = 4;
```

万能引用：可以接受左值，也可以接受右值

* T类型可以是引用，加上引用折叠，万能引用可以是左值引用，也可以是右值引用
  * 传入右值引用，T推导为MyClass
  * 传入左值引用，T推导为MyClass&（运用引用折叠）
  * 在内部进行调用其他函数的时候，以上两种情况，都会作为左值引用（左值）的方式进行传参调用，如果强制转为右值引用，又只会调用右值对应的传参
  * 当需要根据传进来的引用类型进行函数调用匹配的时候，可以使用完美转发

```c++
#include <iostream>
using namespace std;

template<typename T>
void func(T&& val)
{
    cout << val << endl;
}

int main()
{
    int year = 2020;
    func(year);
    func(2020);
    return 0;
}
```

#### 完美转发

```c++
void print(const int& t)  //左值版本
{
    cout <<"void print(const int& t)" << endl;
}

void print(int&& t)     //右值版本
{
    cout << "void print(int&& t)" << endl;
}

template<typename T>
void func(T&& val)
{
    print(std:forward<T> val);
}
```

实际上也是一个强制类型转换的过程，将val转换为T类型，相当于

```c++
static_cast<T&& &&>(val)
```

参考：https://www.cnblogs.com/5iedu/p/11324772.html

#### std::move()函数

（c++11）：将左值、右值转为右值引用，内部以强制类型转换的方式进行实现，以用于移动语义。

三目运算符根据内部返回类型决定是否是左值函数



#### 默认的几种类函数

样例类：

```c++
class MyClass{
public:
    MyClass(): chars(nullptr), ints(nullptr), doubles(nullptr);
    ~MyClass(){
        if(this.chars != nullptr)free(chars);
        if(this.ints != nullptr)free(ints);
        if(this.doubles != nullptr)free(doubles);
    }
private:
    char *chars;
    int *ints;
    double *doubles;
}
```

包括构造函数、拷贝构造函数、拷贝赋值函数、移动赋值函数、移动构造函数、析构函数

参数传递是调用拷贝构造函数

移动语义的作用：避免额外内存的创建，减少不必要的拷贝操作

这里主要对拷贝构造函数、拷贝赋值函数、移动赋值函数以及移动构造函数进行分析

* 移动构造函数 
  * 将参数中对象的数据转移给自己，并且解除参数中对象与其数据的关系
  * 不会进行内存的操作，只是进行简单的资源转移，一般是指针的转移
  * 转移后原本的对象并没有销毁，销毁的时候还是调用对应的析构函数

```c++
MyClass(MyClass&& item): chars(item.chars), ints(item.ints), doubles(item.doubles){
    item.chars = item.ints = item.doubles = nullptr;
};
```

* 拷贝构造函数

```c++
MyClass(const MyClass& item): chars(item.chars), ints(item.ints), doubles(item.doubles);
```

当两者中有一者不存在的时候，移动构造函数可以降级为拷贝构造函数，而拷贝构造函数不可降级（即隐式删除拷贝构造函数）。

* 移动赋值函数

```c++
MyClass& operator=(MyClass&& item){return *this};
```

* 拷贝赋值函数

```c++
MyClass& oprator=(const MyClass& item){return *this};
```

参考：

http://www.008ct.top/blog/2020/05/05/C-11%E5%9C%A8%E5%B7%A6%E5%80%BC%E5%BC%95%E7%94%A8%E7%9A%84%E5%9F%BA%E7%A1%80%E4%B8%8A%E5%A2%9E%E5%8A%A0%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8/

https://blog.csdn.net/qq_41453285/article/details/104419356



### C++11特性


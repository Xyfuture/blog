---
title: "cpp相关知识复习"
date: 2022-08-20
draft: false
---

# Linux 进程分段

# global & static variable

> [c++ - When are static and global variables initialized? - Stack Overflow](https://stackoverflow.com/questions/17783210/when-are-static-and-global-variables-initialized)

全局变量和static变量都拥有static lifetime(静态的生命周期，生命周期同整个进程相同,这些变量可以被称为static objects)，他们在初始化时有一些性质。在cpp中针对global或者static变量有3种初始化的方式。这3种方式分别是 1. Zero initialization 2. Static initialization 3. Dynamic initialization。

initialization的过程是在main函数执行前就确定的，在cpp中，我们是允许在main函数之前执行一些代码的(貌似c不支持这个feature)。首先只声明的变量做zero initialization，被放置到bss段(数据全为0的一个段)，然后针对赋常量的static objects由编译器进行static initialization处理，放置在data段(放已初始化static objects)，这个过程是离线完成的，data段里直接存着值。最后对于需要执行一段代码才能初始化的static objects，进行dynamic initialization，其放置到bss段，然后在main函数之前执行初始化程序。

多个文件中可以独立设置自己的gobal variable，这些global variable仅能被文件内的函数访问，尽管他们确实在bss段或者data段，此外global variable在多个文件中不能重名，他们是strong symbol，可以使用external关键词引入外部文件的global variable。

不同文件中global variable会在link阶段汇入程序，但是目前不能保证global variable 在dynamic initialization时的顺序，因此可能会产生bug，最好不要相互依赖(Static Initialization Order Fiasco)

> 可以看看supersim实现的libfactory，非常骚的操作，通过全局变量来实现注册功能，之后就可以提供工厂函数功能。

# 继承关系

# static

父类中定义的static member在子类中不会被重新分配空间，子类中调用还是修改父类的static member。这个貌似可以用来实现部分实现pytorch中注册进module的效果。



# inline

> [参考链接](https://www.zhihu.com/question/311420548/answer/592941203)

C++ 11之后添加了新的特性，更加方便实现header only的库。

inline最开始的时候主要是提示给编译器用来将该函数进行内联展开提升性能的，但是目前来说内联函数的实现主要依赖于编译器的实现了，有时候对部分函数不加inline照样展开，有时候加了inline也不展开。

现在inline主要用来标识一个弱符号，cpp正常对于一个符号(函数)是不允许多次进行实现的，但是采用inline之后表示这是一个弱符号，就可以多次实现，但最后链接的时候只会使用同一份实现(从多个实现中随机选择一个，实际上一般多个实现都是一样的)，这样就能方便很多header only的库进行实现，因为header only库中有多次的实现的符号也能正常进行链接。

此外template本身是支持弱符号链接的，不需要单独使用inline关键词了。template每次实例后其实都是弱符号的，最后链接的时候从相同类型的实列中随机选择一个实现的二进制代码(实际上都是一样的)。

# template



# const



# 编译链接

> 结合xmake的流程简单记录一下



## 链接的顺序

静态链接的时候是有顺序要求的，遇到一个obj文件时，后从他开始从左到右开始搜索lib文件和obj文件，如果其中有未知符号对应的实现就进行链接操作，这个从左到右的操作就要求我们把依赖其他库的lib或者obj放到前面。下面用个例子说一下。

比如A.obj 依赖a.lib,但是a.lib又依赖b.lib因此编译的时候要把a.lib写在前面

```shell
g++ -la -lb -o a.exe A.obj #不通过，A.obj 依赖a.lib,应该把A.obj放前面
g++ -o a.exe A.obj -lb -la #不通过，a.lib依赖b.lib，应该把a.lib放前面
g++ -o a.exe A.obj -la -lb #通过
```

动态链接一般能自动处理这个事情，不过一般还是主要用静态链接。

在xmake中更改`add_package()`的顺序就可以更改链接时的顺序。或者手动改cmakelist也可以改变链接的顺序。



#  function & bind

> [参考](https://blog.csdn.net/songsong2017/article/details/103810309)

cpp里面增强的函数指针功能。可以把一个函数用指针传出去。

bind则能给一个函数直接绑定上几个参数，比如类里面的this，还能借助placeholder实现参数位置的调换。

直接看几个例子吧

``` c++
#include <functional>

```


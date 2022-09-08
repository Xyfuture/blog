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

父类中定义的static member在子类中不会被重新分别空间，子类中调用还是修改父类的static member。这个貌似可以用来实现部分实现pytorch中注册进module的效果。



# inline

# template

# const

# 编译链接

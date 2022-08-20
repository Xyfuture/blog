---
title: "xmake体验"
date: 2022-08-12
draft: false
---

# 前言

最近要写一个模拟器，要使用cpp作为主要的开发语言，因此需要一个简单易用的构建工具，经过一番寻找之后，最终选定xmake作为最终的构建工具。

# 常见cpp的构建工具

[参考链接](https://www.zhihu.com/question/29025960/answer/608614147)

## GNU tools

GNU提供了许多用于构建的cpp的工具，比如autoconf、autotools，然后配合最原始的make程序完成构建。这种方式之前用过一次，编译arm版本的libtorch时，用的就是这一套流程，但是说实话不是非常好用，因为在x86上使用arm编译器算是交叉编译，autoconf识别的不是很好，加上他提供的脚本里也有错误，因此最终没有完成编译。

这种方式感觉太原始了，不是很推荐在新的项目中使用。

## cmake

cmake 貌似是当前构建cpp大型工程的事实标准，不过我也接触的不算太多，最多添加一下文件，设置一下include路径啥的，更多的内容也不了解。cmake会根据提供的CMakeLists.txt生成用于编译的makefile，算是符合unix的构建思想吧，一个工具只干一件事。

CLion官方对cmake的支持比较友好，默认的构建方式就是cmake，只要有CMakeLists.txt就能实现补全、调试等一系列流程，所以只要其他工具(xmake)能生成CMakeLists.txt就能借助CLion进行开发了

## bazel

[bazel](https://bazel.build/)是google推出的多语言的构建工具，可以用来构建cpp，java等项目。这个是[supersim](https://github.com/ssnetsim/supersim)采用的构建工具，我在模拟器中借鉴了一下这个项目，所以也简单了解一下bazel这个工具。

bazel基于java编写的，这点我其实不太喜欢，因为需要提前弄jdk环境，在新的电脑上是比较麻烦的。构建速度没话说，能充分利用cpu的资源，多线程编译，速度贼快。

bazel主要基于两个文件进行构建，分别是WORKSPACE和BUILD文件，在WORKSPACE中指定各种第三方库及其拉取方式，比如把库挂到github上，然后到时候拉下来，在BUILD文件中指出构建方式，不同的target，各种依赖项啥的，总体来说比较简单易懂。因为人们一般把第三方的库放到github上，因此跑的时候还是挂梯子，让bazel把库拉下来，不知道能不能手动把库拉下来。

bazel有官方的CLion插件，可以提供补全等功能，替代CMakeLists.txt,但是用起来并不是很好用，这个插件不支持远程开发，而且自身也比较简陋，不太好设置，补全等功能也必须完成一次编译后才能进行，希望之后能添加更多的功能。

# xmake使用

[xmake](https://xmake.io/)是国人开发的一种cpp等语言的构建工具。其基于lua编写，因此其配置文件就是一个lua脚本，不过xmake的binary中直接继承了lua解释器，所以也没有啥额外的依赖，不像bazel一样需要额外的java环境。

xmake的配置文件写起来非常简单。

```lua
set_project("CIMsim")

includes("external")

target("lib")
    set_kind("static")
    add_files("src/**/*.cc|src/**/*_TEST.cc","src/**/*.cpp|src/main.cpp|src/test_main.cpp")
    add_includedirs("./src",{public=true})
    add_deps("external_lib")

target("main")
    set_kind("binary")
    add_files("main.cpp")
    add_deps("lib")
```

本质上就是lua脚本，xmake将脚本分为了描述域和脚本域，描述域只能使用xmake指定的函数，主要用来说明构建的方式，脚本域可以执行自己的lua代码，实现更加丰富的构建设定。

xmake也有一个CLion的插件，但是他比bazel的插件还简陋，甚至不支持补全等功能。但是xmake有一个好处就是能直接生成CMakeLists，CLion根据CMakeLists就能实现补全等功能了。通过cmake的桥，CLion也能很方便对使用xmake构建的项目进行调试等操作。

xmake还带有一个官方的包管理工具，可以很方便的引入第三方库，不过这个工具以lib的方式分发，对本模拟器项目来说不是很合适，因此我也没有过多的了解。

> PS:前几天在windows上自动配置cmake有一个关于路径的小bug，刚提issue作者就给改了，效率特别高，很赞。

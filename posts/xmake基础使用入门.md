---
title: "xmake基础使用"
date: 2022-09-06
draft: false
---

# 配置工具链

cpp 跨平台有一个问题不同平台默认使用的编译工具链是不同的，windows一般默认MSVC，Linux默认使用gcc，Macos则一般使用clang+llvm，这几个编译器对同一份代码的编译行为不一定一致，目前遇到的主要是头文件略有不同，cpp标准支持情况不一致。为了方便跨平台还是统一用gcc吧，在windows可以使用mingw(或者用msys2内置的)。

xmake针对不同平台也是使用平台默认支持的编译器(windows-msvc, linux-gcc, macos-clang),我们需要手动替换编译器。

可以在shell中使用`xmake f`改变当前目录下xmake 的配置信息

```shell
xmake f --toolchain=xxx  gcc/mingw/msvc/clang # 设置不同的toolchain
xmake show -l toolchains # 列出目前xmake 支持的所有toolchain类型
```

除了上面的方法还可以在`xmake.lua`中通过`set_config`可以进行设置

```lua
set_config("toolchain","gcc")
```

如此设置是在运行时改变工具链，但是这样在前期检查阶段仍然会使用旧的编译器，包括生成cmake信息等等，**所以仍然推荐上来手动设置一下**。

`set_config`是全局生效的。

# 添加第三方包

# GCC 12 isystem

msys2使用的包管理器是pacman，这个包管理器对于更新比较友好，但是降级确实有点噩梦的感觉，一大堆依赖不好删除。这次手贱把gcc版本从11.2升级到了12.1，出现一个于头文件相关的bug，stdlib中有些奇怪的问题，本来想的是降级一下，结果折腾了半天依赖都没有去掉，然后开始解决这个于头文件相关的bug了。

具体的解决方案 [xmake should not set -stdlib=libc++ when using gcc from homebrew · Issue #2317 · xmake-io/xmake · GitHub](https://github.com/xmake-io/xmake/issues/2317)

有个人也是遇到了同样的问题，然后问了下作者，本质上是因为包含了 `isystem msys2/mingw-w64/include`参数，这导致搜索头文件时顺序会出现问题，在xmake 中引入外部包时把external设置成true即可解决这一问题。

```lua
add_requeires("xxx",{external=true})
```

# 开启debug模式

进行debug一般会在编译过程中添加额外的symbol来进行标记，此外也禁用优化来方便debug，这一般需要在编译时传入额外的参数。在xmake中我们可以在`xmake.lua`中添加`add_relues("mode.debug","mode.release")`来开启调试编译支持。在使用前需要用`xmake f --mode=debug or xmake f -m debug`开始debug模式，否则默认就是release模式。

通过`xmake f`配置后，不仅对当前目录有效，对于第三方package中的`xmake.lua`同样生效，因此我们可以通过一次配置实现开启所有包的debug选项。(仅针对package同样使用xmake进行构建的情况，如果package没有使用xmake构建，需要在引入时使用`add_requires("xxx",{debug=true})`进行设置。



# 常用指令

```shell
xmake c 
xmake c -a
xrepo remove --all
xmake f --toolchain=gcc --mode=debug
xmake project -k cmake -y
```

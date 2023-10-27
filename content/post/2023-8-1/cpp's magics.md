---
title: "C/C++奇技淫巧"
date: 2023-08-01T22:16:58+08:00
draft: true
---

## GCC魔法

**Attribute**：

属性说明符用于描述函数，变量和类型的属性。GCC可以根据这些指定属性做出相应的优化。它的形式为`__attribute__ ((attribute-list))`，其中属性列表是一个可能为空的以逗号分隔的属性序列。最常见的属性包括：

- packed：类内存紧凑排列。
- aligned(n)：所有类成员按n字节对齐。
- constructor：函数先于main函数执行，一般用于共享库加载的初始化。

此外，C++11时推出了属性语法，来保证在不同的编译环境如：GNU GCC和Microsoft MSVC下效果统一。具体语法可以参考[C++手册](https://en.cppreference.com/w/cpp/language/attributes)

**Inline Assembly**：

GCC中提供了C语言和汇编混合编译的功能，具体可以参考[手册](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)。

通过这个功能，结合着co_yield和co_wait可以实现一个简单的协程库，具体可以参考[NJU-OS M2](https://jyywiki.cn/OS/2022/labs/M2.html)。

## SIMD

SIMD(Single Instruction Multiple Data)即单指令流多数据流，是一种采用一个控制器来控制多个处理器，同时对一组数据（又称“数据向量”）中的每一个分别执行相同的操作从而实现空间上的并行性的技术。简单来说就是一个指令能够同时处理多个数据。

![处理器对SIMD的支持](https://pic3.zhimg.com/80/v2-42f84e676fb838f1e2aa2632d4ab0246_720w.webp)

![使用SIMD](https://pic2.zhimg.com/80/v2-a1490b3ffd4a96a09ed17e342d279635_720w.webp)

**Intel Intrinsics**：

## 宏

**Intrusive Programming**：

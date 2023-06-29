---
title: "NJU-OS笔记"
date: 2023-06-19T21:22:58+08:00
draft: true
---

这篇博客流水线的记录一下自己学习[NJU-OS](https://jyywiki.cn/OS/2022/)的笔记。

## 1 操作系统概述

> 本课程讨论狭义的操作系统，即：对单一计算机硬件系统作出抽象、支撑程序执行的软件系统。

### 计算机历史

### 理解操作系统

理解操作系统，就是回答下面三个问题：

- 操作系统服务谁？
  - 操作系统服务应用程序，而程序可以理解成状态机。
  - 注：课程涉及多线程Linux应用程序。
- 站在应用视角，操作系统能为程序提供什么服务？
  - 操作系统 = （可操作的）对象 + （可调用的）API
  - 注：课程只涉及 POSIX + 部分Linux特性。
- 站在硬件视角，如何实现操作系统并提供这些服务？
  - 操作系统 = C程序
  - 注：课程涉及 xv6，并在此基础上实现一个迷你操作系统。

> xv6 是MIT为操作系统的课程，开发的一个教学目的的操作系统。

## 2 操作系统上的程序

### 什么是程序

### 操作系统中的一般程序

> 课后作业中，使用tmux实现多窗口管理这一项让我收获挺多的，具体参见我的[tmux笔记](/)。此外，还有一些WSL环境，终端个性化配置，Vim快捷键以及gdb调试等等小项目，也非常有意义。

## 3 多处理器编程

### 并发

**并发的基本单位——线程**：

共享内存的多个执行流：

- 执行流拥有独立的堆栈/寄存器
- 共享全部的内存（指针可以相互引用）

POSIX Threads

实现原子性

实现临界区之间的绝对串行化
程序其他部分依然并行执行

## 11 操作系统上的进程

### 回顾

**定制Linux**：

BusyBox: The Swiss Army Knife of embedded Linux

adb shell(toybox)

**Linux操作系统启动流程**：

- CPU Reset
- Firmware
- Loader
- Kernel_start
- /bin/init
- 程序执行+系统调用

**应用视角的操作系统**：

操作系统为所有程序提供API

- 进程&线程（状态机）管理
  - fork,execve,exit - 进程的创建、改变和删除
  - pthread_create,pthread_join,pthread_exit - 线程的创建和退出
- 存储（地址空间）管理
  - mmap - 虚拟地址空间管理
- 文件（数据对象）管理
  - open,close,read,write - 文件访问管理
  - mkdir,link,unlink - 目录管理

### fork()

**理解操作系统是状态机管理者**：

- 操作系统在物理内存中保存多个状态机；
- 通过虚拟内存实现，拿一个来执行；
- 中断后进入操作系统代码，换一个执行。

**状态机管理之创建状态机**：

UNIX的答案是fork，即：做一份状态机的完整复制（内存、寄存器现场）。

**理解fork函数**：

`pid_t fork(void)`

在fork函数返回后，会有两个进程同时运行，其中一个是父进程，另一个是新创建的子进程。

在子进程创建成功后，它会复制父进程的代码和数据段，并且开始执行从fork函数返回的位置。也就是说，子进程会从fork函数返回的地方开始执行，而不是从程序的起始位置开始执行。

需要注意的是，在子进程中，fork函数的返回值是0，而在父进程中，fork函数的返回值是新创建的子进程的进程ID。因此，在父进程中可以通过判断fork函数的返回值来区分当前进程是父进程还是子进程。

### execve()

**状态机管理之替换状态机**：

UNIX的答案是execve，即：将当前运行的状态机重置成另一个程序的初始状态。

**理解execve函数**：

`int execve(const char *pathname, char *const argv[], char *const envp[]);`

- pathname：指定要执行的程序的路径，可以是绝对路径或者相对路径。如果路径中不包含目录分隔符 /，则会从环境变量 PATH 中查找可执行文件。
- argv：一个以 NULL 结尾的字符串数组，用于指定新程序的命令行参数。第一个元素通常是程序的名称，后面的元素是程序的参数。注意，第一个元素是程序的名称，而非路径名。
- envp：一个以 NULL 结尾的字符串数组，用于指定新程序的环境变量。每个字符串都是一个形如 key=value 的键值对，表示一个环境变量的值。

execve函数执行成功时，不会返回到调用进程，而是直接将当前进程替换为新程序的进程。如果执行失败，则会返回 -1，并设置相应的错误码。

## 12 进程的地址空间

## 13 系统调用和UNIX Shell

### Shell

**简介**：

Shell 是一种计算机程序，它将操作系统的服务暴露给人类用户或其他程序。

一般来说，操作系统的 Shell 使用命令行界面（CLI，Command Line Interface）或图形用户界面（GUI，Graphics User Interface），具体取决于计算机的角色和特定操作。

Shell Programming Language是一门把用户指令翻译成系统调用的编程语言。

Shell Scripts 是由 Shell Programming Language 编写的脚本文件，它们通常以 .sh 扩展名结尾，包含一系列的 Shell 命令和脚本语句，用于执行特定的操作和任务。

**功能**：

- 重定向: cmd > file < file 2> /dev/null
- 顺序结构: cmd1; cmd2, cmd1 && cmd2, cmd1 || cmd2
- 管道: cmd1 | cmd2
- 预处理: $(), <()
- 变量/环境变量、控制流……

**未来展望**：

- fish、zsh等用户友好、易用且美观的命令行界面
- [tldr](https://tldr.sh/)、[thefuck](https://github.com/nvbn/thefuck)等自动补全工具
- 类似vscode的命令板块（Ctrl-Shift-P）
- [Executable Formal Semantics for the POSIX Shell](https://dl.acm.org/doi/pdf/10.1145/3371111)

### 终端和Job Control

## 14 C标准库的实现

## 15 a fork() in the road

## 16 可执行文件

**状态机的描述**：

操作系统的诞生就是为了给应用程序提供执行环境，而可执行文件就是最重要的操作系统对象！

可执行文件是一个描述了状态机的初始状态 + 迁移的数据结构，包括：

- 寄存器：大部分由 ABI 规定，操作系统负责设置，例如初始的 PC；
- 地址空间：二进制文件 + ABI 共同决定，例如 argv 和 envp (和其他信息) 的存储；
- 其他有用的信息 (例如便于调试和 core dump 的信息)。

**可执行的条件**：

- 具有执行的权限；
  - 可以使用`chmod +x [file names]`为文件增添可执行权限。
- 加载器能识别的可执行文件。

**常见的可执行文件**：

Windows:

- EXE
- UEFI(Unified Extensible Firmware Interface)
- PE(Portable Executable)

UNIX/Linux:

- ELF(Executable Linkable Format)

### 解析可执行文件

**GNU Binutils**：

Binutils是一组开源工具集，用于创建、操作和转换二进制文件。可以按功能将Binutils分成两大部分，分别是

- 帮助生成可执行文件的工具，比如：ld (linker), as (assembler), ar (Archiver)；
- 帮助分析可执行文件的工具，比如：objcopy/objdump/readelf。

****

### 编译和链接

编译器生成文本汇编代码 → 汇编器生成二进制指令序列。

**重定位 Relocation**：

重定位是指将一个程序或库加载到内存中时，将它们在内存中的位置从编译时指定的地址（静态地址）调整为运行时实际的地址的过程。

重定位通常是由操作系统的动态链接器或加载器完成的，它们负责将程序或库加载到内存中，并修复程序或库中的所有引用，以便在运行时正确地执行。

**ELF文件**：

ELF 就是一个 “容器数据结构”，包含了必要的信息。

**重新理解编译、链接流程**：

编译器 (gcc)：High-level semantics (C 状态机) → low-level semantics (汇编)

汇编器 (as)：Low-level semantics → Binary semantics (状态机容器)

- “一一对应” 地翻译成二进制代码
  - sections, symbols, debug info
- 不能决定的要留下 “之后怎么办” 的信息
  - relocations

链接器 (ld)：合并所有容器，得到 “一个完整的状态机”

- ldscript (-Wl,--verbose); 和 C Runtime Objects (CRT) 链接
- missing/duplicate symbol 会出错

## 17 可执行文件的加载

### 静态ELF加载器

**加载器的功能**：

- 解析数据结构 + 复制到内存 + 跳转；
- 创建进程运行时初始状态 (argv, envp, ...)。

**Boot Block Loader**：

Boot Block Loader也是也一个加载器，它能加载操作系统内核。此外，Boot Block Loader通常也称为引导扇区（boot sector），它的大小则一般为512字节。

具体来说，Boot Block Loader加载操作系统的工作流程是：

- 首先，BIOS在启动盘第一个扇区，中查找引导程序，即：Boot Block Loader；
- 接着，BIOS会将该扇区中的512字节加载到内存中的0x7C00地址，并跳转到这个地址开始执行Boot Block Loader中的代码；
- 最后，由Boot Block Loader加载操作系统内核，并跳转到核心代码入口开始执行操作系统。

### 动态链接

**拆解应用程序的必要性**：

随着库函数越来越大，希望项目能够 “运行时链接”。

动态链接能够减少库函数的磁盘和内存拷贝，如果每个可执行文件里都有所有库函数的拷贝那也太浪费了。

## 18 x86代码导读

## 19 实现上下文切换

---
title: "一文了解GNU Tools"
date: 2023-06-27T10:15:12+08:00
draft: false
---

写这篇博客主要是为了记录和总结我学习和使用GNU Tools的经验。

## GNU

**简介**：

GNU是一个自由软件、开源软件项目，旨在创建一个完全自由的操作系统。

GNU这个名字是"Gnu's Not Unix"的递归缩写。这表明GNU是一个类Unix系统，但不是Unix系统本身。

**成果**：

尽管GNU项目没有完全实现自己的操作系统，但它对自由软件和开源软件的发展做出了巨大的贡献，推广了自由软件的概念和实践，鼓励了开源软件的发展。

事实上，由于GNU Hurd内核的开发进展缓慢，GNU项目在1992年宣布计划使用Linux内核作为GNU操作系统的内核，这导致了Linux和GNU的混淆。Linux内核和GNU项目中的系统工具和库结合起来，形成了现代Linux操作系统。

换句话说，GNU项目深刻的影响和改变了Linux操作系统，我们使用的很多Linux命令，例如：ls、cp、rm，其实都是来自GNU项目。因此，仔细了解常用的GNU Tools就显得很有必要了。

总的来说，GNU项目的成果包括两大部分：

- 以GPL开源协议为代表的自由和开源软件实践；
- 以GNU Tools为代表的操作系统工具。

下面，我将会介绍包含GCC、GDB、Make、Coreutils和Bitutils等一众工具的GNU Tools。

## GCC

**简介**：

GCC（GNU Compiler Collection）是一套由GNU项目开发的、支持多种编程语言、多种目标文件和优化选项、具有很高的可移植性和跨平台性的编译器套件。

具体来说，它可以编译多种编程语言的代码，包括C、C++、Objective-C、Fortran、Ada等等，并且可以生成可执行文件、共享库和静态库等多种形式的目标文件。此外，它也支持多种操作系统和硬件平台，包括Linux、Unix、Windows、macOS、ARM、MIPS等等。它还支持多种优化选项，可以生成高效的目标代码，优化程序的性能和大小。

### 使用GCC前你需要知道

在具体介绍gcc的使用方法之前，我想先补充说明一些编译相关概念和流程，以帮助理解gcc的使用方法以及gcc、gas和gld的关系。

**编译流程**：

一般来说，将源文件生成目标文件的流程通常包括以下四个步骤：

- 预处理：在这个阶段，主要是处理头文件包含、宏替换、条件编译等预处理指令，生成一个新的临时文件，通常以.i或者.ii作为扩展名，这个文件包含了所有的预处理结果。
- 编译：在这个阶段，编译器会将预处理后的代码进行编译，生成汇编代码，通常以.s作为扩展名。
- 汇编：在这个阶段，汇编器会将汇编代码转换为机器码，生成目标文件，通常以.o作为扩展名，这个目标文件包含了可执行代码、符号表、调试信息等。
- 链接：在这个阶段，链接器会将目标文件和相关的库文件进行链接，生成可执行程序或者共享库等。链接器会将不同的目标文件组合起来，解决符号引用、重定位等问题，生成最终的可执行程序或共享库。

**编译器**：

编译器是一种将高级语言源代码转换为目标机器可执行代码的程序。编译器通常由两个主要组件组成：前端和后端。其中，前端主要负责将高级语言源代码进行词法分析、语法分析、语义分析等操作，生成中间代码或汇编语言代码；而后端负责将中间代码或汇编语言代码翻译为目标机器的机器代码。

对于gcc来说，它针对每种语言都有相应的前端，例如：C预处理器：cpp、C编译器：cc1、C++预处理器：cpp、C++编译器：cc1plus，但它在默认情况下，只使用gas（GNU Assembler）作为后端，将中间代码转换为机器码。最后使用gld（GNU Linker）将目标文件和相关的库文件进行链接，生成可执行程序或者共享库等。

### GCC常见参数

此处只简单介绍一下gcc常用的一些参数，详细的gcc用法可以参考它的man page或官方手册。

``` bash
# 默认执行编译链接操作，生成可执行文件a.out。
gcc [source files]
gcc test.c
# -o 指定输出文件名
gcc [source files] -o [output files]
gcc test.c -o test

# -E 只执行预处理操作，将源代码转换为预处理后的代码输出到标准输出流中。
gcc -E [source files]
# -S 只执行编译操作，将源代码编译为汇编代码输出到文件中。
gcc -S [source files]
# -c 只执行汇编操作，将源代码编译为目标文件输出。
gcc -c [source files] 

# -g 生成调试信息，以便后续使用调试器进行调试。
gcc -g [source files]
gcc -g test.c -o test

# -w 选项表示关闭所有警告信息的输出。
# -Wall 选项表示开启所有警告信息的输出。
# -Werror选项表示将所有警告信息视为错误.

# -static 选项表示静态链接，即将所有的库文件都链接进可执行文件中。
# -shared 选项表示动态链接，即将库文件链接为共享库。

# -I 指定头文件搜索路径，可以指定多个。
gcc -I [include path] 
gcc -I/usr/include/xxx test.c -o test
# -L 指定库文件搜索路径，可以指定多个。
gcc -L [library path] 
gcc -L/usr/lib/xxx test.c -o test
```

## GDB

**简介**：

GDB（GNU Debugger）是一款由GNU项目开发的、功能强大的调试工具。

### GDB常用命令

``` bash
# 导入需要调试的可执行程序
file xxx

# 运行
run | r
# 单步步过执行
next | n
# 单步步入执行
step | s
# 单步汇编指令执行
step instruction | si
# 运行到下一断点
continue

# 添加断点
break func1 | break main.c:10
b func1 | b main.c:10
# 删除断点
delete 3

# 查看当前执行位置代码
list
# 查看当前执行位置汇编指令
disassemble | disas
# 查看当前调用链
backtrace | bt

# 查看所有断点信息
info break
# 查看当前寄存器信息
info registers
# 查看当前堆栈
info stack
# 查看当前调用堆栈帧帧
info frame
# 查看当前局部变量
info locals

# 打印某地址值
examine /[length][format] [address] | x /[length][format] [address]
x /1c  0x7ffff7d44060
# 打印某变量值
print xxx | p xxx
```

## Make

**简介**：

Make是一款用于自动化构建程序的工具，可以根据目标文件的依赖关系自动构建出最终的可执行文件或库文件。

### Makefile

Makefile是Make工具使用的配置文件，其包含了一系列规则，用于描述目标文件的依赖关系和构建方法。它的语法与Shell脚本非常类似。一般来说，一个Makefile文件中的规则会包含以下几个部分：

- 目标(Target)：指定要构建的目标文件的名称。目标可以是可执行文件、库文件或其他文件。
  - 伪造目标(Phony Target)：与明确的构建目标相比，伪造目标一般并非真实存在的文件。当使用Make命令构建伪造目标时，操作系统只会执行其指定的一些列目标，而忽略该文件，即使它真的存在。这种技术一般用于清理中间文件，比如：设置clean作为伪造目标。当执行`make clean`命令时，即使当前目录真的有一个clean文件存在，系统也会执行clean目标指定的一些规则。
- 依赖(Prerequisites)：指定构建目标文件所需要的源文件、头文件或其他依赖文件。
- 命令(Command)：指定构建目标文件的具体命令，通常是编译源文件、链接目标文件等操作。

```Makefile
# 变量定义
CC = gcc
CFLAGS = -Wall -Wextra -Werror

# 默认目标
all: myprogram

# 依赖关系和命令
myprogram: main.o utils.o
$(CC) $(CFLAGS) -o $@ $^

main.o: main.c utils.h
$(CC) $(CFLAGS) -c $<

utils.o: utils.c utils.h
$(CC) $(CFLAGS) -c $<

# 清理目标（伪造目标）
.PHONY: clean
clean:
rm -f myprogram *.o
```

此处再补充一下上面出现的通配符含义：

- $@：表示当前规则中的目标文件名。
- $<：表示当前规则中的第一个依赖文件名。
- $^：表示当前规则中的所有依赖文件名。

### CMake

事实上，在大部分时间我们都不会直接去写Makefile文件，尤其是面对大型项目，我们更习惯于使用另一种工具去帮我们生成MakeFile，再使用Make进行编译。其中，最常用的一种工具就是CMake。

CMake（Cross-platform Make）是一个跨平台的构建自动化工具，可以帮助软件开发者自动生成不同平台下的构建系统文件。此外，CMake不光可以使用Make作为其构建工具，也可以结合Ninja、Visual Studio、QT等使用。

使用CMake，我们可以在一个统一的CMakeLists.txt文件中描述项目的所有构建过程，包括源代码文件、依赖库、编译选项、链接选项等，而不需要为不同的构建系统编写不同的文件。CMake会根据CMakeLists.txt文件自动生成相应的构建系统文件，从而实现跨平台的构建。

## Coreutils

**简介**：

Coreutils是一组基本的Unix工具，包括了大量的命令行工具，用于文件操作、文本处理、进程管理等方面。

在Linux系统中，Coreutils通常是系统中最基本的工具集之一，包含了许多常用的命令，例如：ls、rm、cp、mv、mkdir、cat、echo、grep等等。

## Binutils

**简介**：

Binutils是GNU项目的一个组成部分，包括了一系列用于操作目标文件的命令行工具。可以按功能将Binutils分成两大部分，分别是

- 帮助生成可执行文件的工具，比如：ld (linker), as (assembler), ar (archiver)；
- 帮助分析可执行文件的工具，比如：objcopy/objdump/readelf。

## References

- [Debugging Under Unix: gdb Tutorial](https://www.cs.cmu.edu/~gilpin/tutorial/)
- [GNU Make](https://www.gnu.org/software/make/)
- [GNU Binutils](https://www.gnu.org/software/binutils/)

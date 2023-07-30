---
title: "一点的感悟：关于Coding和读研"
date: 2023-07-24T11:11:34+08:00
draft: true
---

作为一个练习时长三年半但依旧迷茫的彩笔面向博客编程工程师，回顾我的Coding学习之路，可谓是走了非常之多的弯路。这篇博客就以我目前浅薄的认识来对讲一讲我对学习写代码并打算以此谋生这件事的认识和感悟。

## 选择一门语言开始

毫无疑问，学习Coding首先就需要选择一门编程语言，从hello world开始，了解其各种语法，再使用它完成一些大的项目，从而融会贯通。可能有一些人建议初学者选择Python或者Java作为第一门语言。前者，简单易学同时一直都是[TIOBE编程语言流行度排行榜](https://www.tiobe.com/tiobe-index/)中的翘楚，而后者则是面向就业。

但我个人认为初学者第一门语言的最优选择还是C/C++，有如下几个原因：

- C/C++是后续学习的前置条件。因为大量著名的计算机课程都将C/C++作为其课程实验的指定语言，比如：[CMU15-445 BusTub](https://github.com/cmu-db/bustub)、[CS144 Sponge](https://cs144.github.io/doc/lab0/)、[NJU OS](https://github.com/NJU-ProjectN/os-workbench-2022)等等。如果你不会C/C++，那么你就没法学习这些优秀的课程。
- C/C++更接近硬件。许多高级语言都屏蔽了硬件设施，相反C/C++更接近底层，它们能够帮助我们理解计算机是如何运作的。事实上，很多高级语言都是依赖于C/C++实现的，比如：Python的一些核心库（如NumPy、SciPy等）是用C/C++编写的；Java虚拟机（JVM）的实现就是用C++编写的。
- 从C/C++迁移到其他语言非常方便。尽管C/C++是一门历史悠久的语言，但它也随着时代再不断修改，吸取其他语言的长处，所以在把C/C++基础打牢之后，学习新语言也并非难事。

不可否认的是，C/C++由于带着沉重的历史包袱，在诸如：字符串处理、时间处理等方面实在是难以恭维。但它仍不失为一门优秀的编程语言。

### 学习资料

**网课**：

- [浙江大学 C语言 翁恺](https://www.bilibili.com/video/BV1YW411x7eN/?vd_source=1602aa2def0a5452e1d6a6f65ea4da59)：首先必须要推荐一下我们学校翁恺老师开始的这门C语言课程，算是一门很不错的C语言网课。记得我当初上的时候，除了学到了基础的语法：变量、流控制、函数和字符串等等，还有一些关于Linux、终端、Shell、GCC、Make等非常有用的工具介绍。
- [Standford C++ CS106L](http://web.stanford.edu/class/cs106l/)：斯坦福开设的另一门C++语言课。与它开设的另一门主要关注数据结构的[CS106B](https://web.stanford.edu/class/cs106b/)不同，CS106L主要聚焦于深入讲解C++语言的语法特性，让学生能够写出高质量的C++代码。

**教材**：

- [C++ Primer](https://github.com/applenob/Cpp_Primer_Practice)：没什么好说的，C++的圣经罢了。
- [Effective Modern C++](https://github.com/CnTransGroup/EffectiveModernCppChinese)：没什么好说的，C++的另一部圣经罢了。

### 练手项目

**基础**：

- [SimpleSTL](https://github.com/Zhytou/SimpleSTL)：模仿STL模板库，实现各种容器，从而进一步熟悉C++类、构造函数、RAII和模板等概念。
- [Json Paser](https://github.com/Zhytou/MyJsonParser)：类似SimpleSTL项目一样，这个项目的目的也是让我们熟悉C++类、构造函数、RAII和模板等概念，只不过它给出了一个更有意义的场景，让我们感觉不那么像造轮子。关于这个项目的介绍可以看这篇文章[知乎 Milo Yip​ 从零开始的 JSON 库教程](https://zhuanlan.zhihu.com/p/22457315)。

**进阶**：

- [MultiThread Practice In CPP](https://github.com/Zhytou/MultiThreadPracticeInCPP)：利用C++实现无锁队列、生产者-消费者问题、线程库等等，从而让我们熟悉C++的并发编程。进一步认识互斥量、条件变量以及RAII等概念。
- [Muduo](https://github.com/chenshuo/muduo)：陈硕大佬写的网络库，可以配合着他的书阅读，会让我们对C++的网络编程有很深的认识。看完之后，也可以自己仿照着写一写，不过难度比较大就是了。这里给出一个不错的参考[TinyWebServer](https://github.com/qinguoyi/TinyWebServer)。

## 学习编程的核心课程

我个人认为，编程的基础其实就是一门编程语言/数据结构+操作系统+计算机网络。甚至再极端一点，只需要学习一门编程语言/数据结构+操作系统，再配合上一些对应岗位的知识，也足以找一份工作了。

### 操作系统

**网课**：

虽然有很多优秀的操作系统课程，但我还是会推荐学习[NJU-OS](https://jyywiki.cn/OS/2022/)。就像JYY老师说的那样，认认真真学完这门课之后，你会变得很强，各种意义上的强。

**资料**：

- [CSAPP Computer Systems A Programmer's perspective](https://github.com/iWangMu/Book-CSAPP)：没啥好说的，又一部圣经罢了。
- [OSTEP Operating Systems: Three Easy Pieces](https://github.com/remzi-arpacidusseau/ostep-translations/tree/master/chinese)：一本很好的操作系统教材，也是NJU-OS的参考教材。

### 计算机网络

## 掌握一些生产工具

相信走到这一步的同学，已经有了一套自己的生产环境以及工具链。不过，这里仍然非常建议你学习这门课程[计算机教育中缺失的一课](https://missing-semester-cn.github.io/)或者阅读我的一篇博客[搭建自己的生产环境](https://zhytou.top/post/2023-7-3/cli-env/)。它会让你对这些工具有全新的认识，从而让你的工作效率翻倍。

## 选择进阶方向

**常规的前端/后端**：

在掌握了一门编程语言，并且了解数据结构和算法、操作系统和计算机网络之后，这时候希望转码做一名前端/后端/客户端开发工程师的朋友，就可以去学习对应的框架技术栈了。比如：想做Java后端开发的朋友，可以去进一步学习Spring/Mybatis/MySQL相关知识；想做前端开发的朋友，可以去进一步学习Vue/React/Node相关知识。

但按我个人观点，现在继续投入到这个互联网赛道是一个竞争激烈、风险高的选择，因为大量招聘这些岗位的企业大部分是业务大于技术的。我当然不是在说腾讯阿里这些大企业没有技术，我想表达的无非是他们大量部门有技术但业务才是他们的立足之本。比如，微信这个产品真的算不上好用吧？但依旧它是国内最主要的即时通讯软件。这是因为它有庞大的用户基数，你周围的人都在用它决定了你大概率也要用它。总之，在这种业务导向的氛围中，技术人员不仅话语权低，而且可替代性也非常高。

那哪种企业是以技术为导向的呢？我个人认为传统的、非行业垄断的IT公司其实都算。比如，华为或者苹果（手机部门）等偏硬件公司、AMD或Nvidia等芯片公司、甲骨文或PingCAP等数据库公司、幻方或JUMP等量化公司、SAP等ERP软件公司、AutoDesk等工业设计软件公司、Xilinx等电子仿真软件公司。毫无疑问，这些公司都是以技术为护城河的代表。工程师就是它们的雇佣兵，真正的按能力挣工资的典型。也正是这一点，想要进入这类型的企业，往往需要在其对应的方向有所研究，除了学历或者Paper之外真正的能力。

说了这么多，无非是建议大家去再拓展一些方向，学习一些真正技术。

**其他方向**：

下面给出一些我觉得不错的入门材料以及它们对应的方向。

- 计算机图形学：[Games101](http://games-cn.org/intro-graphics/)
- 数据库：[CMU14-445](https://15445.courses.cs.cmu.edu/fall2022/schedule.html)
- 分布式系统：[MIT6.824](https://pdos.csail.mit.edu/6.824/)
- 高性能计算：[Introduction to Parallel Computing Tutorial](https://hpc.llnl.gov/documentation/tutorials/introduction-parallel-computing-tutorial)
- 编译原理：[Build a JIT in LLVM](https://llvm.org/docs/tutorial/#building-a-jit-in-llvm)
- 机器学习系统：[智能计算系统](https://novel.ict.ac.cn/aics/)

更多详细的课程入门选择，可以浏览这个网站[CS自学指南](https://csdiy.wiki/)中的信息。除此之外，还有一些非常优秀的教科书、论文或者博客可以参考[这个回答](https://www.zhihu.com/question/37647788/answer/2466804512)。

### 读研

按照上面的这套流程去学习编程语言以及计算机核心课程并选择一些进阶方向深挖，个人认为至少也需要个一年半载。再加上一些走弯路、找方向的时间，一名像我一样大三上才绝对正式地转码的同学，很可能还需要在本科的基础之上，再续两到三年的研究生，也就是这几年的常见的读研转码。这里我也以自己为例，谈一下我现在的感悟。

首先简单概括一下我转码的经历。

### 实习/开源项目

作为一个20年就在面试互联网公司的人，我觉得我对于实习的经验还是可以分享一下的。

尽管我几乎没有什么开源经历，但我真的尝试去加入一些社区了。可能是我准备不够充分，也可能是我选择的一些社区

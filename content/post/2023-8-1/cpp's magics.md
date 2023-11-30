---
title: "C/C++奇技淫巧"
date: 2023-08-01T22:16:58+08:00
draft: false
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

Intel Intrinsics是内置在Intel编译器中的特殊函数，它们可以直接调用处理器的SIMD指令集，比如AVX、SSE等。换句话说，它的主要目的就是简化SIMD编程，帮助开发者充分利用现代CPU的并行计算能力。

**OpenMP**：

除了Intrinsics这种偏向底层的技术之外，我们还可以使用OpenMP这类库来更容易的实现并行，比如：

```c++
#include <omp.h>

int main() {
  #pragma omp parallel for
  for(int i=0; i<100; i++) {
    // parallel loop body
  }

  return 0;
}
```

## 编译期运算

**宏**：

**模板**：

在我们的印象中，模板可能大部分时候都是用于函数或者类，保证一份代码可针对多种类型复用。复杂一点的可能配合萃取技术进行类型运算，实现SFINAE，比如：enable_if、is_same等等。

但实际上，模板还可以用于编译期计算，可以将运行时消耗转移到编译期消耗，比如：

```c++
template <size_t N> 
struct Fibonacci {  
    constexpr static size_t value = 
        Fibonacci<N - 1>::value +
        Fibonacci<N - 2>::value;
};

template <> struct Fibonacci<0> {   
    constexpr static size_t value = 0;
};

template <> struct Fibonacci<1> {   
    constexpr static size_t value = 1;
}

template<size_t N>
constexpr size_t Fibonacci_v = Fibonacci<N>::value; 
```

**constexpr**：

constexpr作为新关键字在C++11引入，它的作用是修饰能在编译期确定或完成计算变量或函数。随着C++20的普及，constexpr的含义越来越丰富，引入了包括if constexpr、constexpr容器等新特性，"constexpr all the things!"也作为元编程的一种新流派出现。常见的constexpr用法包括：

- 编译期计算常量；

```c++
template<size_t N>
constexpr size_t fibonacci = fibonacci<N - 1> + fibonacci<N - 2>;
template<>
constexpr size_t fibonacci<0> = 0;
template<>
constexpr size_t fibonacci<1> = 1;

static_assert(fibonacci<10> == 55);
```

- if constexpr配合类型萃取或约束（C++20特性）实现类似SFINAE；

```c++
template<typename T, typename Tag>
void process(T t, Tag tag) {
  if constexpr(std::is_same_v<Tag, int>) {
    // 处理int标签版本
  } else {
    // 其他标签版本 
  }
}
```

- C++17起，lambda默认为constexpr，比如：

```c++
// constexpr int fibonacci(int n);
auto fibonacci = [](int n) {
    int a = 0, b = 1;
    for (int c = 0; c < n; ++ c) {
        int t = a + b;
        a = b;
        b = t;
    }
    return a;
};

- constexpr容器；

```c++
constexpr std::array<int, 5> array = {1, 2, 3, 4, 5}; 

constexpr int sum() {
  int total = 0;
  for (const auto& element : array) {
    total += element; 
  }
  return total;
}
```
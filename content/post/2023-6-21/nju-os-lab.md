---
title: "NJU-OS-Lab总结"
date: 2023-06-21T16:42:31+08:00
draft: false
---

## M1：实现打印进程树状信息的工具 pstree

## M2：实现协程库 libco

### 协程

**理解协程**：

在开始做本实验之前，我认为可以先阅读一些材料来认识进程、线程和协程之间的关系，比如这一篇[Process , Thread and Coroutines](https://www.linkedin.com/pulse/process-thread-coroutines-amit-nadiger/)。当我们能够认识到协程其实就是调度由用户管理的轻量级线程，即：用户态线程时，实验说明中的大部分内容就很好理解了。

至于协程结构体，则可以直接使用实验讲义中给出的，具体如下：

``` c
// 协程状态
enum co_status {
  CO_NEW = 1,  // 新创建，还未执行过
  CO_RUNNING,  // 已经执行过
  CO_WAITING,  // 在 co_wait 上等待
  CO_DEAD,     // 已经结束，但还未释放资源
};

// 协程
struct __attribute__((aligned(16))) co {
  char *name;
  void (*func)(void *);  // co_start 指定的入口地址和参数
  void *arg;

  enum co_status status;  // 协程的状态
  struct co *waiter;      // 是否有其他协程在等待当前协程
  jmp_buf context;        // 寄存器现场 (setjmp.h)
  unsigned char stack[STACK_SIZE];  // 协程的堆栈
};
```

**管理协程**：

libco中应该记录当前执行协程以及当前所有协程，以便选择协程切换和保存当前协程状态。其中，我用来记录所有协程的数据结构是链表，实现如下：

``` c
// 当前运行协程，默认是main
struct co *current = NULL;

// 当前所有协程表
struct co_mgr_node *co_mgr_head = NULL;

// 协程管理，用链表存储当前上下文所有协程
struct co_mgr_node {
  struct co *val;
  struct co_mgr_node *next;
};
```

**资源初始化和释放**：

定义`__attribute__((constructor))`属性和`__attribute__((destructor))`属性的函数，保证在main函数执行之前正确初始化资源，在main函数 结束之后释放资源。

``` c
// co_mgr_head和current初始化，main函数执行前调用
static __attribute__((constructor)) void co_constructor(void) {
  // 创建主线程，并将其加入到协程管理表中
  current = co_start("main", NULL, NULL);
  if (current == NULL) {
    debug_printf("co_start main failed\n");
    return;
  }
  current->status = CO_RUNNING;
}

// co_mgr_head和current释放，main函数执行后调用
static __attribute__((destructor)) void co_destructor(void) {
  struct co_mgr_node *curr = co_mgr_head;
  while (curr != NULL) {
    struct co_mgr_node *next = curr->next;
    free(curr->val);
    free(curr);
    curr = next;
  }
  co_mgr_head = NULL;
  current = NULL;
}
```

### 上下文切换

**理解setjmp/logjmp**：

**实现co_field**：

在理解了

``` c
// 切换协程
void co_yield () {
  int ret = setjmp(current->context);
  // 若ret为0，即为第一次设置jmp_buf（可以使用longjmp函数恢复寄存器状态），切换到一个新建或运行中的协程执行
  // 否则为longjmp函数调用，直接返回
  if (ret == 0) {
    // 找到一个可以切换的协程
    struct co *co = get_random_available_co(co_mgr_head);

    if (co->status == CO_NEW) {
      // 切换上下文，执行current->func
      ((volatile struct co *)co)->status = CO_RUNNING;
      stack_switch_call(co->stack + STACK_SIZE, co->func, co->arg);
      // 恢复寄存器状态
      restore_registers_call();
      ((volatile struct co *)co)->status = CO_DEAD;
      // 修改等待协程的状态
      if (co->waiter != NULL) {
        co->waiter->status = CO_RUNNING;
      }
      //
      co_yield ();
    } else if (co->status == CO_RUNNING) {
      longjmp(co->context, 1);
    }
  }
  return;
}
```

**实现stack_swicth_call与restore_registers_call**：

![调用栈](https://pic2.zhimg.com/80/v2-bd5a0aa1625c4445ba33e506b91dba29_1440w.webp)

### 调试 & 坑

**调试信息打印函数**：

为了方便调试，我们可以定义一个debug_printf函数，并利用宏的机制来开启或关闭该信息的显示。

``` c
#define DEBUG 0

// 调试打印函数
void debug_printf(const char *fmt, ...) {
  if (DEBUG) {
    va_list args;
    va_start(args, fmt);
    vprintf(fmt, args);
    va_end(args);
  }
}
```

**GDB**：

由于这个实验涉及到使用内联汇编代码实现上下文切换，掌握gdb按汇编指令单步运行，来查看stack_swicth_call与restore_registers_call前后，寄存器是否保存正确、栈是否对齐，就显得十分重要。

关于如何使用gdb调试，具体可以参考我的[博客](https://zhytou.top/post/2023-6-27/gnu-tools/)，里面几乎涵盖了这次实验会用到的所有gdb命令。

**我碰到的坑**：

1. 堆栈没有对齐，报错SEGEFV；

2. 画蛇添足，在co_wait里若发现协程状态为CO_DEAD就主动释放了该协程内存：

不要在co_wait中释放协程，在destructor中统一释放。

### M2 总结

这个实验很好的锻炼

## 参考

**M2**：

- [Process , Thread and Coroutines](https://www.linkedin.com/pulse/process-thread-coroutines-amit-nadiger/)
- [x86系统调用及栈帧原理](https://zhuanlan.zhihu.com/p/27339191)

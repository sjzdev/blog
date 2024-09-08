---
blog: true
tags:
  - 内核态
  - 用户态
  - 特权级
  - 操作系统
categories:
  - ucore实验
collections: 
title: ucore操作系统实验：lab1-拓展实验1
date: 2022-04-14 00:32:27
update: 2024-09-08 21:40:37
---

## 练习内容

![](/blog/img/IMG-20240908212558386.png)

## 问题解答

本实验的入口为如下函数，其中 `lab1_switch_to_user()` 使系统由内核态切换到用户态，`lab1_switch_to_kernel()` 使系统由用户态切换到内核态。

```c
static void
lab1_switch_test(void) {
    lab1_print_cur_status();
    cprintf("+++ switch to  user  mode +++\n");
    lab1_switch_to_user();
    lab1_print_cur_status();
    cprintf("+++ switch to kernel mode +++\n");
    lab1_switch_to_kernel();
    lab1_print_cur_status();
}
```

### 内核态切换到用户态

`lab1_switch_to_user` 函数的定义如下：

```c
static void
lab1_switch_to_user(void) {
    //LAB1 CHALLENGE 1 : TODO
    asm volatile (
        "sub $0x8, %%esp \n"
        "int %0 \n"
        "movl %%ebp, %%esp"  
        :
        : "i"(T_SWITCH_TOU)
    );
}
```

可以看到函数中使用内嵌汇编，作用是触发中断：

- `sub $0x8, %%esp` 将栈指针减去 `0x8`，以分配一些额外的栈空间；
- `int %0` 触发中断处理程序，`%0` 是一个占位符，具体值为 `T_SWITCH_TOU`；
- `movl %%ebp, %%esp` 函数将栈指针恢复为初始值。

该函数调用结束后，系统就由内核态切换到用户态了，为了了解切换的细节，我们在这里以函数栈为切入点，观察一下特权级切换过程中函数栈的变化。

利用 gdb 调试代码，可以看到函数 `lab1_switch_to_user` 对应的汇编指令如下：

![](/blog/img/IMG-20240908212558550.png)

开始执行汇编指令 `sub $0x8, %%esp` 时函数栈如下：

![](/blog/img/IMG-20240908212559188.png)

`sub $0x8, %%esp` 的主要目的是在栈中预留一些空间，具体存放的内容后文中会有介绍，执行完本条指令后函数栈变为：

![](/blog/img/IMG-20240908212559216.png)

`int %0` 指令执行后时系统陷入终端，此时硬件会自动的将一部分寄存器的值压入到函数栈中：

![](/blog/img/IMG-20240908212559664.png)

中断处理程序定义在 `kern/trap/vectors.S` 文件中，所有的中断处理程序的入口都是类似，先将中断号压入栈中，在跳转到 `__alltraps` 进行执行：

```asm
# handler
.text
.globl __alltraps
.globl vector0
vector0:
  pushl $0
  pushl $0
  jmp __alltraps
.globl vector1
vector1:
  pushl $0
  pushl $1
  jmp __alltraps
.globl vector2
vector2:
  pushl $0
  pushl $2
  jmp __alltraps
.globl vector3
```

![](/blog/img/IMG-20240908212559688.png)

`__alltraps` 定义在文件 `kern/trap/trapentry.S` 中：

```asm
#include <memlayout.h>

# vectors.S 将所有陷阱发送至此处。
.text
.globl __alltraps
__alltraps:
    # 将寄存器推入陷阱框架中
    # 因此使堆栈看起来像一个 struct trapframe
    pushl %ds
    pushl %es
    pushl %fs
    pushl %gs
    pushal

    # 将 GD_KDATA 载入到 %ds 和 %es 中，以设置内核的数据段
    movl $GD_KDATA, %eax
    movw %ax, %ds
    movw %ax, %es

    # 推入 %esp 作为参数传递给 trap(tf)
    pushl %esp

    # 调用 trap(tf)，其中 tf=%esp
    call trap

    # 弹出已推入的堆栈指针
    popl %esp

    # 返回，继续执行 trapret...
.globl __trapret
__trapret:
    # 从堆栈中恢复寄存器
    popal

    # 恢复 %ds、%es、%fs 和 %gs
    popl %gs
    popl %fs
    popl %es
    popl %ds

    # 清除陷阱号和错误码
    addl $0x8, %esp
    iret
```

执行到 `pushal` 时，函数栈状态如下：

![](/blog/img/IMG-20240908212600064.png)

执行到到 `call trap` 时函数栈如下：

![](/blog/img/IMG-20240908212600098.png)

之后会跳转到 `kern/trap/trap.c` 中的 `trap` 函数：

```c
void trap(struct trapframe *tf) { 
	// 根据不同的陷阱类型进行分发 
	trap_dispatch(tf); 
}

struct trapframe {
    struct pushregs tf_regs;            // 用于保存寄存器的结构体
    uint16_t tf_gs;                     // 任务切换寄存器
    uint16_t tf_padding0;               // 填充字节
    uint16_t tf_fs;                     // 文件描述符寄存器
    uint16_t tf_padding1;               // 填充字节
    uint16_t tf_es;                     // 段寄存器
    uint16_t tf_padding2;               // 填充字节
    uint16_t tf_ds;                     // 数据段寄存器
    uint16_t tf_padding3;               // 填充字节
    uint32_t tf_trapno;                 // 陷阱号
    /* below here defined by x86 hardware */
    uint32_t tf_err;                    // 错误码
    uintptr_t tf_eip;                   // 指令指针
    uint16_t tf_cs;                     // 指令段寄存器
    uint16_t tf_padding4;               // 填充字节
    uint32_t tf_eflags;                 // 标志寄存器
    /* below here only when crossing rings, such as from user to kernel */
    uintptr_t tf_esp;                   // 堆栈指针
    uint16_t tf_ss;                     // 堆栈段寄存器
    uint16_t tf_padding5;               // 填充字节
} __attribute__((packed));

struct pushregs {
    uint32_t reg_edi;        // 指向堆栈顶部的指针 EDI
    uint32_t reg_esi;        // 指向数据块的指针 ESP
    uint32_t reg_ebp;        // 栈指针的基准值 EBP
    uint32_t reg_oesp;       /* 无用 */ 
    uint32_t reg_ebx;        // 指向匹配项的指针 EBX
    uint32_t reg_edx;        // 索引指针 EDX
    uint32_t reg_ecx;        // 计数器 ECX
    uint32_t reg_eax;        // 结果寄存器 EAX
};

```

`trap` 函数的参数具体值为 `esp`，结构体 `trapframe` 对应的内存区域为图示中被阴影覆盖的范围：

![](/blog/img/IMG-20240908212600423.png)

`trap_dispatch` 函数根据 `tf` 中保存的中断号找到对应的处理程序：

```C
static void
trap_dispatch(struct trapframe *tf) {
    char c;
    switch (tf->tf_trapno) {
    // 省略其他代码
	case T_SWITCH_TOU:
        if (tf->tf_cs != USER_CS) {
	        // 拷贝tf的值
            switchk2u = *tf;
            // 修改cs
            switchk2u.tf_cs = USER_CS;
            // 修改ds、es、ss
            switchk2u.tf_ds = switchk2u.tf_es = switchk2u.tf_ss = USER_DS;
            // 修改esp的值
            switchk2u.tf_esp = (uint32_t)tf + sizeof(struct trapframe) - 8;

            // 设置eflags，使ucore能在用户模式下使用io
            // 如果CPL > IOPL，cpu将生成一般保护错误
            switchk2u.tf_eflags |= FL_IOPL_MASK;

            // 设置临时堆栈
            // 然后iret将跳转到正确的堆栈
            *((uint32_t *)tf - 1) = (uint32_t)&switchk2u;
        }
        break;  
     // 省略其他代码代码   
}
```

![](/blog/img/IMG-20240908212600631.png)

`tarp` 函数执行结束后，跳转到 `__alltraps` 继续执行 ，执行到 `popl %esp` 时，函数栈为：

![](/blog/img/IMG-20240908212600663.png)

执行到 `addl $0x8, %esp` 时，函数栈为：

![](/blog/img/IMG-20240908212601016.png)

执行到 `iret` 后，函数栈状态为：

![](/blog/img/IMG-20240908212601049.png)

```c
static void
lab1_switch_to_user(void) {
    //LAB1 CHALLENGE 1 : TODO
    asm volatile (
        "sub $0x8, %%esp \n"
        "int %0 \n"
        "movl %%ebp, %%esp"  
        :
        : "i"(T_SWITCH_TOU)
    );
}
```

最后函数执行完，函数栈恢复原样：

![](/blog/img/IMG-20240908212559188.png)

> 问题：根据流程的推导，发现 `trap_dispatch` 函数中如下语句的似乎不是必须得，注释掉该语句后操作系统依然能够启动，根据输出结果来看貌似是正常的。

```c
switchk2u.tf_esp = (uint32_t)tf + sizeof(struct trapframe) - 8;
*((uint32_t *)tf - 1) = (uint32_t)&switchk2u;
```

![](/blog/img/IMG-20240908212601232.png)

### 用户态切换到内核态

```c
static void
lab1_switch_to_kernel(void) {
    //LAB1 CHALLENGE 1 :  TODO
	asm volatile (
	    "int %0 \n"
	    "movl %%ebp, %%esp \n"
	    : 
	    : "i"(T_SWITCH_TOK)
	);
}
```

![](/blog/img/IMG-20240908212601350.png)

用户态切换到内核态的流程与内核态切换到用户态大体类似，主要的区别在于相比于 `lab1_switch_to_user`，函数 `lab1_switch_to_kernel` 中的少了指令 `sub $0x8, %%esp`，这是因为由用户态到内核态变换的过程中，触发 `int` 指令的一瞬间，CPU 会因为特权级的改变而索引 TSS，切换 `ss` 和 `esp` 为内核栈，并压入 `ss`、`esp` 和其他一些寄存器的值。内核态向用户态切换的过程中，`int` 指令执行时不存在特权级的变化，所以需要手动为 `ss` 和 `esp` 预留空间。

![](/blog/img/IMG-20240908212601406.png)

```c
static void
trap_dispatch(struct trapframe *tf) {
    char c;
    switch (tf->tf_trapno) {
    // 省略其他代码
	case T_SWITCH_TOK:
        // 如果当前任务切换段不是内核段
        if (tf->tf_cs != KERNEL_CS) {
            // 将当前任务切换段设为内核段
            tf->tf_cs = KERNEL_CS;
            // 将当前任务切换数据段和附加段设为内核数据段
            tf->tf_ds = tf->tf_es = KERNEL_DS;
            // 清除任务切换标志中的I/O段级别掩码
            tf->tf_eflags &= ~FL_IOPL_MASK;
            // 通过将当前任务切换帧的地址减去(sizeof(struct trapframe) - 8)得到从用户段到内核段的任务切换帧地址
            switchu2k = (struct trapframe *)(tf->tf_esp - (sizeof(struct trapframe) - 8));
            // 将当前任务切换帧的内容复制到从用户段到内核段的任务切换帧中
            memmove(switchu2k, tf, sizeof(struct trapframe) - 8);
            // 将当前任务切换帧的上一个内存地址赋值为从用户段到内核段的任务切换帧的地址
            *((uint32_t *)tf - 1) = (uint32_t)switchu2k;
        }
        // 跳出switch语句
        break;
     // 省略其他代码代码   
}
```

![](/blog/img/IMG-20240908212601462.png)
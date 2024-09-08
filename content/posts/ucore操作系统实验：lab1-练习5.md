---
blog: true
tags:
  - 函数栈
categories:
  - ucore实验
collections: 
title: ucore操作系统实验：lab1-练习5
date: 2022-03-25 00:32:28
update: 2024-09-08 21:40:19
---

## 练习内容

![](/blog/img/IMG-20240908212558397.png)

## 问题解答

根据输出样例，我们可以知道在 `print_stackframe` 要做的的是把遍历当前的函数栈，并打印每个栈中的 `ebp`、`eip` 和函数中的参数。

```c
void
print_stackframe(void) {
     /* LAB1 YOUR CODE : STEP 1 */
     /* (1) call read_ebp() to get the value of ebp. the type is (uint32_t);
      * (2) call read_eip() to get the value of eip. the type is (uint32_t);
      * (3) from 0 .. STACKFRAME_DEPTH
      *    (3.1) printf value of ebp, eip
      *    (3.2) (uint32_t)calling arguments [0..4] = the contents in address (uint32_t)ebp +2 [0..4]
      *    (3.3) cprintf("\n");
      *    (3.4) call print_debuginfo(eip-1) to print the C calling function name and line number, etc.
      *    (3.5) popup a calling stackframe
      *           NOTICE: the calling funciton's return addr eip  = ss:[ebp+4]
      *                   the calling funciton's ebp = ss:[ebp]
      */

    // 读取ebp和eip
    uint32_t ebp = read_ebp(), eip = read_eip();
    
    int i, j;

    // 遍历栈
    for (i = 0; ebp != 0 && i < STACKFRAME_DEPTH; i ++) {
        
        // 打印 ebp eip
        cprintf("ebp:0x%08x eip:0x%08x args:", ebp, eip);
        
        // 打应参数
        uint32_t *args = (uint32_t *)ebp + 2;
        for (j = 0; j < 4; j ++) {
            cprintf("0x%08x ", args[j]);
        }
        cprintf("\n");
        print_debuginfo(eip - 1);

        // 切换栈信息, 需要注意的是栈是由高地址到低地址延伸的
        eip = ((uint32_t *)ebp)[1];
        ebp = ((uint32_t *)ebp)[0];
    }
}
```

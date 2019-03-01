# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

进程的切换需要时钟中断；对应的特权指令包括中断使能、软中断的相关指令。

虚存的实现需要MMU（TLB）进行虚实地址转换；对应内存管理的特权指令，如寻址模式设置、页表管理等等。

文件系统要求有稳定的外部存储介质；对应文件系统相关的特权指令，如I/O等。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

在实模式下，没有虚拟地址到实地址的映射，同时也没有内核态、用户态的区分；系统进程和用户进程的权限是相同的。这种情况下，如果系统进程的相关数据和代码被误修改或恶意修改，其结果是很严重的。在保护模式下，用户程序所得到的是虚拟地址，无法直接通过物理地址修改操作系统内核部分的数据，因而也更加安全。

物理地址指的是实际存储设备所对应的地址，是处理器提交到总线上进行实际访问的地址；

逻辑地址指的是程序所看到的逻辑意义上的地址，或者说在指令中进行虚实地址转换前程序所访问的地址，实际上是相对于某个段基址的偏移量；

线性地址是逻辑地址到物理地址的中间层，是处理器通过段机制控制下的形成的地址空间，实际上等于逻辑地址+段基址；有可能通过进一步的转换变成物理地址。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

risc-v提供了相应的特权指令规范和用户级指令规范，其中特权指令只能在特权模式下执行。根据risc-v指令集手册，risc-v一般提供的模式包括机器模式（M mode），管理员模式（S Mode）和用户模式（U mode），此三者的特权等级依次降低。其中机器模式是必须实现的，在此模式下运行的程序是固有可信的，可以在低层次访问机器的实现，即直接访问物理地址；管理员模式和用户模式一般对应操作系统和一般用户程序的运行。

一般用户程序所访问的逻辑地址是相对于自身基址的偏移量，对实际物理地址的访问需要先加上段基址，再进行虚实地址转换；
管理员模式下，只需要进行直接的虚实地址转换；这常常是通过直接的线性映射完成的；
机器模式下，允许直接访问物理地址，类似于所谓的实模式。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```
数字代表各个域在此结构体中占有的位数。
- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

在此处SETGATE的宏中，gate对应以intr地址开始的struct gatedesc, istrap = 1, sel = 2, off = 3, dpl = 0.
经过SETGATE后，gd_off_15_0域变为0x3，gd_ss域变为0x2，这两个域合起来共有32位，正好对应intr的值。考虑端序，已经说明gd_ss为low 16 bits，则intr的值为0x20003。

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)

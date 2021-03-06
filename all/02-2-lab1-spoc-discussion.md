# lec4: lab1 SPOC思考题

##**提前准备**
（请在上课前完成）

 - 完成lec4的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
 - 了解x86的保护模式，段选择子，全局描述符，全局描述符表，中断描述符表等概念，以及如何读写，设置等操作
 - 了解Linux中的ELF执行文件格式
 - 了解外设:串口，并口，时钟，键盘,CGA，已经如何对这些外设进行编程
 - 了解x86架构中的mem地址空间和io地址空间
 - 了解x86的中断处理过程（包括硬件部分和软件部分）
 - 了解GCC的x86/RV内联汇编
 - 了解C语言的可函数变参数编程
 - 了解qemu的启动参数的含义
 - 在piazza上就lec3学习中不理解问题进行提问
 - 学会使用 qemu
 - 在linux系统中，看看 /proc/cpuinfo的内容

## 思考题

### 启动顺序

1. x86段寄存器的字段含义和功能有哪些？

x86的段寄存器包括：

CS段寄存器（Code Segment），存放当前正在运行的程序的代码所在段的段基址；在获取该程序的代码时，通过段基址加上偏移量IP进行访问。

DS段寄存器（Data Segment），存放当前运行程序所需要的数据所在段的段基址（为地址最低的数据所在段的段基址）。

ES段寄存器（Extra Segment）,存放当前程序运行需要的附加数据所在段的段基址。【该段是串操作指令中目的串所在的段】

FS和GS段寄存器和ES的功能相似，同为辅助段寄存器。

SS段寄存器（Stack Segment），存放当前堆栈所在段的段基址。

2. x86描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中存在这些字段？对应的访问条件是什么？

CPL(Current Privilege Level)，表示当前执行的代码所在段的特权集，存放在CS寄存器的低两位。

RPL(Request Privilege Level)，表示当前进程对特定段访问的请求权限，存放在段选择子的低两位；一个段的RPL不一定是固定的。

DPL(Descriptor Privilege Level)，表示该段访问的权限级别，储存在段描述符中，对每个段是固定的。

成功访问某段的前提是，DPL >= max(CPL, RPL)。

3. 分析可执行文件格式elf的格式（无需回答）

### 4.1 C函数调用的实现

### 4.2 x86中断处理过程

1. x86/RV中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？

x86/RV中中断处理的硬件压栈内容包括：Error Code, EIP, CS寄存器，EFLAGS；用户态中断时，还需要额外将ESP寄存器和SS段寄存器的内容压入系统堆栈。

2. 为什么在用户态的中断响应要使用内核堆栈？

这是为了保护中断服务例程的安全。

3. x86中trap类型的中断门与interrupt类型的中断门有啥设置上的差别？如果在设置中断门上不做区分，会有什么可能的后果?

interrupt类型的中断门需要将EFLAGS里的IF位(Interrupt flag)清零，但trap类型的中断门不需要。亦即，在interrupt门调用时，新的interrupt会被禁止；但trap gate会保留相应的中断设置，不会改动。如果在设置中断门上不做区分，可能会导致中断发生嵌套（重复触发），即在处理某一个硬中断的同时发生其它硬中断，引起中断服务例程的跳转。

事实上，一般而言trap gate服务于软中断或者异常，而interrupt gate服务于硬中断。在处理硬中断的过程中强行切换到其它硬中断，可能引起死锁、程序崩溃等等问题（比如硬件所需要的操作可能要求一定的即时性），而异常则不会出现此情况。

### 4.3 练习四和五 ucore内核映像加载和函数调用栈分析

1. ucore中，在kdebug.c文件中用到的函数`read_ebp`是内联的，而函数`read_eip`不是内联的。为什么要设计成这样？

ebp的值可以通过一条指令直接获得，如果不内联则可能在某些情况下ebp的值发生变化，从而得到错误的ebp值。

eip的值不能直接获得，这里是利用call指令将eip压栈的特性，利用函数调用获得eip。如果将read_eip内联，则不存在函数调用，也就无法读取eip。


### 4.4 练习六 完善中断初始化和处理

1. CPU加电初始化后中断是使能的吗？为什么？

CPU加电初始化后中断不是使能的。加电后，BIOS会在自检完成后，向内存中加载中断向量表和中断服务例程。

## 开放思考题

1. 在ucore/rcore中如何修改lab1, 实现在出现除零异常时显示一个字符串的异常服务例程？
2. 在ucore lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
3. GRUB是一个通用的x86 bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
4. 如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？

## 课堂实践
### 练习一
在Linux系统的应用程序中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。

### 练习二
在ucore/rcore内核中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。

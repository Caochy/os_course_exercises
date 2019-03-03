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

    需要的硬件包含：
    1. CPU以及其包含的寄存器，运行单元和内存管理
    2. CPU的管脚接口
    3. 内存
    4. 硬盘
    5. 键盘等输入设备
    6. 显示器等输出设备
    7. 其他的资源如BIOS，ROM，Clock等
    
    需要包含的特权指令有：
    1. 与中断相关的指令：如LIDT，LTR，IRET，STI，CLI等
    2. 与虚存相关的指令：如MOV CRn，INVLPG，INVPCID等
    3. 与运行模式相关的指令：如SYSRET，SYSEXIT，IRET等
    4. 与分段分页相关的指令：如LGDT，LLDT CRx:CR0，CR3等

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

    实模式：
    
    软件可以访问的物理内存空间不能超过1MB，且无法发挥Intel 80386以上级别的32位CPU的4GB内存管理能力。
    
    保护模式：
    
    支持内存分页机制，提供了对虚拟内存的良好支持。支持多任务和优先级机制，不同的程序可以运行在不同的优先级上。配合良好的检查机制，既可以在任务间实现数据的安全共享也可以很好地隔离各个任务。
    
    物理地址：
    
    处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址，一个计算机系统中只能有一个物理地址空间。

    线性地址：
    
    在操作系统的虚存管理之下每个运行的应用程序能访问的地址空间。每个运行的应用程序都认为自己独享整个计算机系统的地址空间，这样可以让多个运行的应用程序之间相互独立。

    逻辑地址：
    
    应用程序直接使用的地址空间。


- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？
    RISC-V定义了四个特权级。最高的特权级为机器级，这是RISC-V平台唯一必须的特权级，可以在低层次访问机器的实现。用户模式和管理员模式用于传统应用程序和操作系统。Hypervisor模式支持了虚拟机监视器。

    不同的模式可访问的地址空间范围，访问权限和访问方式均有不同的限制。


- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义

    ":"后的数字的含义为存储该变量的位数。
    可见该结构体的大小为64位。
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

结构体中各个变量的值如下所示：
```
gd_off_15_0     = 0x0003
gd_ss           = 0x0002
gd_args         = 0x00
gd_rsv1         = 0x0
gd_type         = 0xf
gd_s            = 0x0
gd_dpl          = 0x0
gd_p            = 0x1
gd_off_31_16    = 0x0
```
连接为一个64位整数为：
```
0000 0000 0000 0000
1000 1111 0000 0000
0000 0000 0000 0010
0000 0000 0000 0011
```
即0x00008f0000020003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

    代码来源为lab1/kern/trap/trapentry.S，代码段如下：

    这段代码的主要内容为处理中断的入口代码，需要完成的工作是将当前的寄存器内容存储起来，再将中断处理所需的数据载入到寄存器中，执行中断处理程序。最后返回中断前的状态。

    在.text后就是汇编程序的代码段。第一个函数为__alltraps，函数首先用push将%ds，%es，%fs，%gs四个寄存器中的内容存储到栈中，再用mov指令将$GD_DATA等数据载入到空出的寄存器中。通过push指令将trap()函数需要的参数用栈进行传递，之后调用中断处理函数。在处理完毕后，弹出之前的参数，进行现场还原。

    现场还原的函数为__trapret，函数将存储在栈中的数据依次通过pop指令还原到寄存器中，更新栈指针，返回，继续进行中断后的程序。
```
# vectors.S sends all traps here.
.text
.globl __alltraps
__alltraps:
    # push registers to build a trap frame
    # therefore make the stack look like a struct trapframe
    pushl %ds
    pushl %es
    pushl %fs
    pushl %gs
    pushal

    # load GD_KDATA into %ds and %es to set up data segments for kernel
    movl $GD_KDATA, %eax
    movw %ax, %ds
    movw %ax, %es

    # push %esp to pass a pointer to the trapframe as an argument to trap()
    pushl %esp

    # call trap(tf), where tf=%esp
    call trap

    # pop the pushed stack pointer
    popl %esp

    # return falls through to trapret...
.globl __trapret
__trapret:
    # restore registers from stack
    popal

    # restore %ds, %es, %fs and %gs
    popl %gs
    popl %fs
    popl %es
    popl %ds

    # get rid of the trap number and error code
    addl $0x8, %esp
    iret
```

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

    ```
    #define STS_TG32 0xf            
    // 定义常量
    // 为了编程的方便，将一些常量用宏定义的方式赋予名称
    // 提高代码的可读性

    #define le2page(le, member)      to_struct((le), struct Page, member)
    // 将函数封装为宏定义
    // 精简了调用函数的冗长参数
    // 提高了代码的简洁性

    
    #ifndef __BOOT_ASM_H__
    #define __BOOT_ASM_H__
    // ...
    #endif
    // 通过宏定义
    // 避免对头文件的多次调用
    // 高效实现程序
    ```


#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)

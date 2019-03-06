# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

    从磁盘第一个扇区读入的内容是加载程序。因为加载的大小是有限的，只有512字节，所以无法在此时直接读入操作系统的内核映像，而需要引导程序来完成操作系统映像的读入。

- 比较UEFI和BIOS的区别。

    BIOS是固化到计算机主板上的程序，包括系统设置，自检和自启动等程序。而UEFI是一种详细描述全新类型接口的标准，适用于电脑的标准固件接口。UEFI相比BIOS，能够通过保护预启动和预引导进程，抵制bootkit攻击，提高安全性。缩短了启动时间和休眠恢复时间，并支持了容量超过2.2TB的驱动器和64位现代固件设备驱动程序，可以对更大的内存进行寻址。此外BIOS也可以结合UEFI，共同使用。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

    启动过程主要分为两个阶段：
    
    第一个阶段中，将第二个阶段的Bootloader加载到DDR RAM，并管理DDR RAM与IO，内存之间的映射，初始化Rocket。

    第二个阶段中，就要应用到第一阶段中加载的第二阶段Bootloader，并完成IO设备的驱动，完成多核和虚拟机的支持，在虚拟地址空间中加载RISC-V Linux。

    可见BBL的作用在于完成第二阶段的自启动。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

    0x55AA

- x86中在UEFI中的可信启动有什么作用？

    该过程定义了平台固件如何管理安全证书，如何进行固件验证以及定义固件与操作系统之间的接口协议，该过程能够对硬件进行验证，在保证安全性的条件下完成启动。

- RV中BBL的启动过程大致包括哪些内容？



## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

    中断：来自硬件设备的处理请求。

    异常：非法指令或者其他的原因导致当前的指令执行失败后发出的处理请求。

    系统调用：应用程序向操作系统发出的服务请求。

-  中断、异常和系统调用的处理流程有什么异同？

    中断：来自外设，异步响应，持续进行，对用户应用程序透明。

    异常：来自应用程序意想不到的情况，同步响应，杀死或者重新执行意想不到的应用程序指令。

    系统调用：来自应用程序的服务请求，可以同步也可异步响应，处理通常是等待和持续。

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

    系统调用如下所示：
    ```
    static int (*syscalls[])(uint32_t arg[]) = {
        [SYS_exit]              sys_exit,
        [SYS_fork]              sys_fork,
        [SYS_wait]              sys_wait,
        [SYS_exec]              sys_exec,
        [SYS_yield]             sys_yield,
        [SYS_kill]              sys_kill,
        [SYS_getpid]            sys_getpid,
        [SYS_putc]              sys_putc,
        [SYS_pgdir]             sys_pgdir,
        [SYS_gettime]           sys_gettime,
        [SYS_lab6_set_priority] sys_lab6_set_priority,
        [SYS_sleep]             sys_sleep,
        [SYS_open]              sys_open,
        [SYS_close]             sys_close,
        [SYS_read]              sys_read,
        [SYS_write]             sys_write,
        [SYS_seek]              sys_seek,
        [SYS_fstat]             sys_fstat,
        [SYS_fsync]             sys_fsync,
        [SYS_getcwd]            sys_getcwd,
        [SYS_getdirentry]       sys_getdirentry,
        [SYS_dup]               sys_dup,
    };
    ```

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

    系统调用是通向操作系统的接口，是面向底层硬件的。

    函数调用是通向应用程序的库函数的，是面向软件的。

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。

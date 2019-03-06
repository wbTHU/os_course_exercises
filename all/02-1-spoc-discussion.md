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
 + BIOS完成硬件初始化和自检后，会根据CMOS中设置的启动顺序启动相应的设备。但此时，文件系统并没有建立，BIOS也不知道硬盘里存放的是什么，所以BIOS是无法直接启动操作系统。另外一个硬盘可以有多个分区，每个分区都有可能包括一个不同的操作系统，BIOS也无从判断应该从哪个分区启动，所以对待硬盘，所有的BIOS都是读取硬盘的0磁头、0柱面、1扇区的内容，然后把控制权交给这里面的MBR (Main Boot Record）。
 + MBR由两个部分组成：主引导记录MBR和硬盘分区表DPT。在总共512字节的主引导分区里其中MBR占446个字节，一般是一段引导程序，其主要是用来在系统硬件自检完后引导具有激活标志的分区上的操作系统。DPT占64个字节，最多容纳4个分区，每个分区间有2字节的间隔，0x55AA是每个分区结束的标志。
- 比较UEFI和BIOS的区别。
 + UEFI启动对比BIOS启动的优势有三点：
 + 安全性更强：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动；且为保证安全有数字签名校验；
 + 启动配置更灵活：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统；基本上可以做到不依赖于指定的操作系统启动，磁盘插入且通过校验即可启动；
 + 支持容量更大：传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。
- 理解rcore中的Berkeley BootLoader (BBL)的功能。
 + BBL可以完成如下的三个功能：
 + 硬件初始化（如DRAM，串口等）；
 + 将引导参数传递给内核；
 + 加载内核。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？
 + 0x55AA
- x86中在UEFI中的可信启动有什么作用？
 + 通过启动前校验数字签名检查来保证启动介质的安全性；
- RV中BBL的启动过程大致包括哪些内容？
 + 启动过程大致如下：
 + 硬件初始化（如DRAM，串口等）；
 + 将引导参数传递给内核；
 + 加载内核。

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
 + 中断：来自硬件设备的处理请求，也包括外部意外的响应；
 + 异常：指令执行中产生错误导致的意外响应或失效，有些异常是可以预计的；
 + 系统调用：应用程序向操作系统发出的服务请求。
-  中断、异常和系统调用的处理流程有什么异同？
 + 相同：
   + 都会进入异常服务例程，切换为内核态
 + 不同：
   + 源头：中断来源于外设；异常和系统调用来源于应用程序；
   + 响应方式：中断是异步；异常是同步；系统调用既有异步又有同步。
- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
 + ucore的系统调用有SYS_exit,SYS_fork,SYS_wait,SYS_exec,,SYS_clone,SYS_yield,SYS_sleep,,SYS_kill,,SYS_gettime,SYS_getpid,SYS_mmap,
 SYS_munmap,SYS_shmem,SYS_putc,SYS_pgdir,SYS_open,SYS_close,SYS_read,SYS_write,SYS_seek,SYS_fstat,SYS_fsync,SYS_getcwd,
 SYS_getdirentry,SYS_dup
 + 大致可分为：
  + 进程管理：包括 SYS_fork / SYS_exit / SYS_wait / SYS_exec / SYS_yield / SYS_kill / SYS_getpid / SYS_sleep... 命令
  + 文件操作：包括 SYS_open / SYS_close / SYS_read / SYS_write / SYS_seek / SYS_fstat / SYS_fsync / SYS_getcwd / SYS_getdirentry / SYS_dup... 命令
  + 内存管理：SYS_pgdir... 命令
  + 外设输出：SYS_putc... 命令

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
 + 函数调⽤会有一个压栈出栈处理，所有压栈出栈处理由编译器完成。函数调用使用CALL和RET指令；
 + 系统调⽤要完成用户态和内核态的切换，并且做严格的检查，因为有状态切换等，代价较函数调用⼤得多。系统调用使用INT和IRET指令；有堆栈和特权级的转换过程，更为安全；系统调用比函数调用要做更多和特权级切换的工作，所以需要更多的时间开销在一些情况下，如果函数调用采用静态编译，往往需要大量的空间开销，此时系统调用更具有优势。
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
 + iret是中断服务例程的返回调用，根据是否有特权级的改变，弹出EFLAGS 和 SS/ES；
 + ret是程序中函数的返回调用，弹出EIP。
 + 函数调用与系统调用的堆栈操作的不同在于系统调用有 SS：SP 的压栈出栈操作
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
 + jal将地址保存在$sa寄存器中以便执行完成后返回；jalr可以指定使用哪个寄存器保存；
 + ecall是超级用户模式下的系统调用，通过引发环境调用异常来请求执行环境


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。

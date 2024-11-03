## 9.1

已总结-大致

操作系统内存使用情况

top输出几个内存数据-物理内存总量，实际分配内存（used,free）

物理内存会被操作系统用于缓存，但是当内存需求大时，撤回缓存，会导致成本上升。

VIRT虚拟内存地址空间，RES实际使用的物理内存，VIRT>RES，大部分虚拟内存并未占用实际内存--page fault机制，结合按需分配内存技术



## 9.2

已总结-大致

中断的硬件部分

中断时硬件发出的请求信号，具有异步性、并发性

PLIC负责处理外设中断的硬件模块。

操作系统可以设置不同设备中断的优先级

## 9.3

已总结-大致

设备驱动概述

驱动的结构-bottom中断处理程序和top应用接口，驱动程序有缓冲队列，从中读取或写入数据。

中断无法访问进程的page table，所以无法操作用户进程的内存，所以bottom只负责设备层面的处理。

设备的编程方式：Memory-mapped I/O

设备的地址映射，主板上每个设备都有固定的物理地址

设备控制寄存器

中断管理，操作系统和设备之间需要明确的通信协议，确保双方的协调与数据传输的正确性。



## 9.4

已总结-大致

在XV6系统启动后，Shell在Console上显示提示符$的原理--shell将字符传输到UART(通用异步发送接收设备)，UART会产生一个中断。

RISC-V的中断相关寄存器

中断设置代码分析-start函数，main函数，plicinit函数，plicinithart函数，scheduler函数



## 9.5

已总结-大致

UART驱动的Top部分

xv6启动后，init.c中main函数是系统执行第一个进程，负责系统初始化，并创建console设备，赋予文件描述符0，后dup复制fp0，生成标准输出fp1，标准输入fp2。

console作为UART的抽象文件，从应用程序看，就像普通文件，shell会调用fprintf，最终调用write，将‘$’通过fp2写入系统。

write系统调用处理流程，write->sys_write->检查参数，调用filewrite->系统判断文件描述类型，上述为FD_DEVICE，属于设备类型，调用consolewrite函数->通过either_copyin将字符从用户空间拷贝至内核空间，调用uartputc将字符写入UART设备->uartputc会判断buffer状态，将字符‘$’写入Buffer中，调用uartstart函数通知UART设备开始传输数据->检查UART是否空闲，空闲则从Buffer读取数据写入到UART的THR发送到寄存器->发送都按console后，触发中断。

## 9.6

已总结-大致

UART驱动的Bottom部分

UART产生中断->RISC处理：当PLIC将中断带到特定CPU并触发时，SIE寄存器中负责外部中断的E位被清除->保存当前程序计数器PC到SEPC寄存器，以便中断处理后继续执行用户程序->保存当前模式并切换至Supervisor模式->将当前程序计数器设置为STVEC寄存器值，若当前程序在用户空间，则STVEC指向uservec函数，再调用usertrap函数

->调用usertrap检查中断类型,若中断来自外设，则调用plic_claim获取中断号，若获取到中断号为10，则为UART设备，调用uartintr处理UART中断->uartintr从UART的接收寄存器读取数据，传递给consoleintr函数，若无输入，则直接进入uartstart函数

->`uartstart`函数在此阶段将Shell存储在buffer中的字符传输至Console



通过中断机制，UART驱动的top和bottom部分实现解耦，即数据传输（top）和中断处理（bottom）并行执行。

## 9.7

已总结-大致

中断相关的并发

UART和Shell输出字符至Console并发问题：1.设备与CPU的并行，producer-consumer并发 2.中断停止当前运行程序，对于用户空间代码影响不大，但是内核代码可能必须保持原子性，需要临时关闭中断以确保其完整执行。3.UART驱动的top部分和bottom部分可以并行执行。

Producer-Consumer并发模型

buffer位于内存中，所有CPU核通过同一份buffer并行交互，因此需要通过锁机制来确保互斥

条件同步，sleep与wakeup机制

## 9.8

已总结-大致

UART读取键盘输入

shell调用read获取键盘输入时，系统调用fileread函数->fileread检查文件类型，若文件类型为设备，调用设备相应read函数。UART的read函数为console.c中的consoleread函数。

->consoleread函数中的producer-consumer->shell从buffer中读取数据，shell再输出‘$’后进入等待状态sleep->用户再次输入‘l’时，字符通过键盘UART芯片发送至主板，触发UART中断->UART中断处理流程，devintr函数检测来源于UART，调用uartgetc函数获取字符传递至consoleintr函数

->consoleintr通过consputc将字符输出到Console(界面上给用户看)，同时字符存入buffer->当buffer接收到换行符时，会唤醒之前因buffer为空而`sleep`的Shell进程，Shell从buffer中读取数据并继续执行（执行进程）



## 9.9

已总结-大致

Interrupt的演进

早起interrupt处理速度很快，硬件可以以简单的方式工作。设备在需要CPU处理数据时可以直接产生中断，CPU会暂停当前任务处理设备的数据。

->后来处理一个Interrupt需要经过多个步骤，如果设备频繁产生中断，处理器可能难以跟上

->现代设备在产生Interrupt前会先自行处理一部分数据，从而减轻CPU的负担。

->Polling机制，interrupt的替代方案，CPU不断读取设备的控制寄存器（如UART的RHR寄存器）来检查是否有新数据。相比Interrupt，Polling避免了频繁的中断开销，适用于高速设备

->Polling缺陷，Polling会消耗大量的CPU cycles，因为CPU持续检查设备状态而无法执行其他任务。

->Polling与Interrupt的动态切换，对于一些设计精良的驱动，如网络设备驱动中的NAPI（New API），在Polling和Interrupt之间能够动态切换
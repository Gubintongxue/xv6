## 11.1 

已总结，大致

线程（Thread）概述

为什么计算机需要运行多线程？1.多任务需求 2.简化编程需求 3. 提高多核计算性能

线程的抽象：程序计数器，寄存器，栈

多核并行：在多核处理器上，每个CPU核运行一个线程。

单核多线程切换：若CPU核少于线程数，系统在每个CPU上进行线程切换，即让每个线程都在单一CPU核上分时运行。

**共享地址空间的线程**：多个线程在同一地址空间中运行，可以访问和修改共享内存。例如，XV6内核中的多个内核线程共享内核地址空间，因此可以共享数据。共享内存要求在并发访问时使用锁来防止数据冲突。

**独立地址空间的线程**：XV6中的用户进程彼此不共享内存，每个用户进程拥有独立的地址空间和一个线程。多个用户进程各自独立，不会影响对方的数据，这简化了内存管理。



## 11.2

已总结，大致

XV6线程调度

在 `xv6` 操作系统中，线程调度采用了**抢占式调度**机制，通过**定时器中断**来确保 CPU 不会被运算密集型线程长时间占用。每个 CPU 核心都有一个定时器，能够定期触发中断，强制将 CPU 控制权从用户线程转交给内核的调度器，即便线程未主动出让 CPU。

调度器管理线程的状态，包括**运行中（RUNNING）**、**可运行（RUNABLE）**、和**睡眠（SLEEPING）**状态，以有效地分配 CPU 资源。每当中断发生时，系统将当前线程的状态从运行中转换为可运行，并保存它的程序计数器、寄存器等状态到内存。随后，调度器根据系统状态选择下一个线程恢复其状态并继续执行，实现了多线程环境下的高效调度。

这种机制确保了 `xv6` 系统能够公平地调度运算密集型和 I/O 密集型线程，使系统保持流畅的多任务执行效果。



## 11.3

已总结，大致

在 `xv6` 操作系统中，线程切换的流程主要通过**定时器中断**和**系统调用**实现用户空间与内核空间的状态转换。每当用户进程进入内核空间（如发生系统调用或定时器中断）时，系统会首先保存当前用户进程的状态（如程序计数器和寄存器），这些状态被存储在 `trapframe` 中，以便后续恢复。

### 线程切换的核心流程

1. **进入内核空间**：当发生系统调用或定时器中断时，当前用户进程的状态被保存到 `trapframe` 中，随后切换到内核栈以便处理中断或系统调用。
2. **保存当前内核线程状态**：当内核决定进行进程切换时，当前内核线程的寄存器状态会被保存到 `context` 对象中，确保可以稍后恢复。
3. **切换到目标内核线程**：调度器选择一个新的进程，将 CPU 切换到目标进程的内核线程，并从其 `context` 对象中恢复寄存器状态。
4. **完成中断处理并恢复用户进程状态**：新进程的内核线程完成中断或系统调用处理，随后从 `trapframe` 中恢复用户空间状态。
5. **返回用户空间**：CPU 最终返回用户空间，继续执行新的用户进程。



## 11.4

已总结，大致

XV6线程切换（二）
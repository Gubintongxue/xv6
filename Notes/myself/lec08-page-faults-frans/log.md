## 8.1

已总结-大致

page fault引出的高级功能

## 8.2

已总结-大致

lazy page allocation的思想，好处，实现步骤

主要通过增加p-sz，和page fault的触发后处理机制实现

## 8.3

已总结-大致

按需零填充

## 8.4

已总结-大致

COW的概念、思想，实现步骤（与page fault，引用计数有关），区分普通的 page fault 和 COW 引发的 page fault通过RSW bit作为标志位

## 8.5

已总结-大致

按需分页的思想，原理

程序启动时并不预先加载所有内存页，而是根据需要再访问时按需加载。

触发page fault

通过**Access Bit**和**Dirty Bit**，操作系统能够有效地管理页面撤回

## 8.6

已总结-大致

mmap的思想，流程

mmap是系统调用，核心思想是将文件的内容映射到虚拟内存地址--mmap主要还是针对文件，不用write和read

mmap会将文件内容的虚拟地址与物理内存的映射关系记录在VMA，触发page fault后但是会更新page table，改PTE的dirty位。最后unmap释放内存。

多进程下mmap未理解
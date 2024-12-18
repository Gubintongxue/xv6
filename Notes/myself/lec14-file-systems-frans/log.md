# 文件系统

## 14.1 

已总结，大致

为什么有趣

文件系统的特性和吸引力：除了shell之外最常见的用户接口，特性：1.用户有好的文件命令，支持层级路径名结构

2.持久化 3.并发性

文件系统的实现挑战：1.硬件抽象 2.崩溃安全性 3.磁盘结构 4.性能优化



## 14.2 

已总结，大致

文件操作基础系统调用：1.创建文件，open  2.写入数据，write向文件中写入数据，在文件系统内部，必须有一个文件偏移量（offset）来记录写入的具体位置，因为文件描述符会随着写入位置的变化自动更新偏移。

文件系统的核心结构：文件系统内部使用inode对象代表文件

文件系统分成结构：1.底层：磁盘 2.缓存层：buffer cache 3.持久层：logging 4.同步层 inode cache 5.inode层，直接处理read/write操作 6.文件名和描述符层，负责路径查找、文件描述符管理



## 14.3 

已总结，大致

文件系统如何使用磁盘

文件系统的底层依赖存储设备，这些设备类型多样，具有不同的性能、容量及数据保存时间。**最常见的两种存储设备是 SSD 和 HDD**

为了更好地理解文件系统如何与这些设备交互，我们可以区分一些术语：

- **sector（扇区）**：通常是磁盘驱动读写的最小单位，通常为 512 字节。
- **block（块）**：是文件系统视角下的数据单位，大小由文件系统定义。在 XV6 中，block 大小为 1024 字节，即一个 block 包含两个 sector。

在 XV6 中，磁盘被划分成了多个部分，以满足文件系统的需求。

通常情况下，日志块、inode 块、位图块等被称为**元数据块**，因为它们不存储用户数据，而是用于组织和管理文件系统的数据。



## 14.4 

已总结，大致

inode

xv6上inode结构，64字节，包括

- **type 字段**：指示 inode 是文件还是目录。
- **nlink 字段**：即链接计数，用于跟踪有多少个文件名指向当前 inode。
- **size 字段**：表示文件数据的大小（以字节为单位）。
- **direct block numbers**：XV6 中每个 inode 包含 12 个直接块编号，直接指向文件数据所在的块。这些直接块足以存储文件的前 12 个块。例如，如果文件只有 2 个字节，那么只会用到一个 direct block number。
- **indirect block number**：它指向一个包含 256 个块编号的间接块，这 256 个块编号继续指向文件的数据块。这样，文件系统可以通过 indirect block number 访问更大文件的数据。

由于 XV6 的 inode 结构较简单，文件的最大长度是 **268KB**

文件读取示例：文件大小除以块大小得到block number，文件大小对1024取余得到偏移量

xv6中目录结构，目录是一种特殊文件。在 XV6 中是一个文件系统理解的结构，包含固定格式的目录项（directory entries）：

- 前 2 个字节为文件或子目录的 inode 编号。
- 后 14 个字节为文件名或子目录名，每个目录项 16 字节。

路径查找示例：

假设要查找路径名 `/y/x`：

1. 从根 inode（通常是 inode 编号为 1）开始，读取 root inode 的内容。
2. 扫描 root inode 对应的 block，找到 `y` 的目录项并获取 `y` 的 inode 编号（例如 inode 251）。
3. 根据 inode 251 扫描其所有对应的 block，找到 `x` 的目录项并返回文件 `x` 的 inode 编号。

在 XV6 中，**type 字段** 用于标识 inode 类型，确保文件系统能区分目录和普通文件。

XV6 使用简单的线性结构扫描目录名，效率较低。实际文件系统通常使用更复杂的数据结构以加速查找，例如树形结构或哈希表。



## 14.5


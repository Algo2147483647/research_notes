# $Linux$

[TOC]

# CPU

## Commands

**常用的文件、目录命令**

`ls`：用户查看目录下的文件，`ls -a`可以用来查看隐藏文件，`ls -l`可以用于查看文件的详细信息，包括权限、大小、所有者等信息。

`touch`：用于创建文件。如果文件不存在，则创建一个新的文件，如果文件已存在，则会修改文件的时间戳。

`cat`：cat是英文`concatenate`的缩写，用于查看文件内容。使用`cat`查看文件的话，不管文件的内容有多少，都会一次性显示，所以他不适合查看太大的文件。

`more`：more和cat有点区别，more用于分屏显示文件内容。可以用`空格键`向下翻页，`b`键向上翻页

`less`：和more类似，less用于分行显示

`tail`：可能是平时用的最多的命令了，查看日志文件基本靠他了。一般用户`tail -fn 100 xx.log`查看最后的100行内容

**常用的权限命令**

`chmod`：修改权限命令。一般用`+`号添加权限，`-`号删除权限，`x`代表执行权限，`r`代表读取权限，`w`代表写入权限，常见写法比如`chmod +x 文件名` 添加执行权限。

还有另外一种写法，使用数字来授权，因为`r`=4，`w`=2，`x`=1，平时执行命令`chmod 777 文件名`这就是最高权限了。

第一个数字7=4+2+1代表着所有者的权限，第二个数字7代表所属组的权限，第三个数字代表其他人的权限。

常见的权限数字还有644，所有者有读写权限，其他人只有只读权限，755代表其他人有只读和执行权限。

`chown`：用于修改文件和目录的所有者和所属组。一般用法`chown user 文件`用于修改文件所有者，`chown user:user 文件`修改文件所有者和组，冒号前面是所有者，后面是组。

**常用的压缩命令**

`zip`：压缩zip文件命令，比如`zip test.zip 文件`可以把文件压缩成zip文件，如果压缩目录的话则需添加`-r`选项。

`unzip`：与zip对应，解压zip文件命令。`unzip xxx.zip`直接解压，还可以通过`-d`选项指定解压目录。

`gzip`：用于压缩.gz后缀文件，gzip命令不能打包目录。需要注意的是直接使用`gzip 文件名`源文件会消失，如果要保留源文件，可以使用`gzip -c 文件名 > xx.gz`，解压缩直接使用`gzip -d xx.gz

`tar`：tar常用几个选项，`-x`解打包，`-c`打包，`-f`指定压缩包文件名，`-v`显示打包文件过程，一般常用`tar -cvf xx.tar 文件`来打包，解压则使用`tar -xvf xx.tar`。



*Q: CPU负载 vs. CPU利用率？*

如果CPU负载很高，利用率却很低该怎么办？

如果负载很低，利用率却很高呢？

如果CPU使用率达到100%呢？怎么排查？



# Memory

## Paging

## Segmentation

# File System: Ext2

Ext2 (*Second Extended File System*), is a file system type used in Unix-like operating systems.

## Disk Layout: blocks 

Ext2 divides the disk into fixed-size blocks and uses block groups to organize the file system structure. Each block group contains metadata structures such as **inode** tables and data blocks.

<img src="./assets/640-1692199750905-6.jpeg" alt="Image" style="zoom:50%;" />

## inode

The inode (index node) is a data structure in a Unix-style file system that describes a file-system object such as a file or a directory. Each inode stores the attributes and disk block locations of the object's data. File-system object attributes may include metadata (times of last change, access, modification), as well as owner and permission data.

<img src="./assets/640-1692199771197-9.jpeg" alt="Image" style="zoom:50%;" />



<img src="./assets/640-1692199852304-12.png" alt="Image" style="zoom: 33%;" />

<img src="./assets/640-1692199865108-15.jpeg" alt="Image" style="zoom:50%;" />





<img src="./assets/640-1692199065324-3.png" alt="Image" style="zoom:40%;" />

<img src="./assets/640.png" alt="Image" style="zoom: 45%;" />

<img src="./assets/640-1692199971710-18.png" alt="Image" style="zoom: 50%;" />

# IO

一个文件要从磁盘到我们的内存，需要经过很复杂的操作。首先，需要将数据从硬件读取出来，然后放入操作系统内核缓冲区，之后再将数据拷贝到程序缓冲区，最后应用程序才能读取到这个文件。简单地说，无论什么 IO 模型，其读取过程总会经历下面两个阶段：

- 等待数据到达内核缓冲区
- 从内核缓冲区拷贝数据到程序缓冲区

而我们 Linux 根据这两个阶段的是否阻塞，分成了 5 个经典的 IO 的模型，分别是：

- 阻塞 IO 模型
- 非阻塞 IO 模型
- IO 复用模型
- 信号驱动 IO 模型
- 异步 IO 模型






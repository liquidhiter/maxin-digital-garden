---
{"dg-publish":true,"permalink":"/03-es-deep-dive/3-linux-es/","noteIcon":"","created":"2024-05-25T20:16:37.371+02:00","updated":"2024-05-27T22:10:07.170+02:00"}
---

## Term and description
- BSP
	- Board Support Package
	- consists of
		- Bootloader
		- Kernel
		- Rootfs (root file system)
		- Toolchain
- FHS
	- Filesystem Hierarchy Standard
		- 定义了文件系统中目录、文件分类存放的原则、定义了系统运行所需的最小文件、目录的集合，并列举了不遵循这些原则的例外情况及其原因
	- most Linux / Unix distributions follow this standard


## Linux
### `opt`
- reference: https://www.pathname.com/fhs/pub/fhs-2.3.html#OPTADDONAPPLICATIONSOFTWAREPACKAGES
```Markdown
## /opt : Add-on application software packages

### Purpose

/opt is reserved for the installation of add-on application software packages.

A package to be installed in /opt must locate its static files in a separate /opt/<package> or /opt/<provider> directory tree, where <package> is a name that describes the software package and <provider> is the provider's LANANA registered name.

---

### Requirements

|Directory|Description|
|---|---|
|<package>|Static package objects|
|<provider>|LANANA registered provider name|

The directories /opt/bin, /opt/doc, /opt/include, /opt/info, /opt/lib, and /opt/man are reserved for local system administrator use. Packages may provide "front-end" files intended to be placed in (by linking or copying) these reserved directories by the local system administrator, but must function normally in the absence of these reserved directories.

Programs to be invoked by users must be located in the directory /opt/<package>/bin or under the /opt/<provider> hierarchy. If the package includes UNIX manual pages, they must be located in /opt/<package>/share/man or under the /opt/<provider> hierarchy, and the same substructure as /usr/share/man must be used.

Package files that are variable (change in normal operation) must be installed in /var/opt. See the section on /var/opt for more information.

Host-specific configuration files must be installed in /etc/opt. See the section on /etc for more information.

No other package files may exist outside the /opt, /var/opt, and /etc/opt hierarchies except for those package files that must reside in specific locations within the filesystem tree in order to function properly. For example, device lock files must be placed in /var/lock and devices must be located in /dev.

Distributions may install software in /opt, but must not modify or delete software installed by the local system administrator without the assent of the local system administrator.

|   |   |
|---|---|
|![Tip](https://www.pathname.com/fhs/pub/tip.gif)|**Rationale**|
||The use of /opt for add-on software is a well-established practice in the UNIX community. The System V Application Binary Interface [AT&T 1990], based on the System V Interface Definition (Third Edition), provides for an /opt structure very similar to the one defined here.<br><br>The Intel Binary Compatibility Standard v. 2 (iBCS2) also provides a similar structure for /opt.<br><br>Generally, all data required to support a package on a system must be present within /opt/<package>, including files intended to be copied into /etc/opt/<package> and /var/opt/<package> as well as reserved directories in /opt.<br><br>The minor restrictions on distributions using /opt are necessary because conflicts are possible between distribution-installed and locally-installed software, especially in the case of fixed pathnames found in some binary software.<br><br>The structure of the directories below /opt/<provider> is left up to the packager of the software, though it is recommended that packages are installed in /opt/<provider>/<package> and follow a similar structure to the guidelines for /opt/package. A valid reason for diverging from this structure is for support packages which may have files installed in /opt/<provider>/lib or /opt/<provider>/bin.|
```
- [ ] what does `static package` mean? 
	- pre-built ?
- `/opt` looks like a system-wide (not located in `/usr/`) image of `/`, e.x. `/opt/bin` vs `/bin`

### man
- `1`: Linux command
- `2`: system call
- `3`: C standard library

### 系统调用
- system call
	- 隔离内核
- 内核态
	- 驱动程序
- 用户态
	- 应用程序

### C 库函数
- 对系统调用封装而来
	- 易用性
		- 某些系统调用很复杂
	- 库函数通常是有缓存的，而系统调用通常是无缓存的
	- 库函数属于应用层
		- 用户态 -> 内核态
- 库函数相比系统调用具有更好的可移植性，因为很多操作系统都实现了C语言

### 文件I/O基础
- Linux下一切皆文件
	- 硬件设备
- demo
```c
/* POSIX: Primitive System Data Types */
#include <sys/types.h>
/* POSIX: File Characteristics */
#include <sys/stat.h>
/* POSIX: File Control Operations */
#include <fcntl.h>
/* POSIX: symbolic constants */
#include <unistd.h>

int main(void)
{
    char buff[1024];
    int fd1, fd2;
    int ret;

    /* Open the source file to read data from */
    fd1 = open("./src_file", O_RDONLY);
    if (-1 == fd1) {
        return fd1;
    }

    /* Open the destination file to write data in */
    fd2 = open("./dest_file", O_WRONLY);
    if (-1 == fd2) {
        ret = fd2;
        goto out1;
    }

    /* Read 1KB into buffer */
    ret = read(fd1, buff, sizeof(buff));
    if (-1 == ret) {
        goto out2;
    }

    /* Write buffer into the destination file */
    ret = write(fd2, buff, sizeof(buff));
    if (-1 == ret) {
        goto out2;
    }

    /* Nothing has been wrong so far */
    ret = 0;

out2:
    /* Close the destination file */
    close(fd2);

out1:
    /* Close the source file */
    close(fd1);

    return ret;
}
```
- 文件描述符
	- 内核返回给进程的一个非负整数，用来对文件进行索引
	- 一个进程可以打开的文件数数是有限制的，由于内存的限制
		- `ulimit -n`
	- `0、 1、 2` 这三个文件描述符已经默认被系统占用了，分别分配给了`系统标准输入（0）、 标准输出（1）以及标准错误（2)`
		- 标准输入一般对应的是键盘
		- 标准输出和标准错误一般对应的是显示器
```c
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <fcntl.h>

       int open(const char *pathname, int flags);
       int open(const char *pathname, int flags, mode_t mode);
```
- `mode`
	- file access modes
	- ONLY when the flags contains `O_CREAT` or `O_TMP_FILE`
	- `mode_t`: `uint32_t`
		- `[0 : 2]`: other users
		- `[3 : 5]`: group
		- `[6 : 8]`: owner
		- `[9 : 11]`: super
	- use pre-defined macros, such as `S_IRUSR`
- offset
	- at which position the `read` or `write` starts
	- `0` by default
- `close`
	- close an opened file
	- *kernel automatically closes the opening files when a process is being terminated*
	- good practice of explicitly `close` an opened file though
- Linux 文件
	- 硬盘的最小存储单位是*扇区(sector)*
		- 每个扇区存储512字节
	- 文件的最小存取单位是*块(block)*
		- 通常*1*个block由*8*个sector组成
	- inode
		- meta-data of a file
		- 硬盘包含数据区和inode区
			- inode table中存放每一个文件对应的inode
				- 结构体，用来记录文件的多重信息
					- NOTE：文件名不包含在这一结构体中
				- 每一个inode对应唯一的数字index
		- example
			```bash
				╭─    ~/Git/Linux-ES  on   main                                                                                                                                                                                                                                                                                                                           ✔    at 10:34:23 AM 
				╰─ ls -li
				total 20
				254463 -rw-r--r-- 1 maxin maxin  262 May 26 09:24 CMakeLists.txt
				  4803 drwxr-xr-x 2 maxin maxin 4096 May 26 10:06 IO
				254467 -rw-r--r-- 1 maxin maxin  233 May 26 09:47 README.md
				  4800 drwxr-xr-x 4 maxin maxin 4096 May 26 10:11 build
				254603 -rwxr-xr-x 1 maxin maxin  323 May 26 09:43 build.sh
				╭─    ~/Git/Linux-ES  on   main                                                                                                                                                                                                                                                                                                                           ✔    at 10:34:29 AM 
				╰─ stat build.sh
				  File: build.sh
				  Size: 323             Blocks: 8          IO Block: 4096   regular file
				Device: 820h/2080d      Inode: 254603      Links: 1
				Access: (0755/-rwxr-xr-x)  Uid: ( 1000/   maxin)   Gid: ( 1000/   maxin)
				Access: 2024-05-26 09:43:13.436456632 +0200
				Modify: 2024-05-26 09:43:13.426456541 +0200
				Change: 2024-05-26 09:43:13.426456541 +0200
				 Birth: 2024-05-26 09:28:32.993885165 +0200
			```
		- [ ] how the file name is associated with the inode index ?
- 打开文件的过程
	- `open`函数打开文件的时候，内核会申请一段内存，并且将静态文件的数据内容从硬盘中读取到内存中进行管理和缓存
		- 内存中的文件数据拷贝称为动态文件或者内核缓冲区
	- 文件打开后对于这个文件的读写操作都是针对内存中这一份*动态文件*进行操作
	- 内核会将内存中的动态文件更新同步到硬盘设备中
	- 设计原则
		- 存储设备多是基于Flash块设备
			- 读写限制：必须是以块位最小存取单位
		- 内存可以按字节为单位来进行操作
			- 文件缓存在内存使得操作更灵活
		- 内存的读写速率更快
- 进程控制块（Process Control Block，PCB）
![Z - assets/images/Pasted image 20240526110816.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240526110816.png)
	- 文件描述符表
	- 文件表
		- 偏移量
		- 对应的i-node
		- ...
- `errno`
	- Linux系统下对常见的错误都做了一个特殊的编号
	- 每一个进程都维护了自己的`errno`变量，是程序中的全局变量（当前进程/程序），用于存储就近发生的函数执行时发生的错误编号
		- 查看对应的函数的手册来确定`errno`是否被使用
	- example
		```c
		#include <stdio.h>
		#include <errno.h>
		```
- `strerror`
	- "string.h"
- `perror`
	- "stdio.h"
- 终止进程(might be wrong)
	- `_exit()` and `_Exit()`
		- system call
		- `return`执行后把控制权交给调用函数（caller），结束该进程
		- 调用`_exit()`函数会清楚其使用的内存空间，并*销毁其在内核中的各种数据结构，关闭进程的所有文件描述符，并结束进程，将控制权交给操作系统*
	- `exit()` 
		- 终止进程
		- function from C standard library
		- 调用`exit()` 会执行一些housecleaning，最后调用`_exit()`函数
- 空洞文件
	- `lseek`偏移量超过编辑的文件的实际大小
	- 空洞文件产生时不会占用任何物理空间，直到某个时刻对空洞文件部分进行写入数据时才会为其分配对应的空间
		- [ ] 逻辑上该文件的大小是包含了空洞部分大小的？
	- 应用场景
		- 多线程共同操作同一个文件
	- example
		```bash
		ls -lsh ./hole_file.txt
		4.0K -rwxr--r-- 1 maxin maxin 8.0K May 26 15:23 ./hole_file.txt
		```
	- `ls`: returns the logic size of the file
		- `8.0K`
	- `du`: returns the actual size of the file
		- `4.0K`
	- read back from the hole file
		- `0x00`
		- `4.0K -rw-r--r--  1 maxin maxin 4.0K May 26 15:51 hole_bytes.txt`

### NAT
- Network Address Translation
	- 网络地址转换
- 使用NAT网卡时，Ubuntu访问外网是委托Windows发出数据包
- Windows接受到回应后再转发给Ubuntu
- [ ] 建议使用 NAT 网卡 IP，因为使用桥接网卡的话必须启动开发板 ?
- [ ] 桥接网卡 ？


### `/etc/network/interfaces`
- reference: [Understanding and Configuring Linux Network Interfaces | Baeldung on Linux](https://www.baeldung.com/linux/network-interface-configure)
- explanation
```markdown
What Is the /etc/network/interfaces File?
To clarify,  /etc/network/interfaces file is a way to configure network interfaces. It’s mostly used by Linux Debian-like distributions.
The majority of the network setup can be done via this file. We can give an IP address to a network interface either statically or dynamically. Further, we can set up routing information, a DNS server, etc.
Moreover, when we use the network interface management commands, they bring up and down the interfaces and configure them based on the /etc/network/interfaces file: the ifup command brings the network interface up, while the ifdown command takes it down.
```
> configuration file, setup the network between the host and the network


### 网络配置分析
- USB网卡
	- 手动分配windows中IP地址：`192.168.5.10`
	- 用作桥接网卡
		- 开发板连接网络
	- 手动分配ubuntu中IP地址：`192.168.5.11`
- 开发板地址
	- 手动分配：`192.168.5.9`
- 在windows中防火墙将开发板的IP地址添加到白名单
	- reference: [windows系列---【如何给window系统添加ip白名单？】 - 少年攻城狮 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hujunwei/p/14613497.html)
- NFS协议
	- 开发板挂载ubuntu的目录

### Linux内核编译流程
#### 编译内核
- `make mrproper`
- `make 100ask_imx6ull_defconfig`
- `make zImage -j8`
- `make dtbs`
#### 编译内核模块
- `make modules`

### 待了解
- [ ] zImage内核文件
- [ ] 设备树的二进制文件
- [ ] 编译Linux驱动需要先编译内核
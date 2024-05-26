---
{"dg-publish":true,"permalink":"/03-es-deep-dive/3-linux-es/","noteIcon":"","created":"2024-05-25T20:16:37.371+02:00","updated":"2024-05-26T10:45:42.607+02:00"}
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
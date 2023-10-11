## UNIX基础知识

严格意义上，操作系统可被定义为名叫**内核（Kernel）**的软件，其接口被称为**系统调用（System Call）**，公用函数库、应用程序都构建在系统调用接口之上。

### 登录

用户的登录信息一般都保存在`/etc/passwd`中，各用户的登录项一般由7个以冒号分隔的字段组成：
- 如`root:x:0:0:root:/root:/bin/bash`，依次表示：登录名、密码、用户id、组id、注释、home目录、shell程序。

用户登录后即可向shell程序键入命令，UNIX中常见的shell有：
- Bourne shell：即`/bin/sh`，由Steve Bourne在贝尔实验室开发；
- C shell：即`/bin/csh`，由Bill Joy在伯克利开发，支持一些`sh`没有的特色功能；
- Korn shell：即`/bin/ksh`，由贝尔实验室的David Korn开发，兼容`sh`并支持`csh`的特殊功能；
- Bourne-again shell：即目前Linux中最常用的`/bin/bash`，设计遵循POSIX，保持与`sh`的兼容性的同时支持以上两者的特色功能。

### 文件与目录

UNIX文件系统中，所有文件与目录都以根目录`/`为起点。

目录是一个包含目录项的文件。大多数UNIX实现中，文件属性都不会放在目录项中，因为当某个文件具有多个硬链接时，那样难以在多个副本间保持属性的同步。目录或文件的属性都存放在类型为`dirent`的结构体中，可通过`stat`函数进行读取。

创建新目录时都会自动新建两个文件`.`和`..`，分别指向当前目录和其父目录，且根目录中两者都指向当前目录。

`<dirent.h>`中定义了`opendir`、`readdir`等可用来打开、读取目录信息的函数原型。

- 在ubuntu中，头文件被拆分为`/usr/include/dirent.h`和`/usr/include/x86_64-linux-gnu/bits/dirent.h`，前者包含了后者，声明了函数原型，后者则定义了`struct dirent`。

各进程都有一个工作目录，所有相对路径都是从工作目录开始解释。

### 输入和输出

UNIX内核都用**文件描述符**（file description）来标识已打开的文件，它是一个小的非负整数。

运行新程序时，shell都会默认开启三个文件描述符：
- 标准输入（standard input）
- 标注输出（standard output）
- 标准错误（standard error）

I/O有带缓冲和不带缓冲之分：

- `open`、`read`、`write`、`lseek`及`close`都提供不带缓冲的I/O，它们的函数原型和标准I/O文件描述符`STDIN_FILENO`、`STDOUT_FILENO`一起，都定义在`<unistd.h>`；

- C标准库中定义的`printf`、`getc`、`putc`等标准I/O函数，则为不带缓冲的I/O提供了带缓冲的接口。

### 程序和进程

程序是存储在磁盘上某个目录中的**可执行文件**，内核通过`exec`来执行它们。

**进程**（process）是程序的执行实例，UNIX系统的各个进程通过唯一的**进程ID**进行标识，它是一个非负整数。

- 进程ID的数据类型为`pid_t`，可通过`getpid`获得；

- 进程主要通过`fork`、`exec`、`waitpid`这三个函数进行控制。

一个进程可有多个控制**线程**（thread），所有的线程共享进程的地址空间、文件描述符、栈等资源的同时保持独立。

### 错误处理

在`<errno.h>`中定义了整型变量`errno`及可以赋与它的表示各种错误类型的以`E`开头的各种常量。
- 其中定义的各种错误有**致命性**和**非致命性**之分，前者不可恢复。

UNIX系统函数出错时，通常会返回负值或`null`，并把`errno`设定为特定值，多线程环境里每个线程有专属的`errno`以避免相互干扰。
- `strerror`可将参数`errno`映射为一个标识出错信息字符串。
- `perror`则可根据当前的`errno`值，在标准错误上输出错误信息，通常直接将程序名作为实参传入。

对`errno`的设定包含两个规则：
- 没有出错时，当前的程序不会改变其值，因此当且仅当程序的返回值表明程序出错时，才检验其值；
- 表示各种错误类型的各种常量都不为`0`，因此任何函数都不该将`errno`的值设为`0`。

### 用户

用户ID（user ID）用以标识不同的用户的数值，由系统管理员在新增用户是分配，root账户的ID通常为0。

- 可通过`getuid()`获取

组ID（group ID）用以标识用户所属的群组，以实现组成员之间的资源共享。组文件`/etc/group`将组名映射为组ID数值。一个用户允许属于最多16个其他组，由此有了附属组ID（supplement group ID）。

- 可通过`getgid()`获取

### 信号

**信号**（signal）用于通知进程发生了某种状况，每种信号都有各自的命名和编号。进程存在3种信号处理方式：

- 忽略信号
- 系统默认方式
- 捕捉信号，通过函数处理

`kill`函数是产生信号的一种方式，终端键盘上就有最常见的两种信号产生方法：

- 中断：Ctrl+C或Delete
- 退出：Ctrl+\

### 时间

UNIX系统上使用过两种不同是的时间值：

- 日历时间：从协调世界时（UTC，1970/1/1/00:00:00）以来经过的秒数累计数，类型为`time_t`
- 进程时间：也称CPU时间，用以度量进程使用的CPU资源，类型为`clock_t`，

UNIX上共维护了3个进程时间，可通过`time`命令来获取：

- 时钟时间：又称墙上时钟时间（wall clock time），进程允许的时间总量，与系统中同时进行进程数有关
- 用户CPU时间：执行用户指令所用时间量
- 系统CPU时间：为进程执行内核程序所用时间量

### 系统调用与库函数

OS一般都会提供多种服务的入口点，程序由此向内核请求服务，在UNIX中的这些入口点被称为**系统调用**（system call），它们都用C语言定义。

UNIX中为每个系统调用在标准C库中设置了一个同名的函数，用户通过标准C库来进行系统调用。

系统调用通常提供一种最小接口，库函数则提供复杂功能。

## UNIX标准及实现

### ISO C

C语言的ANSI（美国国家标准协会）标准在1989年得到批准，此标准也被采纳为ISO/IEC（国际标准化组织/国际电子技术委员会）标准，简称**”C89“**。其为了提供C程序的可移植性，不仅定义了C语言的语法和语义，还定义了其标准库。ISO C标准在1999年、2011年、2018年又被更新过几次，分别简称**”C99/C11/C17/C2x“**。

按ISO C标准定义的各个头文件将C库分成24个区：

![ISO C头文件](https://raw.githubusercontent.com/Hugsy19/Picbed/master/tech/iso_c_header.png)

### IEEE POSIX

POSIX（Portable Operating System Interface）最初是由IEEE（电气电子工程师学会）为提升应用程序在各种UNIX环境间的可移植性而制定的标准族，编号为1003.1-1988，后面被扩展成很多标记为1003的标准及草案。1003.1标准说明的只是接口而非实现，所以其中不区分系统调用和库函数，所有接口统称为函数。后面正式的版本1003.1-1990递交给了ISO作为国际标准，通常被称为**POSIX.1**，POSIX.1包含了ISO C标准库函数。

### Single UNIX Specification

SUS（Single UNIX Specification）是POSIX.1标准的一个超集，其定义了一些附加接口扩展了POSIX.1规范提供的功能，POSIX.1则相当于是其基本规范部分。SUS是Open Group的出版物，且Open Group是两个工业社团X/Open和开放系统软件基金会（Open System Software Foundation，OSF）合并构成的，POSIX.1中的XSI（X/Open System Interface）选项描述了可选的接口，也定义了遵循XSI的实现必须支持POSIX.1可选部分，只有遵循XSI的实现才能称为UNIX系统。当前较新的规范有2010年发布的SUSv4。

### 限制与选项

各类UNIX系统上运行的程序，在可移植性方面存在以下限制：

- 编译时限制：头文件列出
- 文件/目录无关的运行时限制：`sysconf`函数获取
- 文件/目录相关的运行时限制：`pathconf`、`fpathconf`函数获取

如ISO C定义的所有编译时限制都列在头文件`<limits.h>`中，主要是说明给定的系统的中整型数据的范围。运行时限制则可以通过`<unistd.h>`中定义的以下三个函数之一获得：

- `long sysconf(int name);`：`name`为以`_SC_`开头的常量
- `long pathconf(const char *pathname, int name);`：`name`为以`_PC_`开头的常量，下同
- `long fpathconf(int fd, int name);`

## 文件I/O




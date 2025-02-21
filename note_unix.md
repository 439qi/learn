# Unix 系统  
>
> The Linux Programming Interface.pdf -- Michael KerrisK
>
## 历史

### Unix

1969 年由AT&T的贝尔实验室(Bell Laboratories)的Ken Thompson开发，基于先前AT&T、通用电气和MIT合作的操作系统MULTICS(Multiplexed Information and Computing Service)  

> 在Unix开发过程中，其同事Dennis Ritchie设计了C语言用于编写Unix内核

### BSD

Thompson1975/1976学年作为访问专家任职位于伯克利的母校加利福尼亚大学，并在此期间诞生了Unix变种BSD(Berkeley Software Distribution)

> 还诞生了许多工具，包括C shell，vi等

### GNU与Linux

1. GNU  
    1984年MIT的Richard Stallman出于反对计算机制造商对于操作系统的产权限制——即用户无法获取所购买软件的源代码且无法自由复制、改动、分发这些软件，启动了GNU(GNU's Not Unix)项目  

    > 在此期间诞生了通用公共许可证(GPL, General Public License)，任何使用该许可证的软件必须开放源代码且允许自由分发，且这些软件的改动版本也必须使用GPL许可证

    GNU项目产生了许多Unix-like系统可移植程序，包括Emacs，GCC等，但并没有一个能工作的内核

2. Linux内核  
    荷兰的大学教授Andrew Tanebaum出于教学目的编写了Minix  
    Linus Torvalds基于Minix开发了能运行GNU程序的基础内核
    其余程序员加入并逐渐完善Linux内核

### 标准化

大量具有不同特性的Unix变种使得软件难以移植  

1. C语言标准的提出使得仅使用标准C语言库的程序能够在任意操作系统间移植
2. POSIX(Portable Operating System Interface)标准  
    该标准包含Unix系统调用的C语言接口、shell程序和工具、线程及网络编程等  
    确保源代码级别的可移植性  
    NFY

## 基础概念

### 内核(kernel)

管理和分配计算机资源的核心软件  

#### 内核的任务

- 进程管理
- 进程调度
- 内存管理
- 文件系统
- 外部设备访问
- 网络
- 系统调用API

#### 内核态(kernel mode)和用户态(user mode)

现代处理器架构允许CPU运行在两种模式：用户态和内核态  
相应地，内存被标记为用户空间和内核空间；用户态的CPU仅能访问用户空间，内核态的CPU能访问全部内存空间  
部分操作仅能在内核态执行

### Shell

shell是用于读取用户输入的命令并执行对应程序的专用程序  
有时也称作**命令解释器**(command interpreter)  
但在Unix系统中，命令解释器是内核的一部分，shell是用户进程

shell脚本是包含shell命令的脚本文件  
shell具备编程语言的一些特性以支持shell脚本：变量、循环、条件语句、IO和函数

### 用户和组

#### 用户(Users)

系统的每个用户被独特的用户名和UID(User ID)唯一标识  
用户信息记录在/etc/passwd，其包含

- 用户名
- UID
- 用户所属组的GID
- Home目录
- 登录shell
- 密码  
    出于安全原因，密码不再存储在/etc/passwd，而是存储在仅特权可读的影子密码文件/etc/shadow

#### 组(Groups)

为便于管理，用户可被按组管理  
组信息记录在/etc/group，其包含

- 组名
- GID
- 用户列表

#### 超级用户(Superuser)

超级用户具有系统内的特权，能通过系统的所有权限检查  
超级用户UID为0，用户名通常为`root`

### 单目录层次，目录，链接和文件

Unix内核使用单一层次目录结构来管理文件，即所有文件/目录均为根目录的子节点

#### 文件类型

Unix所有文件都被标识了类型

- **普通数据文件**(ordinary data files)
- **设备**(devices)
- **管道**(pipes)
- **套接字**(sockets)
- **目录**(directories)  
    目录属于特殊的文件，其内容为文件名到文件引用的映射表  
    > 文件名到文件引用的映射称为**链接**(links)，也称为**硬链接**(hard links)  

    每个目录至少包含两个条目
  - `.` ：表示当前目录，即目录自身
  - `..`  : 表示父目录；对于根目录，`..`指向它自身
- 符号链接(symbolic links)  
    也称为**软链接**(soft links)  
    符号链接以文件的形式而非目录内映射表条目的形式存在  
    其内容为另一个文件的文件名，称之为**目标**(target)  
    当符号链接指向的文件不存在时，称之为**空悬链接**(dangling link)

    当系统调用接收到路径名时，会对路径中的每个符号链接递归解引用并替换，直至路径名中不包含任何符号链接  

#### 文件名

大多数Linux文件系统支持最大255字符的文件名  
文件名可以为`/`和`\0`之外的任意字符  
但建议只使用**可移植文件名字符集**(portable filename character set)，`[-._a-zA-Z0-9]`，以及避免使用`-`开头，以避免一些字符在shell、正则表达式等情况下的特殊含义

#### 当前工作目录(current working directory)

每个进程都有其当前工作目录(或简称为当前目录或工作目录)，其表示该进程的当前所在目录  
子进程会继承父进程的当前工作目录  

对于shell，可通过`cd`命令改变当前工作目录

#### 路径名

分为绝对路径和相对路径

- 绝对路径  
    以`/`开头，表示文件相对于根目录的路径
- 相对路径  
    不以`/`开头，表示文件相对于进程的当前工作目录的路径

#### 文件所有权(ownership)和权限(permissions)

每个文件都有一个与其相关联的UID和GID，表明文件所有者和所属的组  

在访问文件的角度，系统将用户分为三类：文件所有者、组的成员、其余用户  
每类用户可设置三种权限  

| 权限| 文件| 目录|
|- |-|-|
| **读**(read)  | 读取文件内容| 读取目录下文件  |
| **写**(write)  | 修改文件内容| 增加、删除、修改文件名 |
| **执行**(execute)  | 执行文件| 将该目录设置为当前工作目录 |

### 文件I/O模型

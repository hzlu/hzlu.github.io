---
title: apue
categories:
  - linux
tags:
  - linux
---


## 文件 I/O

> 函数 open creat close lseek read write

### write read

都在内核执行，称为不带缓冲的 I/O 函数

使用文件系统的块长度作为缓冲区长度可以使得读操作更快。

### 进程共享文件描述符

支持在不同进程间共享打开的文件:

- 进程表项，进程在**进程表**中的一个记录项。记录项含有一张**打开文件描述符表**
  + 文件描述符标志 (close_on_exec)
  + 指向一个文件表项的指针
- 文件表项，内核为所有打开文件维持一张**文件表**。
  + 每个文件表项包含：文件状态标志 （读 写 添写 同步 非阻塞等）
  + 当前文件偏移量
  + 指向 **v 节点表项**的指针
- 节点表项
  + v 节点信息
  + i 节点信息
  + 当前文件长度等其他信息

每个进程都有它自己的文件表项，其中也有它自己的当前文件偏移量。当多个进程写同一个文件时，则可能产生预期不到的结果。

### dup 和 dup2 函数

复制一个现存的文件描述符

```c
#include <unistd.h>

int dup(int filedes);
int dup2(int filedes, int filedes2);
```

这些函数返回的新的文件描述符与参数 `filedes` 共享同一个文件表项。

### sync fsync 和 fdatasync 函数

延时写 **delayed write**
输出方式：当数据写入文件时，内核通常先将数据复制到其中一个缓冲区中，如果该缓冲区未写满，则不会将其排入输出队列，等待其写满或内核需要重用该缓冲区（以便存放其他磁盘块数据）时，再讲缓冲排入输出队列，然后等待其到达队首时才进行实际的 I/O 操作。

延迟写减少了磁盘读写次数，但却降低了文件内容的更新速度，当发生故障时这种延迟写可能造成文件更新内容的丢失，为了保证磁盘上实际文件系统与高速缓存中内容的一致性，UNIX 系统提供了 sync fsync 和 fdatasync 三个函数。

```c
#include <unistd.h>
/*
 * 等待写磁盘操作结束，然后返回 0 或 -1
 * 还会同步更新文件的属性
 */
int fsync(int filedes);

/*
 * 等待写磁盘操作结束，然后返回 0 或 -1
 * 只影响文件的数据部分
 */
int fdatasync(int filedes);

/*
 * 将所有修改过的块缓冲区排入写队列，然后返回
 * 并不等待实际写磁盘操作结束
 */
void sync(void);
```

### fcntl 函数

改变已打开文件的性质。 file control


```c
#include <fcntl.h>

int fcntl(int filedes, int cmd, ...);
```

cmd 取值:

- F_DUPFD 复制FD 新文件描述符有它自己的一套文件描述符标志 FD_CLOEXEC 文件描述符标志被清除
- F_GETFD 获得FD
- F_SETFD
- F_GETFL 获得文件状态标志
- F_SETFL
- F_GETOWN 获得异步 I/O 所有权
- F_SETOWN
- F_GETLK 获得记录锁
- F_SETLK
- F_SETLKW

比较 fsync 和 fdatasync 函数与 O_SYNC 标志：

- fsync 和 fdatasync 函数在我们需要时更新文件内容
- O_SYNC 标志则在我们每次写至文件时更新文件内容

### /dev/fd/n 目录

使用 open 打开任何一个文件，相当于进行 dup 操作一样进行文件描述符的复制


```bash
$ filter file2 | cat file1 - file3 | lpr

# 提高文件名参数的一致性，也更加清晰
$ filter file2 | cat file1 /dev/fd/0 file3 | lpr
```

## 文件和目录

### stat fstat lstat 函数

```c
#include <sys/stat.h>

/* 检测不出符号链接 */
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int filedes, struct stat *buf);

/* 返回该符号链接的有关信息，而不是由该符号链接引用文件的信息 */
int ltat(const char *restrict pathname, struct stat *restrict buf);
```

第二个参数 buf 是指针，它指向一个我们必须提供的结构。


```c
struct stat {
  mode_t st_mode; /* 文件类型及权限 */
  ino_t st_ino; /* i-node 号 */
  dev_t st_dev; /* 设备号 */
  dev_t st_rdev; /* 特殊文件设备号 */
  nlink_t st_nlink; /* 链接数 */
  uid_t st_uid; /* uid */
  gid_t st_gid; /* gid */
  off_t st_size; /* 普通文件大小 */
  time_t st_atime; /* access 时间 */
  time_t st_mtime; /* 修改时间 */
  time_t st_ctime; /* 文件状态最新更改时间 */
  blksize_t st_blksize; /* 最优 I/O 长度 */
  blkcnt_t st_blocks; /* 磁盘块分配数量 */
}
```

### 文件类型

- 普通文件 (二进制可执行文件应该遵循一种格式，以使内核能确定程序文本和数据的加载位置)
- 目录文件
  + (包含了其他文件的名字以及指向这些文件有关信息的指针)
  + 对目录文件具有读权限的进程可以读取该目录的内容
  + 只有内核可以直接写目录文件
  + 进程必须使用特定函数才能更改目录
- 块特殊文件 block special file
  + 提供对设备带缓冲的访问
  + 每次访问以固定长度为单位进行
- 字符特殊文件 character special file
  + 提供对设备不带缓冲的访问
  + 每次访问长度可变
- FIFO
  + 用于进程间通信
  + 有时也称为 命名管道 named pipe
- socket
  + 用于进程间网络通信
  + 也可用于一台宿主机上进程间非网络通信
- 符号链接 symbolic link

对于目录文件的执行权限就是可以在目录中搜索特定的文件名

### 设置用户 ID 和设置组 ID

- 实际用户 ID
- 实际组 ID
- 有效用户 ID
- 有效组 ID
- 附加组 ID
- 保存的设置用户 ID
- 保存的设置组 ID

**设置用户ID位** `set-user-ID` 及**设置组ID位** `set-group-ID` 都包含在 st_mode 值中。这两位可用常量 `S_ISUID` `S_ISGID` 测试。


```bash
chmod u+s foo # 打开设置用户ID位
```

当执行一个程序文件时，进程的有效用户ID通常就是实际用户ID ，有效组ID通常就是实际组ID。但可以在 st_mode 中设置一个特殊标志，其含义是 **当执行此文件时，将进程的有效用户ID设置为文件所有者的用户ID** `st_uid`。与此类似可以设置另一位使得进程的有效组ID 设置为文件的组所有者ID `st_gid`

### 文件访问权限

- 为了在一个目录中创建新文件，必须对该目录具有写权限和执行权限
- 为了删除目录中的一个文件，必须对目录具有写权限和执行权限，对该文件本身不需要读写权限

内核权限测试：

1. 若进程的有效用户ID 是 0 则允许访问
2. 若进程的有效用户ID 等于文件的所有者ID，若所有者适当的访问权限位被设置则允许访问
3. 若进程的有效组ID 或进程的附加组ID 之一等于文件的组ID 那么根据组适当的访问权限位来判断
4. 最后根据其他用户访问权限位进行判断

### 新文件及目录的所有权

新文件的用户ID 设置为进程的有效用户ID
组ID可能是：

- 进程的有效组ID
- 新目录所在目录的组ID 向下传递组所有权

### access 函数


```c
#include <unistd.h>
int access(const char *pathname, int mode);
/* 成功返回0 出错返回-1 */
```

检测访问权限，按**实际** uid 和 gid 来检测而不是进程的**有效** uid 和 gid

### umask 函数

```c
#include <sys/stat.h>

mode_t umask(mode_t cmask);
/* 设置新 mask 返回以前的 mask */
```

`open` 和 `creat` 函数都有一个 mode 参数，它指定新文件的访问权限位

mask 中为 1 的位，在文件 mode 中相应位一定被关闭


```bash
umask -S # 打印符号形式的掩码
```

### 粘住位

sticky bit 在分页机制出来之前的Unix系统，设置粘住位可以使得程序的正文段始终驻留内存中加快运行速度。

后来的Unix版本称它为**保存正文位** saved-text bit 因此也就有了常量 `S_ISVTX` ，现今较新的Unix系统大多数都配置有虚拟存储系统以及快速文件系统，所以不需要使用这种技术。

现今系统扩展了粘住位的使用范围，允许针对目录设置，针对该目录具有写权限的用户满足下列条件之一才能删除或更名目录下的文件：

- 拥有此文件
- 拥有此目录
- 是超级用户

`/tmp` `/var/spool/uucppublic` 是设置粘住位的例子，任何用户都可以在目录中创建文件，但不能删除或更名其他人的文件。

### chown fchown lchown


```c
#include <unistd.h>

int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int filedes, uid_t owner, gid_t group);
int lchown(const char *pathname, uid_t owner, gid_t group);
```

若参数 owner 或 group 任意一个是 -1 则对应的ID不变

如果函数由非超级用户调用，则在成功返回时，该文件的设置用户ID位 和 设置组ID位会被清除

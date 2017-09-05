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



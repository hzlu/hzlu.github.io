---
title: APUE 标准IO库
categories:
  - unix
tags:
  - unix
---

- 不仅仅在UNIX上，很多操作系统都实现了标准I/O库
- 标准IO库处理很多细节：缓冲区分配、优化长度执行I/O等

## 流和FILE对象

- 不带缓冲的I/O函数都是针对 *文件描述符* 的，对于标准IO库，它们的操作都是围绕流进行的
- 用标准IO库打开或创建一个文件，就使得 **一个流** 与 **一个文件** 相关联。
- 文件流的 **字宽** 表示文件流用于 *单字节* 还是 *多字节* 字符集
- **流的定向** 决定了文件流的字宽
- 一个文件流创建时，它没有定向
- 在未定向的流上使用 *多字节IO函数* 则流的定向会设置为 *宽定向*
- 在未定向的流上使用 *单字节IO函数* 则流的定向会设置为 *字节定向*
- freopen 函数清除流的定向
- fwide 函数设置流的定向

### fwide 函数

> mode 不会改变已定向流的定向

```c
#include <stdio.h>
#include <wchar.h>

int fwide(FILE *fp, int mode);

/* 流是宽定向返回正值 */
/* 流是字节定向返回负值 */
/* 流是未定向返回0 */
```

- mode 为负，fwide 试图使流字节定向
- mode 为正，fwide 试图使流宽定向
- mode 为0，fwide 不设置流的定向，但返回标识该流定向的值
- fwide 无出错返回，只能先清除通过检查 errno的值

### fopen函数

fopen 返回一个指向FILE对象的指针，FILE对象通常是一个结构，包含标准IO库管理流的信息

- 用于实际IO的文件描述符
- 指向用于该 *流缓冲区* 的指针
- 缓冲区的长度
- 当前缓冲区中的字符数
- 出错标志

为了引用一个流，将FILE指针作为参数传递给标准IO函数，FILE对象的指针称为 *文件指针*

## 标准输入 输出 出错

进程预定义的三个流，三个预定义文件指针，定义在头文件 <stdio.h> 中

- stdin
- stdout
- stderr

## 缓冲

### 缓冲的目的

- 尽可能减少使用 read 和 write 函数的次数
- 对每个IO流自动进行缓冲管理，避免应用程序需要考虑这点

### 缓冲类型

- 全缓冲 填满标准IO缓冲区后才进行实际IO操作
  + 调用 malloc 获得需要使用的缓冲区
  + 冲洗 flush 说明标准IO缓冲区的 *写操作*
  + 自动冲洗（写满时）或 调用函数 fflush 冲洗
- 行缓冲 在输入和输出中遇到换行符进行IO操作
  + 流涉及一个终端时（例如标准输入和标准输出）通常使用行缓冲
  + 当填满缓冲区，即使没有写一个换行符也进行IO操作
  + 从不带缓冲的流或一个行缓冲的流得到输入数据，那么就会造成冲洗所有行缓冲输出流
- 不带缓冲 标准IO库不对字符进行缓冲存储
  + 标准错误流 stderr通常是不带缓冲的

### 改变缓冲类型

```c
#include <stdio.h>

void setbuf(FILE *restrict fp, char *restrict buf);
int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
```

- 这些函数需要在已打开的流上调用
- 并应该在对流执行其他操作前调用
- setbuf 函数buf参数指向一个长度为 `BUFSIZ`的缓冲区，就是带缓冲进行IO （系统设置全缓冲还是行缓冲）
- setbuf 函数buf参数设置为NULL 就是关闭缓冲
- setvbuf 精确指定缓冲类型，根据mode参数指定
  + _IOFBF 全缓冲 (buf是NULL 则自动为流分配合适长度的系统缓冲区)
  + _IOLBF 行缓冲 (buf是NULL 则自动为流分配合适长度的系统缓冲区)
  + _IONBF 不带缓冲（忽略buf和size参数）

有些实现缓冲区的一部分用于存放它自己的管理操作信息，所以可以存放在缓冲区中的实际数据字节数会 **少于size**

### fflush 冲洗流


```c
#include <stdio.h>

int fflush(FILE *fp);
```

一个特例：当fp是 NULL 时，则此函数会导致**所有输出流**被冲洗。

## 打开流

```c
#include <stdio.h>

FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
FILE *fdopen(int filedes, const char *type);
```

- fopen 打开一个指定的文件
- freopen 在指定流上打开一个指定的文件
  + 如果流已经打开，则先关闭流
  + 如果流已经定向，则清除该定向
  + 一般用于将一个指定的文件打开为预定义的流（标准输入 标准输出 标准出错）
- fdopen 使标准的IO流与现有的文件描述符相结合
  + 常用于由创建管道和网络通信通道函数返回的描述符（因为这些特殊类型的文件不能用标准IO fopen函数打开）

type参数

- r 或 rb 读
- w 或 wb 文件截短为0 或为写而创建
- a 或 ab 添加；在文件尾写
- r+ 或 r+b 或 rb+ 读和写
- w+ 或 w+b 或 wb+ 把文件截短为0，读写
- a+ 或 a+b 或 ab+ 在文件尾读写

字符b 作为type的一部分，使得标准IO可以区分文本文件和二进制文件
unix内核不区分这两种文件

- 如果有多个进程用标准IO添写方式打开同一个文件，那么来自每个进程的数据都将正确地写到文件中

### fclose 关闭一个流

```c
#include <stdio.h>
int fclose(FILE *fp);
```

文件被关闭前，冲洗缓冲区中的输出数据。丢弃缓冲区中的任何 *输入数据*
一个进程正常终止时，则所有带未写缓冲数据的标准IO都会被冲洗，所有打开的标准IO都会被关闭

## 读和写流

- 每次一个字符的IO
- 每次一行的IO
- 直接IO （常用于二进制文件中读或写一个结构）

### 输入函数

```c
#include <stdio.h>
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);

/* 返回下一个字符 */
/* 达到文件尾或出错返回EOF */
```

> getchar 等价于 getc(stdin)

- fgetc 一定是一个函数，不能实现为宏，允许将其地址作为一个参数传送
- getc的参数不应当具有副作用的表达式
- 调用fgetc所需时间很可能长于调用getc，因为调用函数通常所需时间长于调用宏

### 区分出错还是到达文件尾

每个流在FILE对象中维持两个标志：

- 出错标志
- 文件结束标志

```c
#include <stdio.h>

int ferror(FILE *fp);
int feof(FILE *fp);

void clearerr(FILE *fp);
```

### ungetc 将字符再压送回流中

```c
int ungetc(int c, FILE *fp);
```

- 当已经到达文件尾端时，仍可以回送一个字符，下次读将返回这个字符，再次读则返回EOF
- ungetc函数会清除流的文件结束标志

### 输出函数

```c
int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
```

- putchar(c) 等效于 putc(c, stdout)
- putc 可实现为宏
- fputc不能实现为宏

## 每次一行I/O

```c
char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf);
```

- gets从标准输入读
- fgets从指定的流读
- 指定缓冲区的地址，读入的行送入其中
- fgets 指定缓冲区的长度 n
- 一直读到下一个换行符为止，但不超过n-1个字符
- 缓冲区以null字符结尾
- gets不推荐使用，因为不能指定缓冲区长度，可能造成缓冲区溢出，写到缓冲区后面的存储空间


```c
int fputs(const char *restrict str, FILE *restrict fp);
int puts(const char *str);
```

- 将一个以null符终止的字符串写到指定流，但null不写出
- 不要求null字符之前一定是换行符，通常null符之前是一个换行符
- puts会自动在最后添加一个换行符

## 标准IO的效率


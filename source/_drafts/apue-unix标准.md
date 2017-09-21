---
title: APUE Unix标准
categories:
  - unix
tags:
  - unix
---


## UNIX 标准

三个主要标准

- ISO C
- IEEE POSIX
  + POSIX 指的是可移植的操作系统接口
  + portable operating system interface
  + 标准的目的是提高应用程序在各种Unix系统环境之间的可移植性
- Single UNIX Spectification
  + 单一Unix规范
  + 是 POSIX.1 标准的一个超集
  + 相应的系统接口全集被称为 X/OPEN系统接口 XSI
  + 只有遵循 XSI 的实现才能称为 UNIX系统

## UNIX 实现

主要实现

- FreeBSD
- Linux
- Mac OS X
  + 核心操作系统称为 Darwin （基于 Mach 内核和 FreeBSD操作系统的组合）
- Solaris
  + 基于SVR4

## 限制

- 编译时限制
- 运行时限制

编译时限制可在头文件中定义，程序在编译时可以包含这些头文件。
但是，运行时限制则要求进程调用一个函数以获得此种限制值。

## 基本系统数据类型

在头文件 `<sys/types.h>` 中定义了某些与实现有关的数据类型，称为**基本系统数据类型**

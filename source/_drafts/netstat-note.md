---
title: netstat 命令详解
category: linux
tags: [netstat]
---

### `netstat`结果分为两部分

- `Active Internet connections` 有源TCP连接
- `Active UNIX domain sockets` 有源Unix域套接口（和网络套接字一样，但只能用于本机通信，性能提高一倍）

### 各字段含义

- `Proto` 连接使用的协议
- `Recv-Q` 指接收队列，一般为0
- `Send-Q` 指发送队列，一般为0
- `State` 当前状态
- `RefCnt` 表示连接到本套接口上的进程id
- `Types`表示套接口的类型
- `Path` 表示连接到套接口的其他进程使用的路径名

### 常用选项

|选项|作用|
|--|--|
|`-a`|显示所有结果，不加这参数是看不到LISTEN相关的|
|`-t`|仅显示tcp相关结果|
|`-u`|仅显示udp相关结果|
|`-n`|不现实别名，能显示数字的都显示数字，比如ssh会显示成22|
|`-p`|显示建立相关连接的程序名<br>会多出一个字段`PID/Program name`|
|`-l`|仅列出有在监听LINTEN的服务状态|
|`-r`|显示路由信息，路由表|
|`-e`|显示扩展信息|
|`-s`|按各个协议统计|
|`-c`|每隔一个固定时间更新信息|
|`-i`|显示网络接口列表|
### 实例

```bash
netstat -a # 列出所有端口（包含未监听）
netstat -at # 列出所有tcp端口
netstat -au # 列出所有udp端口
netstat -l # 列出所有处于监听状态的Sockets
netstat -lt # 列出所有在监听的tcp端口
netstat -lu # 列出所有监听的udp端口
netstat -lx # 列出所有监听UNIX端口
netstat -st # 显示tcp端口的统计信息
netstat -i # 显示网络接口列表
netstat -ie # 类似ifconfig，显示详细信息
netstat -nat | grep "192.168.1.15:22" |awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -20
# 查看连接到某服务端口最多的IP地址
netstat -nat |awk '{print $6}' # TCP各种状态列表
netstat -tnp # 列出开放端口与服务，常用选项组合
```


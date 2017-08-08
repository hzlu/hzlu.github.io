---
title: AWK 使用
category: linux
tags: [awk]
---

- awk语句只能被**单引号包含**
- `$1`..`$n` 表示第n列，`$0`表示整行，默认以**空格**或**tab**分隔

```bash
awk '{print $1, $4}' file
```

## 格式化输出，类似C语言的`printf`

```bash
awk '{printf "%-8s %-8s %-15s\n", $1, %2, %3}' file
```

## 条件过滤

```bash
awk '$3==0 && $6=="LISTEN"' file
awk '$3>0 {print $0}' file
awk '$3==0 && $6=="LISTEN" || NR==1' file # 内建变量NR表示表头
```

## 内建变量

|内建变量|含义|
|--|--|
|FS|输入分隔符，默认为空格或tab|
|OFS|输出字段分隔符|
|NF|当前记录有多少列（字段数）|
|NR|行号，从1开始，多个文件会累加|
|FNR|各文件自己的行号|
|RS|输入记录的分隔符，默认为`\n`|
|ORS|输出的记录分隔符|
|FILENAME|当前输入文件的名字|

## 指定分隔符

```bash
awk 'BEGIN{FS=":"} {print $1, $3, $6}' file
awk -F: '{print $1, $3, $6}' file
awk -F '[;:]' '{print $1, $3, $6}' file # 指定多个分隔符
awk -F: '{print $1, $3, $6}' OFS="\t" file # 指定多个分隔符
```

## 字符串匹配

```bash
awk '$6 ~ /pattern/ || NR==1 {print NR, $4, $5}' OFS="\t" file
awk '/pattern/' file
awk '!/pattern/' file # 模式取反
awk '$6 ~ /pattern1|pattern2/ {print $4, $5}' file
```

## 拆分文件

根据某一列使用重定向符号

```bash
awk 'NR!=1 {print > $6}' file
awk 'NR!=1 {print $4, $5 > $6}' file
```

## 使用环境变量

将外部变量值传递给awk，借助`-v`选项，或放在语句块后面

`-v`和`ENVIRON` （ENVIRON的变量需要export）

```bash
awk -v val=$x '{print $1, ENVIRON["y"}' file
```

放在`BEGIN``{}``END`语句块后面

```bash
awk '{print v1, v2}' v1=$var1 v2=$var2 filename
```

## 从awk中读取命令输出

通过使用`getline`将外部shell命令的输出读入变量cmdout

```bash
"cmd" | getline cmdout;
echo | awk '{ "grep root /etc/passwd" | getline cmdout; print cmdout }'
```

## 在awk中使用循环

```bash
for(i=0;i<10;i++) { print $i; }
for(i in array) {print array[i]}
```

## `awk`脚本

脚本`cal.awk`

```bash
#!/bin/awk -f
# 运行前
BEGIN {}
# 运行中
{}
# 运行后
END {}
```

执行脚本

```bash
awk -f cal.awk data_file
```

或

```bash
./cal.awk data_file
```



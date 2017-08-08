---
title: Shell 脚本编程
category: linux
tags: [shell]
---

## fork_bomb

- **Fork BOMB** `:() { :|:& };:`
- 常量通常用大写字母表示，变量通常用小写字母表示。
- `true`是作为`/bin`中一个二进制文件来实现的，使用shell 内建的`:`命令，它总返回为`0`的退出码，更快。
- 运行命令知道执行成功的函数

```bash
repeat () { while :; do $@ && return; sleep 5; done }
```

## 函数

shell函数是位于其他脚本中的微脚本，定义`shell函数`**两种语法**：

```bash
function name {
    commands
    return
}

name () {
    # 函数名和括号之间有空格
    local foo # 定义局部变量用local
    foo=1
    commands
    $0 # 在函数中$0表示函数名
    $1 # $1表示调用函数时传递的变量
    return
}
```

函数定义时需要圆括号，**但如果没实参，则调用时不能带上括号**。

##流程控制

```bash
if [ $x = 5 ]; then
    echo 'yes'
elif commands; then
    commands...
else
    echo 'no'
fi
```

- 条件判断相等单双等号都可以，**等号两边有空格**。
- 函数和脚本执行完毕会发送给系统一个`退出状态`，这个值从0-255，`0表示成功`，其他都是失败。
- `$?`该参数用来检查上一个命令的退出状态。
- `if`语句做的事情其实就是**计算命令执行成功或失败**，即计算`$?`是否为`0`！！！
- 脚本中末尾可以使用exit命令，接收可选退出状态数作为参数，表示脚本的退出状态。遇到exit，脚本执行就会结束返回。

## 测试

`[ expression ]` 与 `test expression` 是等价的。

### 算术比较

```bash
[ $var -eq 0 ]
[ $var -ne 0 ]
# 逻辑与 -a 逻辑或 -o
[ $var1 -ne 0 -a $var2 -gt 2 ]
```

|操作符|含义|
|--|--|
|`-eq`|等于|
|`-ne`|不等于|
|`-gt`|大于|
|`-lt`|小于|
|`-ge`|大于等于|
|`-le`|小于等于|

### 文件系统相关测试

|操作符|含义|
|--|--|
|`-f`|正常的文件路径或文件名|
|`-e`|文件存在|
|`-x`|文件可执行|
|`-d`|判断目录|
|`-c`|判断字符设备文件|
|`-b`|块设备文件|
|`-w`|文件可写|
|`-r`|文件可读|
|`-L`|是一个符号链接|

```bash
fpath="/etc/passwd"
if [ -e $fpath ]; then
	echo file exists;
else
	echo does not exists;
fi
```

### 测试字符串表达式

最好使用双中括号`[[]]`

|操作符|含义|
|--|--|
|`-z`|空字符串|
|`-n`|非空字符串|


[[]] 命令是更现代的测试版本，支持正则匹配=~ 支持路径名展开==
if [[ $FILE == foo.* ]]; then

(()) 用来执行算术真测试，如果算术结果非零，算术真测试为真。在双括号内只处理整数，因此能通过名字识别出变量而不需要展开操作。
if (( (( INT % 2 )) == 0 )); then

逻辑操作符

AND

-a

&&

OR

-o

||

NOT

!

!


read int
read var1 var2 var3

read -a array

输入赋值到数组array中

read -e

使用Readline的方式编辑输入

read -r

反斜杠不解释为转义字符

read -u fg

使用文件描述符输入，不是标准输入

IFS=“:” read user pw id god name home shell <<< “$file_info"

临时改变环境变量赋值
<<<操作符指示一个here字符串
这里不能使用管道的原因是管道线会创建子shell，子shell永远不能改变父进程的环境，当子shell退出，子shell做的改变在父进程里都看不到。

利用source来执行脚本是在父进程中执行的。点号执行，直接执行，或sh执行会创建子进程执行。

循环while/until
while commands; do
    xxx
done
break 跳出循环
continue 执行下一次循环

until [ xxx ]; do
    xxx
done < foo.txt

foo.txt | while read var; do
    xxx
done

流程控制case
case $REPLY in
    0) commands1 ;;
    1) commands2 ;;
    a|A) echo “可以使用竖线字符作为分隔符把多个模式结合起来，或条件"
       ;;
    [[:alpha:]]) xxx ;;
    *) default ;;
esac

添加;;&语法允许case语句继续执行下一条测试而不是简单终止运行。

for循环
花括号展开：
for i in {A..D}; do
    echo $i
done

路径名展开：
for i in foo*.txt; do
    echo $i
done

命令替换
for i in $(strings file); do
    len=$(echo $i | wc -c)
    xxx
done
strings 能为文件生成一个可读的文本格式的words列表，用于for循环

C语言格式
for (( i=0; i<5; i=i+1 )); do
  echo $i
done

位置参数

$0

命令行中出现的第一个单词，也就是执行的程序名

$1...

第一个参数，第二个参数

$#

命令行参数个数

${10}

第十个参数要用花括号括住

shift命令
提升位置参数

所有的位置参数向左shift掉一个，shift出队列，$#减1，$0不变，$1的值变成$2的值

集体处理位置参数

$*
"$*"

不用引号，展开成从1开始的位置参数列表
展开成由IFS分隔开位置参数的字符串
"word words with spaces"

$@
"$@"

不用引号和$*一样
展开成引号引起来的分开的字符串，
"word" "words with spaces"

脚本调试
执行脚本时添加选项

n

不执行，只检查语法

v

执行前打印脚本内容

x

跟踪命令展开

可以对脚本中部分区域进行追踪而不是整个脚本
command1
set -x  # 开始追踪
command2
set +x  # 关闭追踪
command3

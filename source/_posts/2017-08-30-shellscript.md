---
title: Shell 脚本编程
tags:
  - shell
categories:
  - 命令行工具
date: 2017-08-30 15:44:31
---


## 特殊变量

变量 | 含义
-- | --
`$?` | 上一个命令的退出状态
`$$` | 当前shell进程ID
`$0` | 在函数中表示函数名
`$1` `$2` ... `${10}` | 调用函数时传递的变量 注意花括号包围
`$#` | 命令行参数个数
`$*` `$@` | 全部展开成列表
`"$*"` | 变量 `IFS` 分隔开位置参数的**字符串** *一个整体*
`"$@"` | **位置参数列表** "$1" "$2" ... "$n"

示例

```bash
ubuntu@ubuntu-xenial:~$ ./demo "a" "b" "c and d"
$*= a b c and d
"$*"= a b c and d
$@= a b c and d
"$@"= a b c and d
print each param from $*
a
b
c
and
d
print each param from $@
a
b
c
and
d
print each param from "$*"
a b c and d
print each param from "$@"
a
b
c and d
```

## shell参数替换

### 模式替换

模式替换 | 描述
-- | --
`${var/pattern/string}` | 替换一次
`${var//pattern/string}` | 全部替换

```bash
var = hello1 world1 hello2 world2
${var/hello/olleh} # => olleh1 world1 hello2 world2
${var//hello/olleh} # => olleh1 world1 olleh2 world2
```

### 模式删除

模式删除 | 描述
-- | --
`${var#pattern}` | 从开头删除最小匹配的子串
`${var##pattern}` | 从开头删除最大匹配子串
`${var%pattern}` | 从结尾删除最小匹配子串
`${var%%pattern}` | 从结尾删除最大匹配子串

```bash
var1 = abcd12345abc6789
Number of characters in abcd12345abc6789 = 16

pattern1 = a*c
${var1#$pattern1} # => d12345abc6789
${var1##$pattern1} # => 6789

pattern2 = b*9
${var1%pattern2} # => abcd12345a
${var1%%pattern2} # => a
```

### 字符数计算

```bash
# 返回变量var的字符串长度
echo ${#var}
```

## 带冒号的参数替换

```bash
${var:-word} # 如果var未设置，则整个参数替换表达式值为word
${var:+word} # 如果var设置了，则整个表达式的值为word
${var:=word} # 如果var未设置，则表达式值为word，变量var的值同时也设为word
${var:?word} # 如果var未设置则打印错误，如果有设置则表达式值为var原来的值
${var:offset:length} # var变量切片，生成子串
string=abcdefghijklmn...
echo ${string:4} # => efghi...
echo ${string:4:8} # => efghijkl
echo ${string:(-2):2} # => yz
```

### 打印匹配的变量名


```bash
${arr[*]} # 或
${arr[@]} # 数组全部元素
${arr[1]} # 第一个元素
${#arr[*]} # 数组长度
```

## 变量展开

```bash
var="hello world"
$var # 等价 ${var}
${var} # 等价 $var
"$var" # 可以放在双引号中

${#var} # 获得字符串长度
${var:+exp} # 如果变量var有值且不为空，则使用exp的值
${var:+'HELLO WORLD'} # 返回"HELLO WORLD"

num1=1
num2=2
$[ num1 + num2 ] # [ ] 操作符用于计算，类似let命令不需要使用$
$[ $num1 + $num2 ] # 或者使用$也是可以的
$((num1 + num2)) # $(()) 操作符
$(($num1 + $num2)) # 变量前$不是必需的

# 子 shell
result=$(expr 1 + 1) # $()
result=`expr 1 + 1` # 子 shell 的另一种写法

# 把子 shell 放入双引号中，会保留空格和换行符
out="$(cat file)"

array=(1 2 3 4 5)
${array[$index]} # 数组取值
${!array[*]} # 获取关联数组的键列表对应普通数组的索引列表
${!array[@]} # 等价
${array[*]} # 获取关联数组的值列表，对应普通数组的值
${#array[*]} # 数组长度
```

## shift命令

提升位置参数

所有的位置参数向左 `shift` 掉一个，`shift` 出队列，`$#` 减1，`$0` 不变，`$1` 的值变成 `$2` 的值

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

# 函数名和括号之间有空格
name () {
  local foo # 定义局部变量用local
  foo=1
  commands
  return
}
```

函数定义时需要圆括号，**但如果没实参，则调用时不能带上括号**。

## 流程控制

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
- 函数和脚本执行完毕会发送给系统一个`退出状态`，这个值从0-255，**0表示成功**，其他都是失败。
- `$?` 该参数用来检查上一个命令的退出状态。
- `if` 语句做的事情其实就是**计算命令执行成功或失败**，即计算`$?`是否为`0`！！！
- 脚本中末尾可以使用 `exit` 命令，接收可选退出状态数作为参数，表示脚本的退出状态。
- 遇到 `exit`，脚本执行就会结束返回。

## 测试

`[ expression ]` 与 `test expression` 是等价的。

### 算术比较

```bash
[ $var -eq 0 ]
[ $var -ne 0 ]
[ $var1 -ne 0 -a $var2 -gt 2 ] # 逻辑与 -a 逻辑或 -o
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

最好使用双中括号`[[ ]]`

|操作符|含义|
|--|--|
|`-z`|空字符串|
|`-n`|非空字符串|


`[[ ]]` 命令是更现代的测试版本，支持正则匹配 `=~` 支持路径名展开 `==`

```bash
if [[ $FILE == foo.* ]]; then
```

`(( ))` 用来执行算术真测试，**如果算术结果非零，算术真测试为真**。
在双括号内只处理整数，因此能通过名字识别出变量而不需要展开操作。

```bash
if (( (( INT % 2 )) == 0 )); then
```

### 逻辑操作符

- `AND` `-a` `&&`
- `OR` `-o` `||`
- `NOT` `!`

### read

读取键盘输入

```bash
read int
read var1 var2 var3

read -a array #输入赋值到数组array中

read -e # 使用Readline的方式编辑输入

read -n 5 # 只读取5个输入字符

read -p "提示输入"

read -s # 输入密码使用的静默输入

read -r # 反斜杠不解释为转义字符

read -u fg # 使用文件描述符输入，不是标准输入

IFS=":" read user pw id god name home shell <<< "$file_info"
# 临时改变环境变量赋值
<<< # 操作符指示一个here字符串
```

这里不能使用管道的原因是管道线会创建子shell，子shell永远不能改变父进程的环境，当子shell退出，子shell做的改变在父进程里都看不到。

- 利用 `source` 来执行脚本是在父进程中执行的。
- 点号执行，直接执行，或sh执行会创建子进程执行。

### 循环while/until

```bash
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
```

### 流程控制case

```bash
case $REPLY in
  0) commands1 ;;
  1) commands2 ;;
  a|A) echo “可以使用竖线字符作为分隔符把多个模式结合起来，或条件"
    ;;
  [[:alpha:]]) xxx ;;
  *) default ;;
esac
```

添加 `;;&` 语法允许 `case` 语句继续执行下一条测试而不是简单终止运行。

### for循环

花括号展开

```bash
for i in {A..D}; do
  echo $i
done
```

路径名展开

```bash
for i in foo*.txt; do
  echo $i
done
```

命令替换

```bash
for i in $(strings file); do
  len=$(echo $i | wc -c)
  xxx
done
```
strings 能为文件生成一个可读的文本格式的words列表，用于for循环

C语言格式

```bash
for (( i=0; i<5; i=i+1 )); do
  echo $i
done
```


## 脚本调试

执行脚本时添加选项 | 作用
-- | --
`n` | 不执行，只检查语法
`v` | 执行前打印脚本内容
`x` | 跟踪命令展开

可以对脚本中部分区域进行追踪而不是整个脚本

```bash
command1
set -x  # 开始追踪
command2
set +x  # 关闭追踪
command3
```

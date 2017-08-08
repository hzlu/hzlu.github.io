---
title: Vimscript 语法小结
category: devtools
tags: [vim]
---

通过内置的帮助系统读取Vim自带的Vimscript文档`:help vim-script-intro`

查看特殊符号完成列表`:help keycodes`

```vim
let i = 1
while i < 5
  echo "count is" i
  let i += 1
endwhile

for i in range(1, 4)
  echo "count is" i
endfor
```

### 三种数值

- 十六进制
- 八进制
- 十进制

### 变量

变量名由字母、数字、下划线组成，不能以数字开头（大部分语言都这样啦）

变量**类型**一旦分配就会保持不变，因此不能给一个变量名重新赋值为其他类型。

```vim
let g:name = 1 " 定义全局变量
let s:name = 1 " 定义脚本文件的局部变量
let w:name " 窗口的局部变量
let t:name " 编辑器选项卡的局部变量
let b:name " 缓冲区的局部变量
let l:name " 当前函数局部变量
let a:name " 当前函数参数变量
let v:name " Vim预定义的变量
```

列举当前定义的所有变量使用命令`:let`

删除一个变量`:unlet s:name`（当变量不存在时报错）
删除一个可能不存在的变量`:unlet! s:name`

```vim
" 当脚本结束时，它的局部变量在下一次脚本执行时仍可用
if !exists("s:call_count")
" if not exists("s:call_count")
  let s:call_count = 0
endif
let s:call_count = s:call_count + 1
echo "called" s:call_count "times"
```

### 布尔判断

非0为真，0为假，`if "true"` 为假，因为`"true"`会解读为0**（当一个字符串看起来不像数值时，就会被当作0处理）**

### 表达式

- `$NAME` 环境变量
- `&name` Vim选项
- `&l:name` 本地Vim选项
- `&g:name` 全局Vim选项
- `@varname` 一个Vim注册器


### 算术

可以用`.`点把两个字符串连接起来

```vim
:echo "foo" . "bar"
" 一般echo命令遇到多个参数时会在参数之间加上空格，但上面的例子中参数是一个表达式，所以没有空格
:echo i > 5 ? "i is big" : "i is small"
```

### 条件语句

```vim
if {condition}
    {statements}
elseif {condition}
    {statements}
else
    {statements}
endif
```

### 模式匹配

```vim
if str =~ " "
    echo "字符串包括空格"
endif
if str !~ '\.$'
    echo "字符串不以句号结束"
endif
```

- `#`表示大小写敏感，`!~#` 表示检查一个模式是否匹配时考虑大小写
- `?`表示忽略大小写，`==?` 表示比较两字符串是否相等时不考虑大小写

### 循环

```vim
while counter < 40
    call do_something()
    if skip_flag
        continue " 重新循环
    endif
    if finished_flag
        break " 结束循环
    endif
    sleep 50m " sleep 50毫秒，不带m表示秒
endwhile " 结束循环
```

### 执行一个表达式

```vim
:execute "tag get_cmd" " 表示要跳到标签get_cmd
```

- `:execute` 只能用来执行**冒号命令**
- `:normal` 可以用来执行**普通模式命令** `:normal gg=G`(跳到第一行并以`=`操作符排版所有行)
- `eval()`函数，执行参数作为表达式计算的结果

```vim
:execute "normal Inew text \<Esc>"
:let optname = "path"
:let optval = eval('&' . optname) " 得到的返回值就是`path`选项的值
" 或使用exe
:exe 'let optval = &' . optname
```

### 使用函数

`:function` 列出函数清单
`:help function-list` 访问内置函数分类列表
`:delfunction Name` 删除函数Name

```vim
:call search("Date: ", "W")

:let line = getline(".") " 从当前缓存区获取光标所有行文本
:let repl = substitute(line, '\a', "*", "g")
:call setline(".", repl)
" 相当于
:substitute/\a/*/g
```

#### 定义函数

函数名必须大写字母开头，要在函数内部访问一个全局变量必须加上`g:`前缀

```vim
:function Min(num1, num2)
    " a:前缀表示该变量是一个函数参数
    if a:num1 < a:num2
        let smaller = a:num1 " smaller 是一个函数内部的局部变量
    else
        let smaller = a:num2
    endif
    return smaller " 返回给调用者，没带参数则返回0
:endfunction
```

重新定义一个已存在的函数使用`function!`

#### 带行范围执行call

可在定义函数时使用关键字`range`，不过没有给出关键字Vim将把光标移动到范围内的每一行并对该行调用此函数。

```vim
:function Count_words() range
:  let lnum = a:firstline
:  let n = 0
:  while lnum <= a:lastline
:    let n = n + len(split(getline(lnum)))
:    let lnum = lnum + 1
:  endwhile
:  echo "found " . n . " words"
:endfunction
:10,30call Count_words()
```

#### 函数不定参数

```vim
function Show(start, ...)
    {body}
endfunction
```

- `a:0`表示参数的个数
- `a:000` 变量表示所有`...`参数的列表
- `a:1`表示第一个**可选**参数
- `a:2`类推

#### 字典函数

```vim
let uk2nl = {'three': 'drie', 'four': 'vier', 'one': 'een', 'two': 'twee'}
function uk2nl.translate(line) dict
  return join(map(split(a:line), 'get(self, v:val, "???")'))
endfunction
:echo uk2nl.translate('three two five one')
" dire twee ??? een
```

`self` 局部变量引用该字典。
`get()`函数检查某键是否在字典存在，如果是提取对应的值，如果不存在返回缺省值。

### 注释

`:map :abbreviate :execute !`等命令之后不能有注释，但可以使用`|`分隔两个命令，在后一个命令写注释。

在缩写`abbrev`和映射`map` 命令直到行尾或者`|`字符为止的内容都认为是有效的，因此不要随意在后面加空格。

`:exe '!ls *.c'  |" list C files`

### 续行机制

```vim
let s:save_cpo = &cpo " 保存compatible选项
set cpo&vim

iabbrev otehr
    \ other

let &cpo = s:save_cpo " 恢复compatible选项
unlet s:save_cpo
```

### 映射

```vim
map <unique> <Leader>a <Plug>TypercorrAdd
```

`<unique>`使得Vim在映射已经存在时给出错误信息。

检查映射是否存在

```vim
if !hasmapto('<Plug>TypecorrAdd')
  map <unique> <Leader>a <Plug>TypecorrAdd
endif
inoremap <Esc> <nop>
" no operation
nnoremap <buffer> <leader>x dd " 该映射只在定义它的那个缓冲区有效
```

- `nmap` normal-mode key mapping
- `nunmap -` 删除-键的映射
- `nnoremap` 非递归映射

### 缩写iabbrev

`iabbrev`替换时机：在insert模式下输入非字母、数字、下划线的字符就会触发abbreviation替换。

```vim
:setlocal wrap " 缓冲区设置
```

### 自动命令

```vim
:autocmd BufNewFile * :write
" BufNewFile 要监听的事件
" * 用于事件过滤的模式
" 要执行的命令
```

`:help autocmd-events`

- `BufWritePre`
- `BufRead`
- `BufNewFile`
- `FileType` 该事件会在Vim设置一个缓冲区`filetype`的时候触发

```vim
:autocmd FileType javascript nnoremap <buffer> <localleader>c I// <esc>
:autocmd BufWrite * :echom "writing buffer!"
```

查看消息日志`:messages`

#### 把自动命令放到组中

Grouping Autocommands

```vim
:augroup testgroup
:   autocmd BufWrite * :echom "Foo"
:   autocmd BufWrite * :echom "Bar"
:augroup END
```

当多次使用`augroup`的时候VIM每次都会组合那些组，要想清除一个组，可以把`autocmd!`命令放到组里

```vim
:augroup testgroup
:   autocmd!
:   autocmd BufWrite * :echom "Bar"
:augroup END
```

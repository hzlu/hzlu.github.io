---
title: IO类
type: notes
order: 6
category: Ruby
---

向标准输出（IO对象）写入的数据会显示在屏幕中。

把输出结果重定向到文件时，标准输出的内容会被写入到文件，只有标准错误输出的内容被输出到屏幕中。

STDIN STDOUT STDERR 都是与控制台关联的，但使用重定向或管道会发生变化，可以通过`tty?`方法判断IO对象是否与控制台关联。

`TeleTYpe`

File类是IO类的子类。

打开文件的模式有：

* r 默认只读
* r+ 读写
* w 只写，文件不存在创建文件，文件存在清空文件重新写
* w+ 读写
* a 追加模式，文件不存在创建文件
* a+ 读写/追加模式，文件不存在就创建

`io.each_byte`方法逐个字节读取io中的数据，将得到的字节所对应的ASCII码以整数值的形式传递。

`io.getc` 方法只读取一个字符。要注意与`io.each_char`的区别。

* gets
* each
* each_line
* readlines
* lineno
* lineno=
* each_char
* each_byte
* getc
* ungetc
* getbyte
* ungetbyte
* read

输出操作

* io.puts
* io.putc
* io.print
* io.printf(fmt,arg0,arg1, ...)
* io.write(str) 输出参数str指定的字符串。方法的返回值是输出的字节数。

###文件指针
file pointer文件指针和 current file offset 当前文件偏移量来表示IO对象指向的文件的位置。

`io.pos` 获取文件指针当前的位置。

`io.pos=(position)` 改变指针的位置。

`io.seek(offset, whence)`
其中whence的值有：

* IO::SEEK_SET 文件指针移到offset的位置
* IO::SEEK_CUR 根据当前位置来偏移
* IO::SEEK_END 相对于文件末尾的偏移位置

`io.rewind` 文件指针返回开头

`io.truncate(size)` 截断文件，按size大小截断。

换行符以行为单位做输入输出处理，需要换行符转换时称为文本模式，不需要转换时称为二进制模式。
新的IO对象默认是文本模式，使用`.binmode`方法可以把它变为二进制模式，注意，转换是不可逆的。

###buffering

强制输出缓冲中的数据使用`io.flush`

设置`io.sync = true` 程序写入缓冲时，flush方法就会自动调用。进行同步输出。

使用`IO.popen(command, mode)`与命令行进行交互，mode模式与File.open的一样，也就是`r r+ w w+ a a+` 这些。

IO对象的输出会作为命令的输入，命令的输出则会作为IO对象的输入。

带有管道符号的命令传给open方法的效果与IO.popen方法一样。

`open("|command", mode)`注意是方法没有接收者。

###StringIO对象

`require "stringio"`

模拟IO对象的对象，对StringIO对象进行输出并不会输出到任何地方，而是保存在对象中。可以用read来读

另外一种用法就是将字符串数据作为IO数据处理

```
require "stringio"
io = StringIO.new
io.puts("A")
io.puts("B")
io.rewind
p io.read

io2 = StringIO.new("A\nB\n")
p io2.gets
```

---
title: 《理解 Unix 进程》笔记
category: ruby
tags: [linux, ruby]
---

## 基础知识

1. Unix系统组成：用户空间`userland`与内核`kernel`。
2. 内核是处在硬件之上的层，内核管理硬件，但程序不可直接访问内核，而是通过**系统调用**来进行通信。

## 进程都有进程指示符`pid`

内核把所有的进程都看成数字，这个数字就是进程指示符。

~~~ruby
puts Process.pid  #在irb中执行，打印当前irb进程的进程ID
~~~

Ruby的`Process.pid`封装了**系统调用**`getpid`，Ruby也有一个全局变量`$$`保存了当前进程的指示符。

~~~bash
$ ps -p <pid-of-irb-process>
# 在命令行中执行，替换pid，打印刚才的irb进程
~~~

## 父进程

系统中运行的每一个进程都有**父进程**。父进程标识符`ppid`。
大多数情况下，进程A的父进程就是调用A进程的进程。比如，`iTerm`进程是`zsh`进程的**父进程**，在zsh中执行`ls`命令，`ls`进程的**父进程**就是`zsh`进程。

~~~ruby
puts Process.ppid # 打印当前进程的父进程id
~~~

Ruby的`Process.ppid`是对应系统调用`getppid`。

## 文件描述符

**文件描述符代表打开的文件**。

在一个进程中打开一个文件（设备，管道，socket等）就会分配一个文件描述符，文件描述符在不相关的进程间**不会共享**，进程退出后，进程中的资源文件就会关闭。

在Ruby中，`IO`类描述了打开的资源。任意一个IO对象都有一个相关联的**文件描述符编号**，可以使用`IO#fileno`进行访问。

~~~ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno
~~~

`内核`通过唯一的数字标识**跟踪**`进程`所用的**资源**。只是用来跟踪打开的资源，已关闭资源没有文件描述符。

1. 文件描述符优先分配最小可用的整数，
2. 如果之前打开的文件关闭了，那么这个文件占用的描述符就会空出来随时准备分配。

~~~ruby
# 当尝试读取一个关闭的文件的描述符时，会引发异常
passwd = File.open('/etc/passwd')
passwd.close
puts passwd.fileno    #IOError: closed stream
~~~

### 标准流

每个unix进程都分配3个打开的资源：

1. **STDIN** `0` 标准输入
2. **STDOUT** `1` 标准输出
3. **STDERR** `2` 标准错误

~~~ruby
puts STDIN.fileno
puts STDOUT.fileno
puts STDERR.fileno
~~~

> 对应的系统调用指令有`open close read write pipe fsync stat`可以通过`man`指令获得详细文档 `man 2 open`

## 进程有资源限制

~~~ruby
p Process.getrlimit(:NOFILE)
# 返回一个二元数组，比如[1024, 4096]
~~~

### 软限制`soft limit`

> 软限制并不是真正的限制，当进程打开文件超过软限制时会引发异常**Error::EMFILE**，但你可以更改这个限制。

**使用场景**：httperf进程需要创建5000个网络连接，这个时候就需要修改`soft limit`。或者使用第三方代码库需要限制它打开的资源。

~~~ruby
# 把限制都设成4096 => [4096, 4096]
Process.setrlimit(:NOFILE, 4096)

# 把软限制提高到硬限制水平
Process.setrlimit(:NOFILE, Process.getrlimit(:NOFILE)[1])
~~~

### 硬限制`hard limit`

> 硬限制只能通过超级用户修改，有兴趣的可以参考`$man 8 sysctl`。

注意设置hard limit的过程是不可逆的，设低了就不能调高了。

### 其他的资源限制

~~~ruby
# 当前用户所允许的最大并发进程数
Process.getrlimit(:NPROC)

# 可创建的最大文件
Process.getrlimit(:FSIZE)

# 进程栈的最大段的大小
Process.getrlimit(:STACK)
~~~

相关的封装的系统调用指令
~~~bash
$ man 2 getrlimit
$ man 2 setrlimit
~~~


## 进程的环境变量

每个进程都从它们各自的父进程中**继承**了环境变量。注意，环境变量具有**`全局性`**！

~~~bash
$ MESSAGE='wing it' ruby -e "puts ENV['MESSAGE']"
~~~

在bash中可以通过`VAR=value`的语法设置环境变量，通过`$VAR`获取环境变量的值。而在Ruby中则从ENV常量获取或设置环境变量：

~~~ruby
ENV['MESSAGE'] = 'wing it'
system "echo $MESSAGE"
~~~

> Ruby的`ENV` 不是哈希，它实现了`Enumerable`和部分`Hash API`

### 环境变量实际应用

环境变量通常被用于**把输入传递到命令行程序中**的通用方法，用环境变量作为命令行程序的输入比使用*选项*`开销更低`。

~~~bash
$ RAILS_ENV=production rails server
$ QUEUE=default rake resque:work
~~~

> 系统没有提供相应的系统调用去操作环境变量，但有C库函数可以`setenv(3) getenv(3) environ(7)`

## 传递给进程的参数

每个进程都能访问一个特殊的数组`ARGV`（或叫`argument vector` 参数向量）,它保存着在命令行中传递给进程的参数。在Ruby中它就是一个数组对象。

~~~ruby
ARGV.include?('--help')
#获取-c选项的值
ARGV.include?('-c') && ARGV[ARGV.index('-c') + 1]
~~~

## 进程名

在Ruby中可以通过全局变量`$PROGRAM_NAME`（别名`$0`）来**获取**或**修改**当前进程名，改变进程名没什么意义。

~~~ruby
$PROGRAM_NAME
$PROGRAM_NAME = 'fuck name'
~~~

## 进程有退出码

退出码范围（`0-255`）除了`0`以外，其他退出码都表示**异常退出**，退出码是进程通信的方式之一。

Ruby的退出方式`Kernel#exit`，最简单的进程退出方式，返回退出码`0`，也是脚本执行结束的默认退出方式。

### `Kernel#exit`

~~~ruby
# 退出码0 正常退出
exit
# 进程退出前调用
at_exit {puts 'last!'}
exit
# 返回退出码22
exit 22
~~~

### `Kernel#exit!`

~~~ruby
# 返回默认退出码1，并且退出前不会执行at_exit块中的代码。
exit!
exit! 33  #返回退出码33
~~~

### `Kernel#abort`

- abort提供了从错误进程中退出的通用方式
- 传递的消息会在进程退出前打印到STDERR
- 返回默认退出码1
- abort退出会调用at_exit块

~~~ruby
abort
abort "something wrong"
at_exit {puts 'last!'}
abort
~~~

### `raise`

- 不会马上使进程结束，但如果没有rescue处理这个异常，进程就会退出返回退出码1。
- at_exit块会被调用，异常信息打印到STDERR

~~~ruby
raise 'hell'
~~~

## 进程衍生`forking`

1. 衍生出的子进程继承了父进程所有的**内存拷贝**
2. 子进程**随意更改**内存内容而不会对父进程造成影响
3. 也继承了父进程的**文件描述符**
4. 也获得了父进程的所有文件描述符的**编号**
5. 进程间就共享了这些打开的文件或sockets，以便**进程间通信**
6. 子进程是一个全新的进程，有自己的**pid**
7. 衍生**调用很快**，几乎瞬间完成。
8. 衍生可能造成**内存过载**

~~~ruby
puts "parent pid is #{Process.pid}"
if fork
  puts "enter if from #{Process.pid}"   #父进程执行
else
  puts "enter else from #{Process.pid}" #子进程执行
end
~~~

> 在子进程中`fork`返回`nil`，父进程中`fork`返回创建的子进程`pid`。

~~~ruby
# 父进程跳过块，块在子进程中执行，执行后子进程会退出。
fork do
  xxx
end
~~~

> 系统调用`$ man 2 fork`

## 孤儿进程

- 当父进程退出后，**子进程未退出**，子进程就变成了孤儿进程
- 父进程退出后，子进程照常运行，父进程不会带着子进程同归于尽
- **守护进程是孤儿进程**，有意让守护进程安静的保持运行
- 可以通过`Unix信号`与脱离终端会话的进程进行通信，从而管理孤儿进程。

## 写时复制`CoW`

CoW，copy-on-write，fork出的子进程如果直接物理拷贝开销会很大，于是现代的unix系统提出了一个CoW的概念，延迟内存拷贝，直接它需要对内存进行写操作。所以父子进程会共享同一片物理内存数据，直到它们当中某一个进程需要改变内存数据。其他没变的内存数据还是共享的。

但不幸的是**MRI**或**Rubinius**并不支持CoW，因为MRI的垃圾回收使用`mark-and-sweep`标记清除算法，当GC时，会迭代每个对象并写入信息。fork之后，当第一次GC，写时复制带来的好处会被撤销。

> Ruby企业版`REE`是`CoW`友好的。

## 进程等待`Process.wait`

**阻塞调用**，会让父进程等待它**任意一个**子进程退出，然后才会继续执行下去。该方法**返回值**是退出的子程序`pid`，表示等到的退出的子程序。

~~~ruby
Process.wait
~~~

### `Process.wait2`

返回`pid`和`status`，`status`是一个`Process::Status`实例对象，包含大量有用信息。

~~~ruby
# Process.wait2
# ==
fork do
  exit 111
end
pid, status = Process.wait2
puts status.exitstatus  #返回111
# ==
# 进程间通信就不需要依赖文件系统和网络！
~~~

### `Process.waitpid`

等待**特定**的子进程

~~~ruby
Process.waitpid <pid>
Process.waitpid2 <pid>

# 等同于 Process.wait
Process.waitpid -1

# 等同于 Process.waitpid 111
Process.wait 111
~~~

~~~ruby
favourite = fork do
  exit 77
end
middle_child = fork do
  abort "I want to be waited on!"
end
pid, status = Process.waitpid2 favourite
puts status.exitstatus
~~~

### 竞争条件

> 问：当处理退出进程的代码还在运行时，有别的子进程退出，会发生什么？
> 答：内核会把退出进程的信息放入队列，所以父进程能**依次**接收子进程的退出信息。

如果没有子进程执行时去`wait`会触发异常`Errno::ECHILD`，所以要记录好创建了多少个子进程。

关注子进程的做法是一个很普遍的UNIX编程模式，通常叫做**看顾进程**、`master/worker` 或者`preforking`。
核心概念是，一个主进程fork出多个子进程来并行，主进程负责管理子进程，确保子进程响应或者对子进程退出做出回应。

> 系统调用 `$man 2 waitpid`

## 僵尸进程

内核会一直**保留**已退出的子进程的**状态信息**，直到父进程使用`Process.wait`请求这些信息。
如果父进程一直不请求这些状态，内核就不会回收资源。任何子进程如果在结束时父进程仍在运行，那么这个子进程很快就会变成**僵尸进程**。

如果你不打算使用`Process.wait`来等待某个子进程退出，那么你需要分离`detach`这些子进程。以免变成**僵尸进程**。

~~~ruby
pid = fork do
  xxx
end
Process.detach(pid)
~~~

`detach`的工作就是生出一个新的线程，**唯一的任务就是等待这个特定的子进程结束**。`detach`就保证了内核不会保留不需要的状态信息。

`$ ps -ho pid,state -p <pid of zombie>`

## Unix信号

通过捕获`:CHLD`信号，当有子进程退出时，父进程会被内核通知。

~~~ruby
trap(:CHLD) do
  puts Process.wait
end
~~~

### 信号并发传递

**信号传递是不可靠的**，当代码在处理CHLD信息时，有其他子进程退出，你可能会也可能不会收到第二个CHLD信号。

要正确处理`CHLD`，必须在一个循环中调用`Process.wait`，查找所有已经结束的子进程。

要避免`Process.wait`阻塞，可以使用wait方法的第二个参数，告诉内核如果没有子进程退出，就不需要进行阻塞。

> `Process.wait(-1, Process::WNOHANG)`

~~~ruby
child_processes = 3
dead_processes = 0
# 衍生出3个子进程
child_processes.times do
  fork do
    sleep 3
  end
end

$stdout.sync = true

trap(:CHLD) do
  begin
    while pid = Process.wait(-1, Process::WNOHANG)
      puts pid
      dead_processes += 1
      exit if dead_processes == child_processes
    end
  rescue Errno::ECHILD
  end
end

loop do
  (Math.sqrt(rand(44)) ** 8 ).floor
  sleep 1
end
~~~

### 信号基本概念

信号是一种**异步通信机制**。当进程从内核那里接收到信号时，它可以做以下事情：

1. 忽略信号
2. 执行特定操作
3. 执行默认操作

信号从一个进程发送到另一个进程，借助内核作为中介。

信号最初的设计目的是为了以不同的方式去终结进程。以下代码是一个进程给另一进程发信号，用`Process.kill`

~~~ruby
# 进程一
puts Process.pid
sleep

# 进程二
Process.kill(:INT, <pid-of-first-process>)
~~~

### 重新定义信号的行为

~~~ruby
trap(:INT) {print "nananana"}
trap(:INT, "IGNORE") # 忽略信号
~~~

现在我们的进程就算收到INT信号也不会退出。

只有`SIGKILL`信号是不会被重定义的。一般我们还是**不要重定义这些信号**，要重定义，就重定义`SIGUSR1`和`SIGUSR2`信号

重定义信号的行为是全局性的，因此要小心不要影响到其他代码。

> 系统调用
> `$ man 2 sigaction`
> `$ man 7 signal`

## 进程间通信

**进程间通信**，`IPC`, `Inter-process communication`
常用的方式是管道`pipes`和套接字对`socket pairs`

### 管道

管道是单向数据流，一个进程拥有管道的一端，另一进程拥有另一端。

如果不关闭`writer`端，`reader`端会不断尝试读取数据，直到读到`EOF`，在这之前reader会**阻塞**。

~~~ruby
reader, writer = IO.pipe
# => [#<IO:fd 5>, #<IO:fd 6>]

writer.write("Into the pipe I go")
writer.close
puts reader.read
~~~

###共享管道

~~~ruby
reader, writer = IO.pipe
fork do
  # 子进程中写数据
  reader.close
  10.times do
    writer.puts "xxx"
  end
end

writer.close
# 父进程读数据
while message = reader.gets
  $stdout.puts message
end
~~~

由于文件描述符会拷贝，现在就有4个实例，只要两个用于通信，另外两个就需要关闭。

注意这里用`gets`和`puts`，在管道中传递的是数据流`stream`，有特定的分隔符`\n`，当读取数据流时，每次只会读取一截，以分隔符分隔。

### `SOCKET`通信

可以使用`消息`代替流进行通信，但无法在管道中使用消息，不过在`Unix套接字`中就可以。

`Unix套接字`是套接字的一种类型，它只能在同一台物理机器上进行通信。因此比`TCP套接字`要快很多。

~~~ruby
# 创建一对可以通过*消息*来通信的Unix套接字
# 一对已经互相连接好的套接字，不使用流，而是使用数据报通信，写入整个消息，无需分隔符
require 'socket'
Socket.pair(:UNIX, :DGRAM, 0)
# => [#<Socket:fd 15>, #<Socket:fd 16>]
~~~

管道提供的是单向通信，套接字对提供的是双向通信。

~~~ruby
require 'socket'
child_socket, parent_socket = Socket.pair(:UNIX, :DGRAM, 0)
maxlen = 1000
fork do
  parent_socket.close
  4.times do
    instruction = child_socket.recv(maxlen)
    child_socket.send("#{instruction} accomplished", 0)
  end
end
child_socket.close
2.times do
  parent_socket.send("Heavy", 0)
end
2.times do
  parent_socket.send("Feather", 0)
end
4.times do
  $stdout.puts parent_socket.recv(maxlen)
end
~~~

> 系统调用
`$man 2 pipe`
`$man 2 socketpair`
`$man 2 recv`
`$man 2 send`

## 守护进程

首个进程`init`进程，`pid`为`1`，就是一个**守护进程**。它的`ppid`为`0`

任意进程都能变成守护进程，这里拿rack项目为例

~~~ruby
def daemonize_app
  if RUBY_VERSION < "1.9"
    # 父进程退出，调用脚本的终端会认为命令已经结束
    # 返回控制权给终端。
    # 子进程变成了孤儿进程，孤儿进程的PPID为1
    exit if fork

    # 使衍生进程成为一个新进程组和会话组的leader
    # 新的会话组就不会有控制终端了
    # 只能从子进程调用，如果进程已经是组长，调用会失败
    Process.setsid

    # 控制终端只能分配给会话领导，再次衍生的进程就绝不会有
    # 确保脱离控制终端，独立运行，守护进程
    exit if fork

    # 确保守护进程的工作目录不会消失
    Dir.chdir "/"

    # 把标准输入输出干掉，因为不需要，但不能直接close
    # 其他程序还是需要它们的
    STDIN.reopen "/dev/null"
    STDOUT.reopen "/dev/null", "a"
    STDERR.reopen "/dev/null", "a"
  else
    # 1.9版本后才附带的方法，把当前进程变成守护进程
    Process.daemon
  end
end
~~~

> 衍生的进程虽然摆脱了终端，但依然继承了gid和sid，终端依然有办法能给fork出的进程发信号。但我们需要我们的进程彻底摆脱（`detach`）终端。`Process.setsid`会使得当前进程成为**新的**进程组和session组的leader。（leader能被终端发送信号）
>
> 变leader之后需要再fork一个新的子进程出来，然后leader退出，最后这个子进程（既没有控制它的终端，也不是leader）就是需要的守护进程了！！！

### 调用`Process.setsid`的作用

1. 该进程变成一个新会话的会话领导
2. 该进程变成一个新进程组的组长
3. 该进程没有控制终端

### 进程组和会话组

#### 进程组

1. 每个进程都有一个所属的**组**，每个组都有唯一的id（通常是`leader`进程的`pid`，也是在终端里输入命令启动的进程id）
2. 一个进程组是一些相关进程的集合，通常是父进程和它的子进程们的集合。
3. 也可以任意组合进程组
4. **终端**接收信号，会将信号转发给**前台进程组**中的**每个进程**，因此如果在终端发送信息，进程组里的所有进程都会受影响。

~~~ruby
# 通常这个组id会跟组leader进程的pid相同
Process.getpgrp
# 设置进程的组ID
Process.setpgrp(new_group_id)
~~~

> 系统调用`$ man 2 getpgrp`

#### 会话组

会话组是更高一级的抽象，是进程组的集合。

~~~bash
git log | grep shipped | less
~~~

这些指令创建了3个独立的进程组，但ctrl-c还是能一次性把它们都干掉，因为它们处于同一个会话组。在`shell`中的每**一次调用**都会获得自己的`会话租`（**一次调用**可以是单个命令，也可以是由管道连接的一串命令）

> 系统调用 `$man 2 getsid` 能获取当前的会话组ID
> 但Ruby没有相应的接口，使用`Process.setsid`会新建一个会话组并返回会话组ID

## 生成终端进程

Ruby程序中一个常见的交互是在程序中通过shelling out的方式在终端执行某个命令，这在编写Ruby脚本来将若干常用命令粘合在一起时尤为常见。（在Ruby中生成进程来执行终端命令）

exec命令能让你把当前进程替换成一个不同的进程。你可以把当前的Ruby进程转换成一个Python进程或者一个`ls`进程。exec替换进程后是不会返回到原来的进程的。因此需要联合使用`fork+exec`。专门fork出一个进程来exec，这时候可以使用之前的`Process.wait`来等待退出码。

`exec`不会关闭文件，也不做内存清理，以下代码是在Ruby中打开文件，然后生成一个Python进程去读取这个文件，在Ruby中打开了，在Python中就不需要再次打开了

~~~ruby
hosts = File.open('/etc/hosts')
exec 'python', '-c', "import os; print os.fdopen(#{hosts.fileno}).read()"
~~~

> `$man 2 exec`

### exec的参数
参数可以是`字符串`，也可以是`数组`，之间有微妙的不同，字符串参数表示它会启动一个`shell进程`，然后把字符串参数传给shell去解析；数组参数的话，它会跳过shell进程，而直接把数组作为新进程的`ARGV参数`。

推荐使用数组参数，字符串参数有安全问题。

### `Kernel#system`

`Kernel#system`返回`true`或者`false`，当终端命令退出码是0是，方法返回true。
生成的进程与当前进程共享标准流（stdout stdin stderr），但会**阻塞调用**

~~~ruby
system('ls')
system('ls', '--help')
~~~

### 反引号执行

反引号返回值和`system`不同，它是标准输出返回转字符串的形式。
反引号与`%x`作用相同

~~~ruby
`ls`
`ls --help`
%x[git log | tail -10]
~~~
### `Process.spawn`

`Process.spawn` 非阻塞调用，返回`shelling out`进程的PID

~~~ruby
Process.spawn({'RAILS_ENV' => 'test'}, 'rails server')
system 'sleep 5'  #阻塞
Process.spawn 'sleep 5'   #非阻塞

pid = Process.spawn 'sleep 5'
Process.waitpid(pid)
~~~

### `IO.popen`

`IO.popen`是用纯Ruby实现的unix管道，在底层，它还是在做`fork+exec`的事情，不过它同时也建立**管道**来跟**生成的进程**`通信`。

~~~ruby
# IO对象stream传进代码块中，打开stream进行写入。
# 将stream设置成生成进程的STDIN
IO.popen('less', 'w') do |stream|
  stream.puts "some\ndata"
end

# 因此这个流设置成了派生进程的标准输入，如果打开流来读，那么流就是派生进程的标准输出。你只能选一端
~~~

### `Open3.popen3`

~~~ruby
require 'open3'
Open3.popen3('grep', 'data') { |stdin, stdout, stderr|
  stdin.puts "some\ndata"
  stdin.close
  puts stdout.read
}
Open3.popen3('ls', '-uhh', :err => :out) { |stdin, stdout, stderr|
  puts stdout.read
}
~~~

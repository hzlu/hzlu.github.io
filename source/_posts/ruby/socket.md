---
title: Socket
category: ruby
---

网络功能是通过套接字实现的，它是一种 `IO` 对象

### TCPSocket

客户端使用 `TCPSocket` 类，而服务器可以使用 `TCPServer` 类，
所有的套接字类都在标准库中，因此要使用它们，首先要 `require 'socket'`。

获得 `TCPSocket` 实例，通过 `TCPSocket.new` 或 `TCPSocket.open`：

- 第一个参数是主机名
- 第二个参数是端口号，端口号可以使用 `Fixnum` 或 `String` 对象表示

下面代码演示了一个客户端，连接指定主机和端口，并从连接的套接字中读取所有可获得的数据，然后退出：

```ruby
require 'socket'
host, port = ARGV

s = TCPSocket.open(host, port)
while line = s.gets
  puts line.chop
end

s.close
```

可以带一个代码块，可把打开的套接字传给代码块，在代码块返回时自动关闭套接字。

```ruby
require 'socket'
host, port = ARGV
TCPSocket.open(host, port) do |s|
  while line = s.gets
    puts line.chop
  end
end
```

这种客户端可以用于访问老式的时间服务，客户端无须发送请求，只是简单地连接服务器，服务器就会返回响应数据。

### TCPServer

本质上 `TCPServer` 对象是产生 `TCPSocket` 对象的工厂对象。用给定端口号调用 `TCPServer.open` 方法可以创建对象，在对象上调用 `accept` 方法，会等待客户端连接该指定端口，连接后，Server 返回一个 `TCPSocket` 对象，表示与所连客户端的连接。

```ruby
require 'socket'
server = TCPServer.open(2000)
loop do
  client = server.accept
  client.puts(Time.now.ctime)
  client.close
end
```

### UDPSocket

除了 `TCPSocket` 和 `TCPServer` 外，因特网还可以使用 `UDP` 的数据报来实现，使用 `UDPSocket` 类来实现。

发送当个数据包，省去建立永久开销。代码演示功能：客户端发送一个文本数据报给指定主机和端口，服务器接收文本，把它转换为大写，然后用另一个数据报返回。

#### 客户端代码

```ruby
require 'socket'
host, port, request = ARGV
ds = UDPSocket.new #创建数据报socket
ds.connect(host, port)  #连接主机端口
ds.send(request, 0) #发送请求文本，send的第二个参数是必须的，指定发送的标志
response, address = ds.recvfrom(1024) #等待响应，限制接收数据大小为1KB
puts response #打印响应结果
```

#### 服务器代码

```ruby
require 'socket'
port = ARGV[0]
ds = UDPSocket.new  #使用与客户端同样的类，没有UDPServer，只有UDPSocket
ds.bind(nil, port)  #使用bind监听端口
loop do
  request, address = ds.recvfrom(1024) #调用次序相反，先接收，再send，
  response = request.upcase #recvfrom方法返回两个值，一个是接收数据，
  clientaddr = address[3]   #另一个是数组，包含数据来源的信息。
  clientname = address[2]
  clientport = address[1]
  ds.send(response, 0, clientaddr, clientport)
  puts "Connection from: #{clientname} #{clientaddr} #{clientport}"
end
```

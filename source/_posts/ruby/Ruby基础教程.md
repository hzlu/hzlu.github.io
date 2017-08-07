---
title: 《Ruby 基础教程》笔记
category: ruby
---

```
begin
	可能发生异常的处理
rescue => 引用异常对象的变量
	发生异常的处理
end
```

如果不指定变量名，会把异常对象赋值给变量 `$!`

最后发生异常的位置信息： `$@`

异常对象的方法：

* `class` 异常的种类
* `message` 异常信息
* `backtrace` 发生异常的位置信息 `$@` 与 `$!.backtrace` 是等价的。

rescue修饰符

`表达式1 rescue 表达式2`

如果表达式1发生异常，那么表达式2的值就会成为整体表达式的值。

rescue中如果不指定异常类，程序就会默认捕捉`StandardError`类及其子类的异常。

我们自己定义异常类时，一般会先定义继承`StandardError`类的新类：

MyError = Class.new(StandardError)

##块block

`block_given?`方法判断是否有块传递给方法。

对`yield`传递参数后，参数值就会作为**块变量**传递到块中。

在块中使用break，程序会马上返回到调用块的地方。

块变量只能在块内部使用，它不能覆盖外部的局部变量。我们可以定义块变量以外的块局部变量，在块变量后使用`;`语法。

```
ary.each do |x;y|
  ...
end
```

随机数的种子 `Random.new(1)`

如果不对迭代器方法指定块，就会返回 `Enumerator` 对象。

使用 `%w` 创建不包含空白的字符串数组

使用 `%i` 创建符号数组

建议使用`()` `<>` `||` 括起来

*a.at(n)

*a.slice(n)

*a.slice(n..m)

*a.slice(n, len)

a.values_at(n1, n2, ...) 通过多个索引创建数组

##把数组作为列

队列queue 与 栈stack 这两种相对的数据结构都有**追加元素**和**获取元素**的操作数据的方式。

队列queue是一种**按元素被追加时的顺序**来获取元素的数据结构。FIFO先进先出，也称为等待队列。

栈则是一种按照与**元素被追加时的顺序相反的顺序**来获取元素的数据结构。LIFO后进先出。

向数组追加元素的方法`unshift` `push`
向数组删除元素的方法`shift` `pop`
引用元素方法`first` `last`

使用push和shift实现队列

使用push和pop实现栈

注意`concat`与`+`方法的不同，`concat` 会改变原数组，`+` 返回新数组

###从数组中删除元素
a.compact

a.compact! 如果a数组没有nil元素可以删除，就会返回nil

a.delete(x)

a.delete_at(n)

a.delete_if{|item|...}

a.reject{|item|...}

a.reject!{|item|...}

a.slice

a.uniq

a.uniq!
###替换数组元素

a.collect{|item|...}

a.map{|item|...}

a.fill(value,begin,len)把数组的指定元素替换成value

a.fill(value,n..m)

a.flatten平坦化数组，展开嵌套数组

a.reverse

a.sort

a.sort_by{|i|...}

```
	if x >= 1 && x <= 10 then
	end
```
&& || !
优先级略低的逻辑运算符 and or not

unless就是if not

**使用 do~end 时，可以省略把参数列表括起来的（） 。使用{~} 时，只有在没有参数的时候才可以省略（），有一个以上的参数时就不能省略。**

调用类方法时，可以使用 `::` 代替 `.`。在Ruby语法中，两者代表的意思是一样的。

默认参数要放在参数列表右侧，只省略左侧的参数或中间的某个参数是不行的。

如果yield部分有参数，程序就会将其作为块变量传到块里。

类表示对象的种类。Ruby中的对象都一定属于某个类。

相同类的对象所使用的方法也相同。类决定了对象的行为。

对象的 `.classs` 和 `.instance_of?` 方法。根据继承关系反向追查对象是否属于某个类时，可以使用`.is_a？` 方法。

对象的存取器。`attr_reader` `attr_writer` `attr_accessor` 。只要指定实例变量的符号，Ruby就会自动帮我们定义相应的存取器。

```
	attr_reader :name
	attr_writer :name
	attr_accessor :name
```

self这个特殊的变量，引用了方法的接收者，如果省略了接收者，会默认把self作为该方法的接收者。

已经被系统使用不能被我们自定义的变量名有：`nil/true/false/__FILE__/__LINE__/__ENCODING__`

```
class HelloWorld

	Version = 1.0

	attr_accessor :name

	def initialize(name = 'ruby')
		@name = name
	end

	def say_hello
		puts "hello world, said #{@name}"
	end

	def say_hi
		puts "hi world, said #{self.name}"
	end

	#使用self定义类方法
	def self.say_bye
		puts "class say_bye"
	end

	#使用def 类名.方法名 ~ end 的形式定义类方法
	def HelloWorld.say_byebye
		puts "class say_byebye"
	end

	#使用class << self ~ end这样的形式，在class的上下文中定义类方法
	class << self
		def hello(name)
			puts "#{name} said hello"
		end
	end
end
```

还有一种叫单例类定义，在 `class << 类名 ~ end` 这个特殊的类定义中，以定义实例方法的形式来定义类方法。

类对象调用 `instance_methods` 方法后，就会以符号的形式返回该类的实例方法数组

模块的作用之一，提供命名空间，namespace

如果希望把方法作为模块函数公开给外部使用，就需要用到modul_function方法，参数表示方法名的符号。

如果想知道类是否包含了某个模块，使用 `include?` 方法

取得继承关系的列表的方法 `ancestors`，直接返回父类 `superclass`

`Kernel`模块是Ruby内部的一个核心模块，Ruby程序运行时所需要的共通函数都封装在这个模块中，例如p方法，raise方法等都是由Kernel模块提供的模块函数。

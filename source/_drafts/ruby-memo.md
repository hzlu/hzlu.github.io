---
title: ruby-memo
categories:
tags:
---


## Singleton

`Singleton` 模块实现了单例模式

用法是直接在类中 `include`，则该类就实现了单例模式

```ruby
class Klass
  include Singleton
  # ...
end

a, b = Klass.instance, Klass.instance
a == b # => true

Klass.new # => NoMethodError
```

## ObjectSpace


```ruby
ObjectSpace.each_object(Klass) {}
```

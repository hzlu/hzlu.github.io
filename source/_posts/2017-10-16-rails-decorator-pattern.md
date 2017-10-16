---
title: rails-decorator-pattern
tags:
  - rails
  - 设计模式
  - ruby
categories:
  - rails
date: 2017-10-16 17:13:46
---



[原文链接](http://www.thegreatcodeadventure.com/rails-refactoring-part-iii-the-decorator-pattern/)

## 什么是装饰器模式

装饰器模式允许我们对某个实例对象添加一些行为，但又不影响类的其他实例对象。在把模型实例传给视图前，我们能“装饰”这个模型实例，或者我们可以在视图中装饰这个实例。

通过抽取复杂的呈现逻辑，可以简化视图层和模型层

## 装饰视图

- 定义文件夹 `app/decorators`
- 定义文件 `app/decorators/repository_decorator.rb`

```ruby
# app/decorators/repository_decorator.rb
class RepositoryDecorator < SimpleDelegator
  def display_name
    name.gsub('-', ' ').titleize
  end
end
```

`SimpleDelegator` 是 Ruby 原生类

### 装饰前的视图代码

```ruby
# app/views/repositories/_repo.html.erb
<%= @repository.name.gsub("-", " ").titleize %>

# app/views/issues/show.html.erb
<%= @issue.repository.name.gsub("-", " ").titleize %>
```

### 装饰后的视图代码

```ruby
# app/views/repositories/_repo.html.erb
<%= RepositoryDecorator.new(@repository).display_name %>

# app/views/issues/show.html.erb
<%= RepositoryDecorator.new(@issue.repository).display_name %>
```

## 装饰模型

假设有这么个模型

```ruby
# app/models/user.rb
def display_phone_number
  phone_number && !phone_number.empty? ?
    phone_number : "
     <i>add your phone number to receive text
     message updates</i>".html_safe
end
```

在视图中调用这个方法

```ruby
# app/views/users/show.html.erb
...
<p>phone number: <%= @user.display_phone_number %>
```

以上违反了单一职责原则，在模型中写入了表现层的逻辑

### 装饰器修改

- 创建文件 `app/decorators/user_decorator.rb`

```ruby
# app/decorators/user_decorator.rb
class UserDecorator < SimpleDelegator
  def display_phone_number
    phone_number && !phone_number.empty? ?
      phone_number : "
      add your phone number to receive text
       message updates".html_safe
  end
end
```

在视图中使用

```ruby
# app/views/users/show.html.erb
...
<p>phone number: <%= UserDecorator.new(@user).display_phone_number %>
```

以上


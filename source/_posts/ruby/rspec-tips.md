---
title: Rspec 测试技巧
category: ruby
---

### 使用共享用例

共享用例放在代码块`shared_examples`中

~~~ruby
shared_examples("public test case") do
  describe "xxx" do
    it "xxxx" do
      test_somethings
    end
  end
  describe "zzz" do
    it "zzz" do
      test_other
    end
  end
end
~~~

然后在需要用到这个共享用例的describe块或context块中引入`it_behaves_like`

~~~ruby
describe "foo" do
  it_behaves_like "public test case"
  ...
end
~~~

### 使用辅助宏
把多处用到的代码定义成辅助宏，宏存储在`spec/support`的各自模块中，需要在`spec/spec_helper.rb`中设置

~~~ruby
# 定义辅助宏spec/supprt/foo_bar.rb
module FooBar
  def baz(foo)
    do_something_with_foo
  end
end
~~~
~~~ruby
# 设置spec_helper.rb
RSpec.configure do |config|
  config.include FooBar
end
~~~
~~~ruby
# 在测试中使用辅助宏
it xxx do
  baz(foo)
end
~~~

### 自定义RSpec匹配器
匹配器保存在`spec/support/matchers`中，各匹配器保存在单独文件中，RSpec默认会自动加载这些匹配器，一般没必要自定义。

此外，还可以使用匹配器扩展库[shoulda](https://github.com/thoughtbot/shoulda)

### 使用let()
- 不用赋值给实例变量就可以缓存值
- 定义的变量是惰性计算，不调用就不会执行赋值
- 使用let!()就会在测试用例运行前强制赋值，但就违背了let()的初衷

~~~ruby
let(:contact) do
  create(:contact, firstname: "xxx")
end
# 用例中就可以直接使用contact
~~~

### 使用subject{} (别名it{})
用来声明测试的对象，后续的用例就无需明确指定，直接使用`should`即可进行测试。

~~~ruby
subject { build(:user, firstname: "foo", lastname: "bar") }
it "returns a full name" do
  should be_named "foo bar"
end
# 或精简成
it { shoulk be_named "foo bar" }
~~~

### 驭件mock和桩件stub
- mock用来替代真实对象的测试对象，测试替身，类似通过Factory Girl生成的对象，但不会改动数据库中的数据，速度快些。使用Factory Girl提供的`build_stubbed()`方法创建mock，一个完整的假冒对象。
- stub是对指定对象方法的重写，用来重写方法的默认功能，返回你预设的值供测试使用。不方便在测试中调用外部接口时可用stub来模拟。
  + `receive`中的参数，是需要重写输出的方法
  + `with`的参数则是传给重写的方法的参数

~~~ruby
allow(Contact).to receive(:order).with("lastname, firstname").and_return([contact])
~~~


### 给测试用例打上标签
~~~ruby
# 在测试文件中打上标签
feature "FOO", focus: true do
  xxx
end
# 设置
config.filter_run focus: true
~~~

[参考《Everyday Rails Testing with RSpec》](https://leanpub.com/everydayrailsrspec)


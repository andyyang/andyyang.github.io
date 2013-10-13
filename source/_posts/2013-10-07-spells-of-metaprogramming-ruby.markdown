---
layout: post
title: "Ruby元编程惯用法"
date: 2013-10-07 23:00
comments: true
categories: Ruby
---
以下Ruby元编程的惯用法引用自[《Metaprogramming Ruby》](http://pragprog.com/book/ppmetr/metaprogramming-ruby)，该书对Ruby元编程做了很好的总结，非常值得阅读。

## 基本用法

### Argument Array
将参数列表收缩为一个数组。
{% codeblock lang:ruby %}
def my_method(*args)
  args.map { |arg| arg.reverse }
end

my_method('xyz', 'abc', '123') # => ['zyx', 'cba', '321']
{% endcodeblock %}

### Mimic Method
将一个方法假装为语言结构成分使用。
{% codeblock lang:ruby %}
def BaseClass(name)
  name == "string" ? String : Object
end

class C < BaseClass "string"
  attr_accessor :an_attribute
end

obj = C.new
obj.an_attribute = 1 
{% endcodeblock %}

### Monkeypatch
改变一个已有类的特性。
{% codeblock lang:ruby %}
"xyz".reverse # => "zyx"

class String
  def reverse
    "override"
  end
end

"xyz".reverse # => "override"
{% endcodeblock %}

### Named Arguments
将方法参数收集到一个hash中，通过名称识别参数。
{% codeblock lang:ruby %}

def my_method(args) 
  args[:arg2]
end
end

my_method(arg1 : "A", arg2: "B", arg3 : "C") # => "B"
{% endcodeblock %}

### Namespace
在模块中定义常量以避免名称冲突。
{% codeblock lang:ruby %}
module MyNamespace
  class Array
    def to_s 
      "my class"
    end
  end
end

Array.new # => []
MyNamespace::Array.new # => "my class"
{% endcodeblock %}

### Nil Guard
通过or操作覆写nil引用。
{% codeblock lang:ruby %}
x = nil
y = x || "a value" # => "a value"
{% endcodeblock %}

### Code Processor
处理来自外部来源的Strings of Code。
{% codeblock lang:ruby %}
File.readlines("a_file_containing_lines_of_ruby.txt").each do |line| 
  puts "#{line.chomp} ==> #{eval(line)}"
end

# >> 1 + 1 ==> 2
{% endcodeblock %}

### Dynamic Dispatch
在运行时决定调用哪个方法。
{% codeblock lang:ruby %}
method_to_call = :reverse
obj = "xyz"

obj.send(method_to_call) # => "zyx"
{% endcodeblock %}

### Open Class
修改已存在的类
{% codeblock lang:ruby %}
class String
  def my_method
    'my method'
  end
end

"abc".my_method # => "my method"
{% endcodeblock %}

### Ghost Method
响应无相应方法的消息。
{% codeblock lang:ruby %}
class C 
  def method_missing(name, *args) 
    name.to_s.reverse
  end
end

obj = C.new 
obj.my_ghost_method # => "dohtem_tsohg_my"
{% endcodeblock %}

### Hook Method
覆写一个方法以拦截对象模型的事件。
{% codeblock lang:ruby %}
$INHERITORS = []
class C 
  def self.inherited(subclass) 
    $INHERITORS << subclass
  end
end

class D < C
end

class E < C
end

class F < E
end

$INHERITORS # => [D, E, F]
{% endcodeblock %}

### Flat Scope
通过closure在两个scope之间共享变量。
{% codeblock lang:ruby %}
class C 
  def an_attribute 
    @attr
  end
end

obj = C.new 
a_variable = 100

obj.instance_eval do
  @attr = a_variable
end

obj.an_attribute # => 100 
{% endcodeblock %}

### Kernel Method
在模块Kernel中定义一个方法以使该方法能在所有对象中都能使用。
{% codeblock lang:ruby %}
module Kernel 
  def a_method
    "a kernel method" 
  end
end

a_method # => "a kernel method"
{% endcodeblock %}

### Scope Gate
通过class、module或者def关键字隔离一个Scope。
{% codeblock lang:ruby %}
a = 1
defined? a # => "local-variable"

module MyModule
  b = 1
  defined? a # => nil
  defined? b # => "local-variable"
end

defined? a # => "local-variable"
defined? b # => nil
{% endcodeblock %}

### Self Yield
将self传入当前的block。
{% codeblock lang:ruby %}
class Person
  attr_accessor :name, :surname

  def initialize
    yield slef
  end
end

joe = Person.new do |p|
 p.name = 'Joe'
 p.surname = 'Smith'
end
{% endcodeblock %}

### Singleton Method
对单独的一个对象定义方法。
{% codeblock lang:ruby %}
obj = "xyz"

class << obj
  def my_singleton_method
    "a"
  end
end

obj.my_singleton_method # => "a"
{% endcodeblock %}

### String of Code
解析一个Ruby代码字符串。
{% codeblock lang:ruby %}
my_string_of_code = "1 + 2"
eval(my_string_of_code) # => 3
{% endcodeblock %}

### Symbol To Proc
将一个symbol转换为一个block，该block调用单独的一个方法。
{% codeblock lang:ruby %}
[1, 2, 3, 4].map(&:even?) # => [false, true, false, true]
{% endcodeblock %}

## 进阶用法

### Around Alias
为一个方法指定别名，并在重新定义该方法时调用之前被别名的方法。
{% codeblock lang:ruby %}
class String
  alias :old_reverse :reverse
  
  def reverse
    "t#{old_reverse}t"
  end
end

"123".reverse # => "t321t"
{% endcodeblock %}

### Blank Slate
将方法从对象中删除，使这些方法成为Ghost Methods。
{% codeblock lang:ruby %}
class C 
  def method_missing(name, *args) 
    "a Ghost Method"
  end

  instance_methods.each do |m|
    undef_method m unless m.to_s =~ /method_missing|respond_to?|^__/
  end
end

obj = C.new
obj.to_s # => "a Ghost Method"
{% endcodeblock %}

### Class Extension
通过类的eigenclass包含模块来定义类方法。
{% codeblock lang:ruby %}
class C; end

module M
  def my_method 
    "a class method"
  end
end

class << C
  include M
end

C.my_method # => "a class method"
{% endcodeblock %}

### Class Extension Mixin
通过Hook Method让模块扩展包含该模块的类。
{% codeblock lang:ruby %}
module M
  def self.included(base)
    base.extend(ClassMethods)
  end

  module ClassMethods
    def my_method 
      "a class method"
    end
  end
end

class C
  include M
end

C.my_method # => "a class method"
{% endcodeblock %}

### Class Instance Variable
将类层次的状态存储在Class对象的实例变量中。
{% codeblock lang:ruby %}
class C
  @my_class_instance_variable = "some value"

  def self.class_attribute
    @my_class_instance_variable
  end
end

C.class_attribute # => "some value"
{% endcodeblock %}

### Class Macro
在类定义中使用类方法。
{% codeblock lang:ruby %}
class C; end

class << C
  def my_macro(arg) 
    "my_macro(#{arg}) called"
  end
end

class C
  my_macro :x # => "my_macro(x) called"
end
{% endcodeblock %}

### Clean Room
将一个对象作为一个block计算的环境。
{% codeblock lang:ruby %}
class CleanRoom
  def a_useful_method(x) 
    x * 3
  end
end

CleanRoom.new.instance_eval { a_useful_method(3) } # => 9
{% endcodeblock %}

### Context Probe
在一个对象的上下文中执行一个block以获取信息。
{% codeblock lang:ruby %}
class C
  def initialize
    @x = "a private instance variable"
  end 
end

obj = C.new
obj.instance_eval { @x } # => "a private instance variable"
{% endcodeblock %}

### Deferred Evaluation
将一段代码及其上下文存储在一个proc或者lambda中以用于以后的计算。
{% codeblock lang:ruby %}
class C
  def store(&block)
    @my_code_capsule = block 
  end 

  def execute
    @my_code_capsule.call
  end
end

obj = C.new
obj.store { $X = 1 }
$X = 0

obj.execute
$X # => 1
{% endcodeblock %}

### Dynamic Method
确定如何在运行时定义一个方法。
{% codeblock lang:ruby %}
class C; end

C.class_eval do
  define_method :my_method do
    "a dynamic method"
  end
end

obj = C.new
obj.my_method # => "a dynamic method"
{% endcodeblock %}

### Dynamic Proxy
将不匹配的方法调用发送给另一个对象。
{% codeblock lang:ruby %}
class MyDynamicProxy
  def initialize(target)
    @target = target
  end

   def method_missing(name, *args, &block)
     "result: #{@target.send(name, *args, &block)}"
   end
end

obj = MyDynamicProxy.new("a string")
obj.reverse # => "result: gnirts a"
{% endcodeblock %}

### Lazy Instance Variable
直到第一次访问一个实例变量时才对该实例变量进行初始化。
{% codeblock lang:ruby %}
class C 
  def attribute
    @attribute = @attribute || "some value"
  end
end

obj = C.new
obj.attribute # => "some value"
{% endcodeblock %}

### Object Extension
将一个模块Mixin到一个对象的Eigenclass中以定义Singleton Method。
{% codeblock lang:ruby %}
obj = Object.new

module M
  def my_method
    'a singleton method'
  end
end

class << obj
  include M
end

obj.my_method # => "a singleton method"
{% endcodeblock %}

### Pattern Dispatch
根据方法的名称选择调用哪个方法。
{% codeblock lang:ruby %}
$x = 0

class C
  def my_first_method
    $x += 1
  end

  def my_second_method
    $x += 2
  end
end

obj = C.new
obj.methods.each do |m|
  obj.send(m) if m.to_s =~ /^my_/
end

$x # => 3
{% endcodeblock %}

### Sandbox
在一个安全环境中执行非可信代码。
{% codeblock lang:ruby %}
def sandbox(&code)
  proc {
    $SAFE = 2
    yield
  }.call
end

begin
  sandbox { File.delete 'a_file' }
rescue Exception => ex
  ex
end
{% endcodeblock %}

### Shared Scope
在同一个Flat Scope的多个上下文中共享变量。其他方法见不到这些变量。
{% codeblock lang:ruby %}
lambda {
  shared = 10

  self.class.class_eval do
    define_method :counter do
      shared
    end

    define_method :down do
      shared -= 1
    end
  end
}.call

counter # => 10
3.times { down }
counter # => 7
{% endcodeblock %}


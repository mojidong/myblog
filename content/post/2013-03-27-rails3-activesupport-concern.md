---
title: "Rails3:ActiveSupport::Concern详解"
date: 2013-03-27
description: ""
categories: [ "rails" ]
tags: ["rails","ruby"]
aliases: [/rails/2013/03/27/rails3-activesupport-concern/]
---

模块是方法和常量的集合，模块和类一样，其中的可以包括两种方法：实例方法(Instance Method)、模块方法(Module Method)。

当一个类`include`(Mixin)一个模块时，模块中的实例方法会成为该类的实例方法，`expand`(Mixin)时,模块中的实例方法会成为该类的类方法。两种情况下模块方法都会被忽略。

同时实现类方法和实例方法的混入看起来是鱼和熊掌的问题，要放弃吗？显然是不可能的，我们都是贪婪的。

一个解决办法

```ruby
module ClassMethods
end

module InstanceMethods
end

class Foo
    extend ClassMethods
    include InstanceMethods
end
```
看起来可以正常工作，但是不那么优雅，粒度太大。

聪明的程序员总是能想到解决办法

```ruby
module Mod

    def self.included(base)
        base.extend(ClassMothods)
    end

    module ClassMethods
        # 类方法定义
    end

    #实例方法定义
end

class M
    include Mod
end
```

嗯，看起来不错，比第一种方法优雅多了。但是用着用着问题就来

```ruby
module Foo
  def self.included(base)
    base.extend ClassMethods
  end

  module ClassMethods
    def say
      puts 'say'
    end
  end
end

module Bar
  include Foo

  def self.included(base)
      base.say
  end
end

class Host
  include Bar
end
```
上面代码会抛出`NoMethodError: undefined method 'say' for Host:Class`异常

原因在于`include Bar`的时候`Foo`中`included(base)`被执行,此时`base`是`Bar`

一个解决办法

```ruby
module Foo
  def self.included(base)
    base.extend ClassMethods
  end

  module ClassMethods
    def say
      puts 'say'
    end
  end
end

module Bar
  def self.included(base)
      base.say
  end
end

class Host
  include Foo
  include Bar
end
```

问题虽然解决的，但是引入了一个新的问题，所有使用`Bar`的地方都要知道它依赖`Foo`,需要分出额外的精力来维护这种依赖关系，我们应该把他们的依赖关系隐藏起来。

看来我们还是太挑剔

### ActiveSupport::Concern

本文重点来了

为了解决这种依赖关系，rails中增加了`ActiveSupport::Concern`([源码](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/concern.rb))这个工具。

```ruby
module ActiveSupport
  # A typical module looks like this:
  #
  #   module M
  #     def self.included(base)
  #       base.extend ClassMethods
  #       base.class_eval do
  #         scope :disabled, -> { where(disabled: true) }
  #       end
  #     end
  #
  #     module ClassMethods
  #       ...
  #     end
  #   end
  #
  # By using <tt>ActiveSupport::Concern</tt> the above module could instead be
  # written as:
  #
  #   require 'active_support/concern'
  #
  #   module M
  #     extend ActiveSupport::Concern
  #
  #     included do
  #       scope :disabled, -> { where(disabled: true) }
  #     end
  #
  #     module ClassMethods
  #       ...
  #     end
  #   end
  #
  # Moreover, it gracefully handles module dependencies. Given a +Foo+ module
  # and a +Bar+ module which depends on the former, we would typically write the
  # following:
  #
  #   module Foo
  #     def self.included(base)
  #       base.class_eval do
  #         def self.method_injected_by_foo
  #           ...
  #         end
  #       end
  #     end
  #   end
  #
  #   module Bar
  #     def self.included(base)
  #       base.method_injected_by_foo
  #     end
  #   end
  #
  #   class Host
  #     include Foo # We need to include this dependency for Bar
  #     include Bar # Bar is the module that Host really needs
  #   end
  #
  # But why should +Host+ care about +Bar+'s dependencies, namely +Foo+? We
  # could try to hide these from +Host+ directly including +Foo+ in +Bar+:
  #
  #   module Bar
  #     include Foo
  #     def self.included(base)
  #       base.method_injected_by_foo
  #     end
  #   end
  #
  #   class Host
  #     include Bar
  #   end
  #
  # Unfortunately this won't work, since when +Foo+ is included, its <tt>base</tt>
  # is the +Bar+ module, not the +Host+ class. With <tt>ActiveSupport::Concern</tt>,
  # module dependencies are properly resolved:
  #
  #   require 'active_support/concern'
  #
  #   module Foo
  #     extend ActiveSupport::Concern
  #     included do
  #       def self.method_injected_by_foo
  #         ...
  #       end
  #     end
  #   end
  #
  #   module Bar
  #     extend ActiveSupport::Concern
  #     include Foo
  #
  #     included do
  #       self.method_injected_by_foo
  #     end
  #   end
  #
  #   class Host
  #     include Bar # works, Bar takes care now of its dependencies
  #   end
  module Concern
    def self.extended(base) #:nodoc:
      base.instance_variable_set("@_dependencies", [])
    end

    def append_features(base)
      if base.instance_variable_defined?("@_dependencies")
        base.instance_variable_get("@_dependencies") << self
        return false
      else
        return false if base < self
        @_dependencies.each { |dep| base.send(:include, dep) }
        super
        base.extend const_get("ClassMethods") if const_defined?("ClassMethods")
        base.class_eval(&@_included_block) if instance_variable_defined?("@_included_block")
      end
    end

    def included(base = nil, &block)
      if base.nil?
        @_included_block = block
      else
        super
      end
    end
  end
end
```
实现代码相当简短，使用也非常简单

```ruby
module M
    extend ActiveSupport::Concern
    included do
        self.send(:do_host_something)
    end

   module ClassMethods
      def wo
        # do something
      end
   end

   module InstanceMethods
      def ni
         # do something
      end
   end
end
```

终极版

```ruby
require 'active_support/concern'
module Foo
  extend ActiveSupport::Concern
  module ClassMethods
    def say
      puts 'say'
    end
  end
end

module Bar
  extend ActiveSupport::Concern
  include Foo
  included do
      self.say
  end
end

class Host
  include Bar
end
```
洗洗睡觉

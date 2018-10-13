---
title: "rails3中autoload_paths加载详解"
date: 2013-03-16
description: ""
categories: [ "rails" ]
tags: ["rails", "ruby"]
aliases: [/rails/2013/03/16/rails3-autoload_paths-principle/]
---

在rails3中默认不在加载lib目录下的rb文件。

config/application.rb文件中

```ruby
#Custom directories with classes and modules you want to be autoloadable.
#config.autoload_paths += %W(#{config.root}/exters)
```

显然被注释掉了。

要开启则取消注释，修改为自己想加载的路径。

```ruby
config.autoload_paths += %W(#{config.root}/lib)
```

这样rails会自动加载lib下的rb文件。

---

* 没有子目录

	在lib下有go.rb
	
	```ruby
	module Go
		def say
			puts 'hello'
		end
	end
	```
	
	在其他地方使用的时候include进来，例如
	
	```ruby
	class ProductsController < ApplicationController
		include Go
		def index
			say()
		end
	end
	```

* 含有子目录

	在lib下有tools目录，下面有go.rb
	
	```ruby
	module Go
		def say
			puts 'hello'
		end
	end
	```
	
	controller中
	
	```ruby
	class ProductsController < ApplicationController
		include Go
		def index
			say()
		end
	end
	```
	
	很遗憾这这样会抛出异常`uninitialized constant` 。
	
	原因rails加载lib使用的`const_missing`机制，在找不到常量时，会根据这个常量的名字，加载文件。
	>如果类名（常量名、模块名都是常量）是 A::B::C，那么就把 :: 替换为 '/'，单词变成小写，然后尝试在 ActiveSupport::Dependencies.autoload_paths 里的路径中，去查找是否有 a/b/c.rb。
	
	显然lib下没有任何rb可加载,`inculde Go`肯定是找不到的，此时触发`const_missing`还是找不到。

	应该将go.rb里modules的名字改为`Tools::Go`
	
	```ruby
	module Tools::Go
		def say
			puts 'hello'
		end
	end
	```
	
	controller中`inlude`修改为`include Tools::Go`
	
	```ruby
	class ProductsController < ApplicationController
		include Tools::Go
		def index
			say()
		end
	end
	```
	
	同样`inculde Tools::Go`肯定是找不到的，此时触发`const_missing`，根据规则可知此时会去加载`tools/go.rb`，显然加载成功。

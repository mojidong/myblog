---
title: "kaminari设置最大页数"
date: 2013-04-03
description: ""
categories: [ "rails" ]
tags: ["rails", "ruby"]
aliases: [/rails/2013/04/03/kaminari-max-page/]
---

[kaminari](https://github.com/amatsuda/kaminari)是一款非常优秀的分页插件，它支持 **Rails/Sinatra/Padrino**。

它基本上解决我所有的分页需求，不过最近发现一个不足，好像没有设置最大页数的功能。

例如我的数据有1000页，但是我只想显示100页，这个时候就搞不定了。

初步来看有两个办法来解决问题

1.修改源代码

2.修改template

个人倾向第二种，侵入性小，方案确定那么就开始动手

---

走起

```erb
<%= paginate @blogs %>
```

该方法定义在
`~/.rvm/gems/ruby-1.9.3-p392/gems/kaminari-0.14.1/lib/kaminari/helpers/action_view_extension.rb` ([源码](https://github.com/amatsuda/kaminari/blob/master/lib/kaminari/helpers/action_view_extension.rb))

```ruby
 	# A helper that renders the pagination links.
    #
    #   <%= paginate @articles %>
    #
    # ==== Options
    # * <tt>:window</tt> - The "inner window" size (4 by default).
    # * <tt>:outer_window</tt> - The "outer window" size (0 by default).
    # * <tt>:left</tt> - The "left outer window" size (0 by default).
    # * <tt>:right</tt> - The "right outer window" size (0 by default).
    # * <tt>:params</tt> - url_for parameters for the links (:controller, :action, etc.)
    # * <tt>:param_name</tt> - parameter name for page number in the links (:page by default)
    # * <tt>:remote</tt> - Ajax? (false by default)
    # * <tt>:ANY_OTHER_VALUES</tt> - Any other hash key & values would be directly passed into each tag as :locals value.
    def paginate(scope, options = {}, &block)
      paginator = Kaminari::Helpers::Paginator.new self, options.reverse_merge(:current_page => scope.current_page, :total_pages => scope.total_pages, :per_page => scope.limit_value, :param_name => Kaminari.config.param_name, :remote => false)
      paginator.to_s
    end

```

看看`Kaminari::Helpers::Paginator#initialize` ([源码](https://github.com/amatsuda/kaminari/blob/master/lib/kaminari/helpers/paginator.rb))

```ruby
 def initialize(template, options) #:nodoc:
        @window_options = {}.tap do |h|
          h[:window] = options.delete(:window) || options.delete(:inner_window) || Kaminari.config.window
          outer_window = options.delete(:outer_window) || Kaminari.config.outer_window
          h[:left] = options.delete(:left) || Kaminari.config.left
          h[:left] = outer_window if h[:left] == 0
          h[:right] = options.delete(:right) || Kaminari.config.right
          h[:right] = outer_window if h[:right] == 0
        end
        @template, @options = template, options
        @theme = @options[:theme] ? "#{@options[:theme]}/" : ''
        @options[:current_page] = PageProxy.new @window_options.merge(@options), @options[:current_page], nil
        #FIXME for compatibility. remove num_pages at some time in the future
        @options[:total_pages] ||= @options[:num_pages]
        @last = nil
        # initialize the output_buffer for Context
        @output_buffer = ActionView::OutputBuffer.new
      end

```

显然`@window_options`和`@options`是重点，它里面包含了我们需要的数据。

继续

```erb
<%= paginator.render do -%>
	# ...
<% end -%>
```

该方法定义在`~/.rvm/gems/ruby-1.9.3-p392/gems/kaminari-0.14.1/lib/kaminari/helpers/paginator.rb` ([源码](https://github.com/amatsuda/kaminari/blob/master/lib/kaminari/helpers/paginator.rb))

```ruby
# render given block as a view template
def render(&block)
  instance_eval(&block) if @options[:total_pages] > 1
  @output_buffer
end
```

显然这个block可以访问`@window_options`和`@options`,那么问题就解决了，于是修改如下

```erb
<%= paginator.render do -%>
	# 默认max_page等于total_pages
	<% @options[:max_page] ||= total_pages %>
	<div class="page-bar">
		<%#= first_page_tag unless current_page.first? -%>
		<%= prev_page_tag unless current_page.first? %>
		<% each_page do |page| -%>
        	# 达到max_pages则不在渲染按钮
			<% break if page > @options[:max_page] %>
			<% if page.left_outer? || page.right_outer? || page.inside_window? -%>
				<%= page_tag page %>
			<% elsif !page.was_truncated? -%>
				<%= gap_tag %>
			<% end -%>
		<% end -%>
        # 隐藏下一页按钮
		<% if current_page < @options[:max_page] %>
			<%= next_page_tag unless current_page.last? %>
		<% end %>
		<%#= last_page_tag unless current_page.last? -%>
	</div>
<% end -%>
```

如果只要显示100页则

```erb
<%= paginate @blogs, :max_page => 100 %>
```

嗯，有些小瑕疵，不过这显然难不倒我们。

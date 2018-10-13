---
title: "rails部署nginx+unicorn详解"
date: 2013-04-20
description: "rails部署nginx+unicorn详解"
categories: [ "rails" ]
tags: ["rails","ruby","nginx"]
aliases: [/rails/2013/04/20/rails-nginx-unicorn/]
---

rails部署方案众多，今天介绍下nginx+unicorn.

### 简介

[unicorn](http://unicorn.bogomips.org/)是一个高性能的Rack HTTP server,它可以使用unix domain socket通信，获得更好的性能。

**官方列举了诸多特性**

> 更好的进程管理。  
> 由内核调度的负载均衡。  
> 类似nginx的无缝升级。  
> 更好的内存管理。  
> Ruby DSL 的简单配置。  
> 等等特性

### 安装

```sh
gem install unicorn
```

或者在Gemfile加入

```ruby
gem 'unicorn'
```

### 使用

1. 非rails的rack应用

  在应用跟目录运行
  
  ```sh
  unicorn #默认会监听8080端口
  ```

2. Rails应用

  在rails跟目录运行
  
  ```sh
  unicorn_rails #默认会监听8080端口
  ```

> 我们可以使用`-l`选项改变默认监听的地址和端口。  
> 我们可以使用 `unicorn -h` 或 `unicorn_rails -h` 查看更多选项说明

### 配置

**unicorn**提供配置文件,让我们进行详细的设置。详细的说明请看[这里](http://unicorn.bogomips.org/Unicorn/Configurator.html)。

* **after_fork(*args, &block)**

	worker fork 之后被调用。
    
* **before_exec(*args, &block)**

	在启动unicorn master之前被调用。
    
* **before_fork(*args, &block)**

	在master fork work之前被调用。
    
* **check_client_connection(bool)**
	
    检查客户端链接是否断开，防止断开的请求调用。` :tcp_nopush`选项下无效。
    
* **listen(address, options = {})**

	监听地址，可以使是`tcp`地址，也可以是`UNIX domain sockets`
    
    ```ruby
    listen 3000 # listen to port 3000 on all TCP interfaces
    listen "127.0.0.1:3000"  # listen to port 3000 on the loopback interface
    listen "/tmp/.unicorn.sock" # listen on the given Unix domain socket
    listen "[::1]:3000" # listen to port 3000 on the IPv6 loopback interface
    ```

* **logger(obj)**
	
    设置logger级别
    * debug
	* info
    * warn
    * error
    * fatal
    默认情况下log将输出到`stderr_path`,如果`unicorn`运行在`daemonized`模式下，
    则必须设置详细的`stderr_path`避免错误信息进入`/dev/null`
    
* **pid(path)**

	设置pid文件的位置
    
* **preload_app(bool)**
	
    在forking worker processes之前预加载程序
    
* **rewindable_input(bool)**

	禁止rewindability，可以提高上传的性能，降低io和内存使用
    
* **stderr_path(path)**

	重定向`stderr`到指定文件

* **stdout_path(path)**

	重定向`stdout`到指定文件
    
* **timeout(seconds)**

	设置worker processes的超时时间
    
* **user(user, group = nil)**

	设置运行worker processes的用户和组
    
* **worker_processes(nr)**

	设置worker_processes数量

* **working_directory(path)**

	设置unicorn工作目录，SIGUSR2将启动一个新的unicorn实例在这个目录。
    
---

官方给出的[最小化配置](http://unicorn.bogomips.org/examples/unicorn.conf.minimal.rb)

```ruby
# Minimal sample configuration file for Unicorn (not Rack) when used
# with daemonization (unicorn -D) started in your working directory.
#
# See http://unicorn.bogomips.org/Unicorn/Configurator.html for complete
# documentation.
# See also http://unicorn.bogomips.org/examples/unicorn.conf.rb for
# a more verbose configuration using more features.

listen 2007 # by default Unicorn listens on port 8080
worker_processes 2 # this should be >= nr_cpus
pid "/path/to/app/shared/pids/unicorn.pid"
stderr_path "/path/to/app/shared/log/unicorn.log"
stdout_path "/path/to/app/shared/log/unicorn.log"
```

官方给出的[完整配置](http://unicorn.bogomips.org/examples/unicorn.conf.rb)

```ruby
# Sample verbose configuration file for Unicorn (not Rack)
#
# This configuration file documents many features of Unicorn
# that may not be needed for some applications. See
# http://unicorn.bogomips.org/examples/unicorn.conf.minimal.rb
# for a much simpler configuration file.
#
# See http://unicorn.bogomips.org/Unicorn/Configurator.html for complete
# documentation.

# Use at least one worker per core if you're on a dedicated server,
# more will usually help for _short_ waits on databases/caches.
worker_processes 4

# Since Unicorn is never exposed to outside clients, it does not need to
# run on the standard HTTP port (80), there is no reason to start Unicorn
# as root unless it's from system init scripts.
# If running the master process as root and the workers as an unprivileged
# user, do this to switch euid/egid in the workers (also chowns logs):
# user "unprivileged_user", "unprivileged_group"

# Help ensure your application will always spawn in the symlinked
# "current" directory that Capistrano sets up.
working_directory "/path/to/app/current" # available in 0.94.0+

# listen on both a Unix domain socket and a TCP port,
# we use a shorter backlog for quicker failover when busy
listen "/tmp/.sock", :backlog => 64
listen 8080, :tcp_nopush => true

# nuke workers after 30 seconds instead of 60 seconds (the default)
timeout 30

# feel free to point this anywhere accessible on the filesystem
pid "/path/to/app/shared/pids/unicorn.pid"

# By default, the Unicorn logger will write to stderr.
# Additionally, ome applications/frameworks log to stderr or stdout,
# so prevent them from going to /dev/null when daemonized here:
stderr_path "/path/to/app/shared/log/unicorn.stderr.log"
stdout_path "/path/to/app/shared/log/unicorn.stdout.log"

# combine Ruby 2.0.0dev or REE with "preload_app true" for memory savings
# http://rubyenterpriseedition.com/faq.html#adapt_apps_for_cow
preload_app true
GC.respond_to?(:copy_on_write_friendly=) and
  GC.copy_on_write_friendly = true

# Enable this flag to have unicorn test client connections by writing the
# beginning of the HTTP headers before calling the application.  This
# prevents calling the application for connections that have disconnected
# while queued.  This is only guaranteed to detect clients on the same
# host unicorn runs on, and unlikely to detect disconnects even on a
# fast LAN.
check_client_connection false

before_fork do |server, worker|
  # the following is highly recomended for Rails + "preload_app true"
  # as there's no need for the master process to hold a connection
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!

  # The following is only recommended for memory/DB-constrained
  # installations.  It is not needed if your system can house
  # twice as many worker_processes as you have configured.
  #
  # # This allows a new master process to incrementally
  # # phase out the old master process with SIGTTOU to avoid a
  # # thundering herd (especially in the "preload_app false" case)
  # # when doing a transparent upgrade.  The last worker spawned
  # # will then kill off the old master process with a SIGQUIT.
  # old_pid = "#{server.config[:pid]}.oldbin"
  # if old_pid != server.pid
  #   begin
  #     sig = (worker.nr + 1) >= server.worker_processes ? :QUIT : :TTOU
  #     Process.kill(sig, File.read(old_pid).to_i)
  #   rescue Errno::ENOENT, Errno::ESRCH
  #   end
  # end
  #
  # Throttle the master from forking too quickly by sleeping.  Due
  # to the implementation of standard Unix signal handlers, this
  # helps (but does not completely) prevent identical, repeated signals
  # from being lost when the receiving process is busy.
  # sleep 1
end

after_fork do |server, worker|
  # per-process listener ports for debugging/admin/migrations
  # addr = "127.0.0.1:#{9293 + worker.nr}"
  # server.listen(addr, :tries => -1, :delay => 5, :tcp_nopush => true)

  # the following is *required* for Rails + "preload_app true",
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection

  # if preload_app is true, then you may also want to check and
  # restart any other shared sockets/descriptors such as Memcached,
  # and Redis.  TokyoCabinet file handles are safe to reuse
  # between any number of forked children (assuming your kernel
  # correctly implements pread()/pwrite() system calls)
end
```

github的[配置](https://github.com/blog/517-unicorn)

```ruby
# unicorn_rails -c /data/github/current/config/unicorn.rb -E production -D
 
rails_env = ENV['RAILS_ENV'] || 'production'
 
# 16 workers and 1 master
worker_processes (rails_env == 'production' ? 16 : 4)
 
# Load rails+github.git into the master before forking workers
# for super-fast worker spawn times
preload_app true
 
# Restart any workers that haven't responded in 30 seconds
timeout 30
 
# Listen on a Unix data socket
listen '/data/github/current/tmp/sockets/unicorn.sock', :backlog => 2048
 
 
##
# REE
 
# http://www.rubyenterpriseedition.com/faq.html#adapt_apps_for_cow
if GC.respond_to?(:copy_on_write_friendly=)
  GC.copy_on_write_friendly = true
end
 
 
before_fork do |server, worker|
  ##
  # When sent a USR2, Unicorn will suffix its pidfile with .oldbin and
  # immediately start loading up a new version of itself (loaded with a new
  # version of our app). When this new Unicorn is completely loaded
  # it will begin spawning workers. The first worker spawned will check to
  # see if an .oldbin pidfile exists. If so, this means we've just booted up
  # a new Unicorn and need to tell the old one that it can now die. To do so
  # we send it a QUIT.
  #
  # Using this method we get 0 downtime deploys.
 
  old_pid = RAILS_ROOT + '/tmp/pids/unicorn.pid.oldbin'
  if File.exists?(old_pid) && server.pid != old_pid
    begin
      Process.kill("QUIT", File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
      # someone else did our job for us
    end
  end
end
 
 
after_fork do |server, worker|
  ##
  # Unicorn master loads the app then forks off workers - because of the way
  # Unix forking works, we need to make sure we aren't using any of the parent's
  # sockets, e.g. db connection
 
  ActiveRecord::Base.establish_connection
  CHIMNEY.client.connect_to_server
  # Redis and Memcached would go here but their connections are established
  # on demand, so the master never opens a socket
 
 
  ##
  # Unicorn master is started as root, which is fine, but let's
  # drop the workers to git:git
 
  begin
    uid, gid = Process.euid, Process.egid
    user, group = 'git', 'git'
    target_uid = Etc.getpwnam(user).uid
    target_gid = Etc.getgrnam(group).gid
    worker.tmp.chown(target_uid, target_gid)
    if uid != target_uid || gid != target_gid
      Process.initgroups(user, target_gid)
      Process::GID.change_privilege(target_gid)
      Process::UID.change_privilege(target_uid)
    end
  rescue => e
    if RAILS_ENV == 'development'
      STDERR.puts "couldn't change user, oh well"
    else
      raise e
    end
  end
end
```

### 我的配置

unicorn.rb(此配置文件在rails项目config目录下)

```ruby
module Rails
  class <<self
    def root
      File.expand_path(__FILE__).split('/')[0..-3].join('/')
    end
  end
end

worker_processes 4

working_directory Rails.root

listen "#{Rails.root}/tmp/sockets/socket", :backlog => 64

timeout 30

pid "#{Rails.root}/tmp/pids/unicorn.pid"

stderr_path "#{Rails.root}/log/unicorn.log"
stdout_path "#{Rails.root}/log/unicorn.log"

# combine Ruby 2.0.0dev or REE with "preload_app true" for memory savings
# http://rubyenterpriseedition.com/faq.html#adapt_apps_for_cow
preload_app true
GC.respond_to?(:copy_on_write_friendly=) and
  GC.copy_on_write_friendly = true

check_client_connection false

before_fork do |server, worker|
  # the following is highly recomended for Rails + "preload_app true"
  # as there's no need for the master process to hold a connection
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!

  old_pid = "#{server.config[:pid]}.oldbin"
  if File.exists?(old_pid) && old_pid != server.pid
    begin
      sig = (worker.nr + 1) >= server.worker_processes ? :QUIT : :TTOU
      Process.kill(sig, File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
    end
  end
  
  # Throttle the master from forking too quickly by sleeping.  Due
  # to the implementation of standard Unix signal handlers, this
  # helps (but does not completely) prevent identical, repeated signals
  # from being lost when the receiving process is busy.
  sleep 1
end

after_fork do |server, worker|

  # the following is *required* for Rails + "preload_app true",
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection

  # if preload_app is true, then you may also want to check and
  # restart any other shared sockets/descriptors such as Memcached,
  # and Redis.  TokyoCabinet file handles are safe to reuse
  # between any number of forked children (assuming your kernel
  # correctly implements pread()/pwrite() system calls)
end
```

nginx.conf

```nginx
worker_processes 1;

events {
  worker_connections 1024;
}

http {
  
  include mime.types;

  default_type application/octet-stream;

  sendfile on;
  
  keepalive_timeout 0;

  upstream app_server {
    # fail_timeout=0 means we always retry an upstream even if it failed
    # to return a good HTTP response (in case the Unicorn master nukes a
    # single worker for timing out).

    # for UNIX domain socket setups:
    server unix: /path/to/app/tmp/sockets/socket fail_timeout=0;
  }
  
  server {
        listen 8080 default;
        return 403; 
  }

  server {
    listen 8080;

    server_name www.explame.com;

    location / {
      proxy_pass http://app_server;
    }

    # Rails error pages
    error_page 500 502 503 504 /500.html;
    location = /500.html {
      root /path/to/app/current/public;
    }
  }
}
```

启动unicorn

```sh
unicorn_rails -c /path/to/app/config/unicorn.rb -D -E production
```


> 无缝重启服务使用``` kill -USR2  `cat path/to/app/tmp/pids/unicorn.pid` ```
> 这样会启动一个新的`unicorn master`然后关闭旧的`unicorn master`

> 有可能你会发现你执行了上述命令但是没起作用，这是因为你可能没有在你的`Gemfile`中加入`gem 'unicorn'`



---
layout: post
title: "Gitlab Shell如何工作"
date: 2015-07-19 11:55
comments: true
categories: 
---

这篇博客的目的是研究一下从`git push`到commit出现在gitlab页面上中间发生到什么

开始之前补充一点背景知识，[ssh可以执行命令](http://cybermashup.com/2013/05/14/restrict-ssh-logins-to-a-single-command/)， Gitlab以及Gitolite这些操作远程仓库应该都有用这个技术

比如我们可以在`~/.ssh/authorized_keys`中加上自己当command

    command="./cmd" ssh-rsa <my-rsa-key>

这样就没法用ssh登陆了，每次登陆只会执行cmd这个程序

就像上面文章里写的，我们还可以用`$SSH_ORIGINAL_COMMAND`变量取到客户端发来当命令

```
#!/bin/sh
echo $SSH_ORIGINAL_COMMAND
```

    chmod +x ~/cmd

现在如果在登陆的时候后面参数，就可以回显里

    $ ssh git@gitlab.williamherry.com a b c
    # => a b c

Gitlab-shell就用的这个技术，`~/.ssh/authorized_keys`加一条类似这样当纪录

    command="/home/git/gitlab-development-kit/gitlab-shell/bin/gitlab-shell key-15",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa <my-rsa-key>

看一下gitlab-shell的代码

```
#!/usr/bin/env ruby

unless ENV['SSH_CONNECTION']
  puts "Only ssh allowed"
  exit
end

key_id = /key-[0-9]+/.match(ARGV.join).to_s
original_cmd = ENV['SSH_ORIGINAL_COMMAND']

require_relative '../lib/gitlab_init'

#
#
# GitLab shell, invoked from ~/.ssh/authorized_keys
#
#
require File.join(ROOT_PATH, 'lib', 'gitlab_shell')

if GitlabShell.new(key_id, original_cmd).exec
  exit 0
else
  exit 1
end
```

它从ARGV读到key_id，从环境变量`SSH_ORIGINAL_COMMAND`读到命令, 我们把它写到文件看看git push对应的命令是什么，在文件中加入下面一行代码

    File.write('/tmp/sss', ENV['SSH_ORIGINAL_COMMAND'])

通过测试发现有下面的对应关系

    git push  => git-receive-pack <repo>
    git pull  => git-upload-pack <repo>
    git fetch => git-upload-pack <repo>

现在key的id有了，命令有了，操作的repo也有了，我们看一下gitlab-shell怎么处理

    GitlabShell.new(key_id, original_cmd).exec

他用参数创建了一个GitlabShell实例之后执行exec方法

``` ruby
def initialize(key_id, origin_cmd)
  @key_id = key_id
  @origin_cmd = origin_cmd
  @config = GitlabConfig.new
  @repos_path = @config.repos_path
end
```

初始化只是把一些变量保存到了实例变量中，我们重点追踪cmd，记住这里到origin_cmd是'git-receive-pack <repo>'（我们只追踪git push）

然后就是执行exec方法，该方法主要执行三个步骤

```
def exec
  parse_cmd
  verify_access
  process_cmd
end
```

parse_cmd解析了一个origin_cmd，把命令（git-receive-pack）赋给了实例变量@git_cmd和@git_access，把repo赋给了实例变量@repo_name

verify_access拿前面得到的数据调用gitlab的api去验证看合不合法

最后process_cmd调用exec_cmd然后再调用了这样一行代码

    Kernel::exec({ 'PATH' => ENV['PATH'], 'LD_LIBRARY_PATH' => ENV['LD_LIBRARY_PATH'], 'GL_ID' => @key_id }, *args, unsetenv_others: true)

把变量都替换成真实的数据得到

    Kernel::exec({ 'path' => /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games, 'LD_LIBRARY_PATH' => , 'GL_ID' => key-15}, ["git-receive-pack", "/home/git/gitlab-development-kit/repositories/williamherry/test2.git"], unsetenv_others: true })

这里更深入的分析再以后再搞，简单说就是git push的时候gitlab-shell再服务端打开一个接受历程等待接收pack，客户端发送pack，好像从这里就断掉了，前面说的要在gitlab网站中现实的数据是在什么什么创建的呢？

你猜对了，他就是利用对git的[hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)，查看gitlab-shell目录下的hooks目录会发现有三个hooks， `post-receive  pre-receive  update`，push应该对应的是post-receive, 再里面添加打印信息

```
#!/usr/bin/env ruby

# This file was placed here by GitLab. It makes sure that your pushed commits
# will be processed properly.

refs = ARGF.read
key_id  = ENV['GL_ID']
repo_path = Dir.pwd

puts refs
puts key_id
puts repo_path

# reset GL_ID env since we already got its value
ENV['GL_ID'] = nil

require_relative '../lib/gitlab_custom_hook'
require_relative '../lib/gitlab_post_receive'

if GitlabPostReceive.new(repo_path, key_id, refs).exec &&
    GitlabCustomHook.new.post_receive(refs, repo_path)
  exit 0
else
  exit 1
end
```

push的输出中包含了相关的信息，这里的hooks是成功后才执行的，如果你没有新commit，push就不会有这些输出

```
remote: refs: 8939f77074dc85ac4529fcec1245f7c4563e17cd a6cd629e9deefb0ab9638217bdd82ae68ff17099 refs/heads/master
remote: key_id: key-15
remote: repo_path: /home/git/gitlab-development-kit/repositories/williamherry/test2.git
```

再这个hooks中它执行了GitlabPostReceive的exec方法和GitlabCustomHook的post_receive方法。而后面这个只是负责去执行custom_hooks下面的hooks，所以我们只看一看前面一个

```
class GitlabPostReceive
  def initialize(repo_path, actor, changes)
    @config = GitlabConfig.new
    @repo_path, @actor = repo_path.strip, actor
    @changes = changes
  end

  def exec
    result = update_redis
    broadcast_message = GitlabNet.new.broadcast_message
  end

  protected

  def update_redis
    system(*config.redis_command, 'rpush', queue, msg, err: '/dev/null', out: '/dev/null')
  end
end
```

我把代码做了简化，去掉了不重要的步骤，简单说这个hooks做了两件是：往redis里塞里一条数据，查了下广播（不知道是干什么的，不过好像不是很重要的样子），用puts发现它是往队列`resque:gitlab:queue:post_receive`中添加里这样一条信息

      {"class":"PostReceive","args":["/home/git/gitlab-development-kit/repositories/williamherry/test2.git","key-15","YTZjZDYyOWU5ZGVlZmIwYWI5NjM4MjE3YmRkODJhZTY4ZmYxNzA5OSA1MjJj\nZDhhN2MwMDEzYWU0MGU3ZTVhZDFiM2I5ZjlhNGE2OWYzNmIyIHJlZnMvaGVh\nZHMvbWFzdGVyCg==\n"]}

那一串奇怪的字符是Base64加密后的@changes，而@changes就是前面的refs即：

    8939f77074dc85ac4529fcec1245f7c4563e17cd a6cd629e9deefb0ab9638217bdd82ae68ff17099 refs/heads/master

好了，现在我们知道gitlab-shell和gitlag是通过redis共享数据的，接下来就是gitlab的`app/workers/post_receive.rb`中的代码做相应的处理了

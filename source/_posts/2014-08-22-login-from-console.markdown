---
layout: post
title: "Login From Console"
date: 2014-08-22 14:42
comments: true
categories:
---

最近有一个问题调试了好久,这个过程中学的了一些有意思的调试方法,在这里记录一下过程

问题是出现在staging环境上的,有一个form已提交就报错,只有那一条数据有这个错误,
但是单独肉眼看没发现有什么问题,查看日志只有很少的错误信息

```
ActiveRecord::RecordNotSaved (ActiveRecord::RecordNotSaved):
  app/forms/my_form.rb:51:in `submit'
  app/controllers/my_controller.rb:60:in `update'
```

`app/forms/my_form.rb`的51行是

```
my_object.save!
```

判断是没保存成功,由于不是在本地开发模式,不能用pry设置断点,只能打印出来了

```
begin
  my_object.save
rescue ActiveRecord::RecordNotSaved => e
  Rails.logger.info my_object.errors.full_messages
end
```

结果什么错误信息也没有,搜索了一下这个错误提示,有人说callback返回false会引起保存不成功并没有错误,例如有这样一个callback

```
def hide
  self.visible = false
end
```

但这个model并没有这样的Callback,把全部callback都删掉问题依旧,所以判断不是这个问题

同事支招把backtrace打出来会不会看出点眉目

```
begin
  my_object.save
rescue ActiveRecord::RecordNotSaved => e
  Rails.logger.info $!.inspect, $@
end
```

错误信息是多了,但任然看不出问题所在,由于经常用pry断点调试,现在已经没办法了,搜索怎么在非development环境用pry

这个过程中学会了怎么在console里发请求如登陆什么的

引用 http://stackoverflow.com/questions/151030/how-do-i-call-controller-view-methods-from-the-console-in-rails 的最后一个答案

```
# Start Rails console
rails console
# Disable forgery_protection
ApplicationController.allow_forgery_protection = false
# Get the login form
app.get '/community_members/sign_in'
# View the session
app.session.to_hash
# Copy the CSRF token "_csrf_token" and place it in the login request.
# Log in from the console to create a session
app.post '/community_members/login', {"authenticity_token"=>"gT7G17RNFaWUDLC6PJGapwHk/OEyYfI1V8yrlg0lHpM=",  "refinery_user[login]"=>'chloe', 'refinery_user[password]'=>'test'}
# View the session to verify CSRF token is the same
app.session.to_hash
# Copy the CSRF token "_csrf_token" and place it in the request. It's best to edit this in Notepad++
app.post '/refinery/blog/posts', {"authenticity_token"=>"gT7G17RNFaWUDLC6PJGapwHk/OEyYfI1V8yrlg0lHpM=", "switch_locale"=>"en", "post"=>{"title"=>"Test", "homepage"=>"0", "featured"=>"0", "magazine"=>"0", "refinery_category_ids"=>["1282"], "body"=>"Tests do a body good.", "custom_teaser"=>"", "draft"=>"0", "tag_list"=>"", "published_at(1i)"=>"2014", "published_at(2i)"=>"5", "published_at(3i)"=>"27", "published_at(4i)"=>"21", "published_at(5i)"=>"20", "custom_url"=>"", "source_url_title"=>"", "source_url"=>"", "user_id"=>"56", "browser_title"=>"", "meta_description"=>""}, "continue_editing"=>"false", "locale"=>:en}
```

登陆可以了但断点还是不行, 不过http://stackoverflow.com/questions/7744848/is-there-a-way-to-identify-why-the-database-rollsback-in-a-rails-application 的回答给了我灵感,保存不成功日志里会有rollback, 所以我修改了日志配置,果然发现了原因,原来是关联的model缺字段保存没成功

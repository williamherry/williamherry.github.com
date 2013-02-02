---
layout: post
title: "Rails图片上传"
date: 2013-01-25 22:27
comments: true
categories: 
---

好久没有更新这个了,竟然连`rake new_post['title']`都忘记了还得查文档,看来以后还是得时不时的写点

这里简单记录一下Rails中如何处理图片上传,我们选择[CarrierWave](https://github.com/jnicklas/carrierwave)这个gem来处理上传的文件,由于重点是处理图片上传,所以我们会使用scaffolding生成一个article的CURD,把图片绑定给它(一个article有一个图片)

首先我们创建一个Rails项目

```
rails new image-upload-demo
```

然后用脚手架生成article

```
cd image-upload-demo
rails g scaffold article title:string content:text
```

执行migration生成数据库表结构

```
rake db:migrate
```

`rails s`启动服务,访问`http://localhost:3000/articles`检查有没错误

如果没有问题,下一步我们就可以设定CarrierWave了,首先是安装,在Gemfile里添加一行

```
gem 'carrierwave'
```

然后执行`bundle install`进行安装,然后用下面的命令创建uploader

```
rails g uploader photo
```

这会创建`app/uploaders/photo_uploader.rb`这个文件,carrierwave的一些设定都在这个文件里面,我们目前先不做更改,只使用默认配置

接着我们给article表加一个字段来存储文件信息

```
rails g migration AddPhotoToArticles photo:string
```

别忘了执行`rake db:migrate`生成表结构

然后的article的model里加入调用carrierwave的代码

```
# app/models/article.rb
class Article < ActiveRecord::Base
  attr_accessible :content, :title, :photo

  mount_uploader :photo, PhotoUploader
end
```
别忘了把`photo`也加到`attr_accessible`里哦
这样Model和数据库就可以处理上传上来的文件了,接下来在view里加入上传图片的代码

```
# app/views/articles/_form.html.erb
<%= form_for(@article, :html => {:multipart => true}) do |f| %>
...
  <div class="field">
    <%= f.label :title %><br />
    <%= f.text_field :title %>
  </div>
  <div class="field">
    <%= f.label :content %><br />
    <%= f.text_area :content %>
  </div>
  <div class="field">
    <label>My Photo</label>
    <%= f.file_field :photo %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
...
```

关键在form_for中添加`:html => {:multipart => true}`,现在就可以使用上传的功能了,并且上传的图片可以简单的使用`<%= image_tag @article.photo %>`来调用,我们再在显示的view里加入显示图片的代码

```
# app/views/articles/show.html.erb
...

<p>
  <b>Photo:</b>
  <%= image_tag @article.photo %>
</p>
...
```

现在就可以测试了

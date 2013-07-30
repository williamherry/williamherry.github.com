---
layout: post
title: "omniauth-github使用示例"
date: 2013-07-29 11:57
comments: true
categories: 
---

这篇文档简单记录一下使用omniauth-github的过程

首先新建项目:

```
rails new omniauth-github-demo
```

为了让示例看起来不是很难看,我会把bootstrap也进来:

- 在Gemfile里加上`bootstrap-sass`然后运行bundle
- 在`app/assets/javascripts/application.js`加上`//= require bootstrap`
- 在`app/assets/stylesheets/application.css`加上` *= require bootstrap`

下一步把`omniauth-github`Gem加进来:
- 在Gemfile里加上`omniauth-github`并执行bundle
- 新建文件`config/initializers/omniauth.rb`包含以下内容

```
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET']
end
```

关于`ENV['GITHUB_KEY']`和`ENV['GITHUB_SECRET']`,需要在github上新建一个application,得到的`Client ID`和`Client Secret`就是对应的这两个东西,可以把他们写到`~/.bashrc`里面

```
export GITHUB_KEY='your client id'
export GITHUB_SECRET='your client secret'
```

还要注意你新建application的时候`The full URL to your application's homepage`和`Your application's callback URL`对应的端口需要和你的服务对应,比如我这里测试用rails开的3000端口,那我就可以填`http://localhost:3000/auth/github`和`http://localhost:3000/auth/github/callback`

下面我们需要在页面上加一个通过github登录的link,我直接写到layout里并加上一些div让bootstrap起作用

``` vim app/views/layouts/application.html.erb
<body>
  <div class="navbar navbar-fixed-top">
    <div class="navbar-inner">
      <div class="container">
        <div class="user-nav pull-right">
          <%= link_to "Github Login", "/auth/github" %>
        </div>
      </div>
    </div>
  </div>
...
```

现在访问还看不到我们新加的内容,因为没有指定root,rails访问到的还是静态的index页面

演示方便我们把root改为`application#index`并加上需要的action和view

``` vim config/routes.rb
root 'application#index'
```

``` vim app/controllers/application_controller.rb
def index
end
```

```
mkdir app/views/application/
touch app/views/application/index.html.erb
```

现在应该能看到这个link了,点一下如果看类似`No route matches [GET] "/auth/github/callback"`说明已经去github做验证并把结果返回回来了

下一步我们需要验证完的回调

从上面的错误信息可以看到回调地址是`/auth/github/callback`, 所以我们在routes中加一行

```
get "/auth/:provider/callback" => "sessions#create"
```

然后建立controller和action

```
rails g controller sessions create
```

如果你想看看返回的数据什么样子,可以这样写sessions#create

```
def create
  raise env['omniauth.auth'].to_yaml
end
```

这样点登录的link就会把返回的信息显示在页面上

得到返回数据怎么处理就看个人需要了,你可能会想建一个表保存返回的数据, 这里我们建立users表来保存用户的nickname,由于github的nickname就是用户名,是唯一的,但是考虑到以后可能会加入其它的认证如twitter,nickname就可能冲突了,所以我们还加个provider

```
rails g scaffold User nickname provider
rake db:migrate

```

有了保存数据的地方,我们就可以处理返回的验证了

```
def create
  auth = request.env["omniauth.auth"]
  user = User.find_by_omniauth(auth)
  session[:user_id] = user.id
  redirect_to root_url, :notice => "Signed in!"
end
```

把`find_by_omniauth`方法放到了model里,它也很简单,能找到就返回,找不到就创建

```
def self.find_by_omniauth(auth)
  user = User.find_by_provider_and_nickname(auth["provider"], auth["info"]["nickname"])
  user ? user : User.create_with_omniauth(auth)
end

def self.create_with_omniauth(auth)
  create! do |user|
    user.provider = auth["provider"]
    user.nickname = auth["info"]["nickname"]
  end
end
```

需要修改一下layout才可以看到登录成功的提示

``` vim app/views/layouts/application.html.erb
<div class="container" style="margin-top: 60px;">
  <% flash.each do |name, msg| %>
    <div class="alert alert-<%= name == :notice ? "success" : "error" %>">
      <a class="close" data-dismiss="alert">x</a>
      <%= msg  %>
    </div>
  <% end %>
  <%= yield %>
</div>
```

现在登录已经成功了,我们还需要修改一下github登录的link,让它登录后显示用户名和logout

首先我们需要定义一个帮助方法来判断用户有没登录

``` vim app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  protect_from_forgery with: :exception
  helper_method :current_user

  private

    def current_user
      @current_user ||= User.find(session[:user_id]) if session[:user_id]
    end
end
```

然后定义一条logout的路由

```
get "signout" => "sessions#destroy", :as => "signout"
```

以及对应的Action

```
def destroy
  session[:user_id] = nil
  redirect_to root_url, :notice => "Signed out!"
end
```

最后修改layout

```
<div class="user-nav pull-right">
  <% if current_user %>
    Welcome <%= current_user.nickname %>!
    <%= link_to "Sign out", signout_path %>
  <% else %>
    <%= link_to "Github Login", "/auth/github" %>
  <% end %>
</div>
```

基本上这样就算完成了

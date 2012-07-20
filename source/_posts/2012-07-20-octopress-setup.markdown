---
layout: post
title: "Octopress - 像黑客一样写博客"
date: 2012-07-20 11:14
comments: true
categories: 
---

Octopress是一个非常优秀的博客框架,现在已经有越来越多的人从Wordpress转向Octopress,它有很多的优点,但我使用它的原因仅仅是因为,在我看来它非常漂亮

这篇文档主要包括

- 搭建Octopress
- 将博客托管到Github上
- 我对主题做的一些修改

## 搭建Octopress

Octopress需要Git和Ruby 1.9.2,Git的安装比较简单,下面我们看一下Ruby1.9.2的安装(我的系统是Ubuntu12.04)

### 安装Ruby 1.9.2

可以有两种方法,RVM和rbenv,这里介绍rbenv的方法,因为RVM我没有成功(实际上安装Ruby1.9.2我花了好多时间)

#### 首先安装rbenv

下载rbenv

```
cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
```

将`～/.rbenv/bin`加到你的`$PATH`变量里以使你可以使用`rbenv`这个命令

```
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```

添加rbenv init到你的shell以启用shims(什么东西我也不清楚)和自动补全功能

```
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```

启动shell让`$PATH`的修改生效,现在可以使用rbenv了

```
exec $SHELL
```

#### 安装Ruby 1.9.2

```
rbenv install 1.9.2-p290
rbenv rehash
```

确实Ruby 1.9.2已经正确安装(如果没有最好不要继续进行)

```
ruby --version  # Should report Ruby 1.9.2
```

### 搭建Octopress

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
ruby --version  # Should report Ruby 1.9.2
```

安装依赖

```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
```

安装默认Octopress主题

```
rake install
```

Octopress的本地搭建工作已经完成,现在可以做一些简单的测试了

创建一篇博文

```
rake new_post["first post"]
```

这会在`source/_posts`目录下创建一个以markdown为后缀的文件,打开编辑后保存(markdown语法请看[这里](http://equation85.github.com/blog/markdown-examples/))

生成html文件

```
rake generate
```

通过下面的命令就可以在本地预览一下效果了

```
rake preview
```

现在访问[http://localhost:4000](http://localhost:4000)就可以看到新创建的页面了,是不是很漂亮:) 


### 将博客部署到Github上去

Octopress可以托管到[Github](https://github.com),[Heroku](http://heroku.com)或者自己有主机通过rsync部署到上面,rsync的方式我没有试过(我没有主机),我只试过Github和Heroku,个人感觉Heroku更好一点,添加自己的域名要比Github方便,但非常蛋疼的一点是Heroku好像被我们亲爱的祖国墙到了外面,所以只能用Github了

要将Octopress托管到Github上需要在Github上有一个帐号,并且创建了个类似username.github.com的仓库(例如我的帐号名为`williamherry`,那么我就要创建一个名为`williamherry.github.com`的仓库),如果还没有点[这里](https://github.com/new)创建一个

Octopress提供了一些方便的工具来帮你完成托管到Github上的工作,只要在博客目录里执行  
`rake setup_github_pages`,当提示你的时候输入你前面创建的仓库的Git Url, 例如我的为  
`git@github.com:williamherry/williamherry.github.com.git`

注意: 下面命令中的`williamherry`和`username`需要换成你自己的Github帐号名

```
octopress$ rake setup_github_pages
Enter the read/write url for your repository: git@github.com:williamherry/williamherry.github.com.git
 
Added remote git@github.com:williamherry/williamherry.github.com.git as origin
Set origin as default remote
Master branch renamed to 'source' for committing your blog source files
Initialized empty Git repository in /home/williamherry/source/octopress/_deploy/.git/
[master (root-commit) 2a4e9e7] Octopress init
1 files changed, 1 insertions(+), 0 deletions(-)
create mode 100644 index.html
 
---
## Now you can deploy to http://williamherry.github.com with `rake deploy` ##
```

那么上面的命令具体做了那些工作呢? 它把我们克隆来的博客指向了我们新的仓库,它还创建了一个`_deploy`目录来包含所有被部署的文件的新仓库,这个目录所指向的远程仓库和我们的`octopress`目录是同一样,但Checked out的分支是`master`.顺便说一下,我们现在在`source`分支

```
octopress$ git remote -v
octopress    git://github.com/imathis/octopress.git (fetch)
octopress    git://github.com/imathis/octopress.git (push)
origin    git@github.com:williamherry/williamherry.github.com.git (fetch)
origin    git@github.com:williamherry/williamherry.github.com.git (push)
 
octopress$ git branch
* source
 
octopress$ cd _deploy/
octopress/_deploy$ git remote -v
origin    git@github.com:williamherry/williamherry.github.com.git (fetch)
origin    git@github.com:williamherry/williamherry.github.com.git (push)
 
octopress/_deploy$ git branch
* master
 
octopress/_deploy$ cd ..
 
octopress$
```

在我们部署到github之前,可以保存`source`并且把它推到Github上去,注意我们将要推的是`source`分支

```
octopress$ git add .
 
octopress$ git commit -m "Initial blog post."
...
 
octopress$ git push origin source
Counting objects: 3927, done.
Compressing objects: 100% (1412/1412), done.
Writing objects: 100% (3927/3927), 910.08 KiB, done.
Total 3927 (delta 2257), reused 3848 (delta 2203)
To git@github.com:williamherry/williamherry.github.com.git
* [new branch]      source -> source
```

为了保存你的博客,每次改变你都要执行上面的命令

接下来我们部署博客,它所做的是把所有`_deploy`下的东西推到`master`分支上去

```
octopress$ rake deploy
 
## Pushing generated _deploy website
Counting objects: 84, done.
Compressing objects: 100% (74/74), done.
Writing objects: 100% (84/84), 180.40 KiB, done.
Total 84 (delta 2), reused 0 (delta 0)
To git@github.com:williamherry/williamherry.github.com.git
* [new branch]      master -> master
```

一旦Github生成页面(通常一到两分钟)后你访问`http://username.github.com`应该就能看到自己的博客了.并且在https://github.com/username/username.github.com你应该可以看到保存生成的页面文件的`master`分支和保存源码的`source`分支

每次想部署都需要执行上面的命令

如果你有一台新的电脑,你需要做的步骤和上面大体相同,但不是从头开始

```
$ git clone git@github.com:username/username.github.com.git
$ cd username.github.com
username.github.com$ git checkout source
username.github.com$ mkdir _deploy
username.github.com$ cd _deploy
username.github.com/_deploy$ git init
username.github.com/_deploy$ git remote add origin git@github.com:username/username.github.com.git
username.github.com/_deploy$ git pull origin master
username.github.com/_deploy$ cd ..
username.github.com$
```

### 添加自己的域名

如果你有自己的域名,可以很方便的绑定到自己的博客上

首先创建一个`CNAME`文件里面写上你的域名

```
echo 'your-domain.com' >> source/CNAME
git add .
git commit -m 'add my own domain name'
git push origin source
rake generate
rake deploy
```

然后更新你的域名指向

如果你使用一个二级域名(如'blog.williamherry.com'),你只要加一个`CNAME`指向`username.github.com`,如果你使用一个顶级域名,你需要加一条`A`记录指向`207.97.227.245`

注意:域名的修改需要一定的时间,所以耐心点

## 我对主题的修改

我的主题的修改主要参考的是[这篇博文](http://melandri.net/2012/02/14/octopress-theme-customization/) (基本全抄他的,感谢它做了这么漂亮的修改)

此外还参考了[Octopress: Theming & Customization](http://octopress.org/docs/theme/template/)

### 修改导航

默认只有`Blog`和`Archives`,我在前面加了一个`Home`

```
mv source/index.html source/blog/index.html
rake new_page[index.html]
```

你可以修改`source/index.html`,它将在主面显示,注意,这不是`markdown`文件

你需要更新你的Rakefile文件以确实当你更新Octopress时你新的博客索引被保留下来(重要)

{% codeblock vim Rakefile %}
...
blog_index_dir = 'source/blog'
...
{% endcodeblock %}

修改导航加上`Home`并更新`Blog`的url

{% codeblock vim source/_includes/custom/navigation.html %}
<ul class="main-navigation">
  <li><a href="{{ root_url }}/">Home</a></li>
  <li><a href="{{ root_url }}/blog">Blog</a></li>
  <li><a href="{{ root_url }}/blog/archives">Archives</a></li>
</ul>
{% endcodeblock %}

### 修改字体

从[Google Webfonts](http://google.com/webfonts)选择自己喜欢的字体,我比较喜欢`Delius`这个字体,需要改两个文件

{% codeblock vim sass/custom/_fonts.scss %}
$sans: "Delius", sans-serif;
$serif: "Delius", serif;
$heading-font-family: "Delius", sans-serif;
$header-title-font-family: "Delius", cursive, Helvetica, Arial, sans-serif;
{% endcodeblock %}

{% codeblock vim sass/_includes/custom/head.html %}
<!-- this font looks great -->
<link href='http://fonts.googleapis.com/css?family=Delius' rel='stylesheet' type='text/css'>
{% endcodeblock %}

### 修改CSS样式

这个基本是抄别人的,我只是加了`ul`以让列表稍微靠右

{% codeblock vim sass/cuscom/_style.scss %}
@font-face {
  font-family: "League";
  src: url('/font/LeagueGothic.otf');
}

a:visited, #content .blog-index article h1 a:hover { color: #1863A1; }


/* ----- main layout ----- */


html { background: #262C33 url("/images/line-tile.png"); }

body { font-size: 1em; }

body > div { background-image: none; }

body > div > div { background-image: none; }


/* ----- header ----- */


body > header{
  background: none;
  padding: 1.6em 0 1em 0
}

body > header h1{
  font-size: 2.8em;
  padding-left: 20px;
  text-shadow: rgba(0, 0, 0, 0.8) 0 0 8px;
}

body > header h2{
  font-size: 0.5em;
  letter-spacing: 1px;
  margin-top: -1.4em;
  padding-left: 20px;
}

body > nav {
  padding-top: .15em;
  padding-bottom: .15em;
}

body > nav a{
  font-size: 1em;
  line-height: 1.4em;
}

body > nav form .search{
  padding: .2em .3em;
}


/* ----- Content ----- */


#content .blog-index article h1 {
  font-size: 1.8em;
  font-weight: normal;
}

#content .blog-index article h1, #content .blog-index article h1 a, article header h1, article header h1 a{
  color: #555555 !important;
}

h1 { font-size: 1.8em; }

h1 span{
  font-weight: normal;
  color: #E0841B;
}

article h2, article h3, article header h1{
  font-weight: normal;
}

.blog-index article h1 a:hover{ text-decoration: none; }

article p {
  text-align:justify;
  margin-bottom: 1em;
}


/\* ----- Sidebar ----- \*/


aside.sidebar section h1 { letter-spacing: 0.1em; }

aside.sidebar a { text-decoration: none; }

.toggle-sidebar{display: none;}

ul#gh\_repos > li > a{
  display: block;
  font-weight: bold;
  margin-bottom: 0.4em;
}

/* make list looks better */
ul {
  margin-left:18px
}

aside.sidebar{
  -moz-border-radius-topright: 0.4em;
  -webkit-border-radius: 0 0.4em 0 0;
  border-radius: 0 0.4em 0 0;
}


/* ----- Footer ----- */


body > footer{
  -moz-border-radius: 0;
  -webkit-border-radius: 0;
  border-radius:0;
  margin-bottom: 0;
}


/* ----- 404 ----- */


.notfound404 article{
  margin-left: 0 !important;
}


/* ----- Media queries ----- */


@media only screen and (min-width: 550px) {

  body > header h1{
      background: url("/favicon.png") no-repeat 0 8px;
      padding-left: 60px;
  }

  body > header h2 { padding-left: 60px; }

  body > footer { margin-bottom: 3em; }
}

@media only screen and (min-width: 1040px) {

  body > nav {
    -moz-border-radius: 0.4em;
    -webkit-border-radius: 0.4em;
    border-radius:0.4em;
    margin-bottom: 2em;
  }

  body > footer{
      -moz-border-radius-bottomleft: 0.4em;
      -moz-border-radius-bottomright: 0.4em;
      -webkit-border-radius: 0 0 0.4em 0.4em;
      border-radius: 0 0 0.4em 0.4em;
  }

  #main{
      -moz-border-radius-topleft: 0.4em;
      -moz-border-radius-topright: 0.4em;
      -webkit-border-radius: 0.4em 0.4em 0 0;
      border-radius: 0.4em 0.4em 0 0;
  }

  #content{
      -moz-border-radius-topleft: 0.4em;
      -webkit-border-radius: 0.4em 0 0 0;
      border-radius: 0.4em 0 0 0;
  }

  #content .blog-index a[rel="full-article"]{
      -webkit-border-radius: 6px;
      -moz-border-radius: 6px;
      border-radius: 6px;
  }
}
{% endcodeblock %}

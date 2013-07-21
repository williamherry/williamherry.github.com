---
layout: post
title: "日期时间选择插件使用"
date: 2013-07-21 23:22
comments: true
categories: 
---

以前做过这样的东西,花了好多时间,因为要选择到底用那个插件,还有好多插件要只有日期没有时间,今天做这样的东西又花了不少时间,决定记录下来,下一次就简单多了

我使用的是`jQuery`的`datepicker`加上[jQuery-Timepicker-Addon](https://github.com/trentrichardson/jQuery-Timepicker-Addon), 示例代码在这里: [datetimepicker-demo](https://github.com/williamherry/datetimepicker-demo),懒得看详细的步骤可以直接看代码

- 首先创建项目

```
rails new datetimepicker-demo
```

- 然后我创建了一个叫activity的scaffold,插件会用到start_at这个字段上
```
rails g scaffold activity title content:text start_at:datetime
rake db:migrate
```

- 下来我们要把需要的插件加进来,jquery-ui我用一个非常方便的gem: [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails), 只要加到`Gemfile`里`gem jquery-ui-rails`,运行`bunder`就行了

- [jQuery-Timepicker-Addon](https://github.com/trentrichardson/jQuery-Timepicker-Addon)我是自己下载下来把需要的文件复制到项目的vender目录下,需要用到三个文件:`jquery-ui-timepicker-addon.js`, `jquery-ui-timepicker-addon.css`和`i18n`目录下的`jquery-ui-timepicker-zh-CN.js`, js文件复制到`vendor/assets/javascripts/`, css文件复制到`vendor/assets/stylesheets/`

- 需要的文件都加进来了,下面我们需要在include他们

``` vim app/assets/javascripts/application.js
//= require jquery
//= require jquery.ui.datepicker
//= require jquery.ui.slider
//= require jquery-ui-timepicker-addon
//= require jquery-ui-timepicker-zh-CN
```

``` vim app/assets/stylesheets/application.css
 *= require jquery.ui.datepicker
 *= require jquery.ui.slider
 *= require jquery-ui-timepicker-addon
```

- 准备工作都做完了,下来就是使用了

``` vim app/views/activities/_form.html.erb
  <div class="field">
    <%= f.label :start_at %><br>
    <%= f.text_field :start_at, :id => "datetimepicker" %>
  </div>
```

``` vim app/assets/javascripts/activities.js.coffee
$ ->
  $("#datetimepicker").datetimepicker
    stepMinute: 5
    dateFormat: "yy-mm-dd"
```

更多参数和例子可以看[这里](http://trentrichardson.com/examples/timepicker/#tp-options)

这样就算完成了,但是虽然加了那个i18n的文件,月和周的显示还是英文, 这可以直接在js文件里加,也可以在`vendor/assets/javascripts/jquery-ui-timepicker-zh-CN.js`中加上这两行

```
  monthNames: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'],
  dayNamesMin: ['周日', '周一', '周二', '周三', '周四', '周五', '周六'],
```

### 资源:
- [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails)
- [jquery datepicker](http://jqueryui.com/datepicker/)
- [jQuery Timepicker Addon](https://github.com/trentrichardson/jQuery-Timepicker-Addon)

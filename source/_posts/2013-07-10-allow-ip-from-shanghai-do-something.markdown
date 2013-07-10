---
layout: post
title: "只允许上海的用户注册"
date: 2013-07-10 13:33
comments: true
categories: 
---

刚开始听到老板说这个需求的时候,我还傻呼呼一直在想要怎么得到所有上海的IP地址,都打算试着写爬虫去爬这些数据,事实上我只要判断一个客户端的IP是不是属于上海就行了,而这个动作可以由一些第三方提供的API来做,像百度就有提供这个的服务

转过弯来就比较简单了,我的方法在`app/controller/application_controller.rb`文件里加一个helper方法

``` vim app/controller/application_controller.rb
class ApplicationController < ActionController::Base
  helper_method :is_shanghai_ip?
...
  def is_shanghai_ip?(ip)
    uri = "http://api.map.baidu.com/location/ip?ak=your_api_key&ip=#{ip}&coor=bd09ll"
    resp = Net::HTTP.get_response(URI.parse(uri))
    data = resp.body
    result = JSON.parse(data)
    return false if result['status'] = 1
    result['content']['address'] == "上海市" ? true : false
  end
...
end
```

然后在你要做控制的action中调用这个方法,比如我们要限制只有上海的用户可以注册

``` vim app/controller/users_controller.rb
def sign_up
  unless is_shanghai_ip?(request.remote_ip)
    redirect_to "/about/us" and return
  end
  ...
end
```

百度提供的这个服务的地址是: http://developer.baidu.com/map/ip-location-api.htm

另外有一个叫`geocoder`的gem非常强大,它直接在`Rack::Request`里塞一个`location`的方法,你可以直接在controller里这样调用

```
city = request.location.city
country = request.location.country_code
```

此外它还有一些非常强大的功能如查找经纬度,查找附近,查找距离等等,更详细的信息可以访问它的[官方网站](http://www.rubygeocoder.com/), [RailsCast](http://railscasts.com/episodes/273-geocoder)还有一期是介绍它的

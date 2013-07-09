---
layout: post
title: "IE10下图片灰度处理"
date: 2013-07-09 17:18
comments: true
categories: 
---

其它浏览器或版本的图片灰度处理可以看[这里](http://www.karlhorky.com/2012/06/cross-browser-image-grayscale-with-css.html)

上面的方法对IE10是没有效果的,Google了好久发现只有用svg才对IE10才有作用,想知道svg是什么请看[这里](http://www.w3school.com.cn/svg/svg_intro.asp)

下面的代码就可以对图片做灰度的处理

``` vim ~/tmp/index.html
<svg xmlns="http://www.w3.org/2000/svg" id="svgroot" viewBox="0 0 100 100" width="100" height="100">
  <defs>
    <filter id="filtersPicture">
      <feComposite result="inputTo_38" in="SourceGraphic" in2="SourceGraphic" operator="arithmetic" k1="0" k2="1" k3="0" k4="0" />
      <feColorMatrix id="filter_38" type="saturate" values="0" data-filterid="38" />
    </filter>
  </defs>
  <image filter="url(&quot;#filtersPicture&quot;)" x="0" y="0" width="100" height="100" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://williamherry.com/images/chef/attributes.jpg" />
</svg>

<img src="http://williamherry.com/images/chef/attributes.jpg">
```

为了做比较我在后面放了一个不做处理的图片,你可以把上面的代码保存为`.html`格式文件用浏览器打开看看

上面的代码可以达到我们期望的效果,但是又有新的问题,我给它的高和宽都设置的100,但从浏览器来看它好像并不是正方形.

这里的问题是它是按比例缩放的,例如图片是100X1000,如果我给它设置成100X100,那么最终图片的显示可能是10X100,没有完全填满100X100

对于这个问题我还没有找到完美的解决办法,但有一个办法已经可以满足我自己的需求,这个方法是在`<image `后面加上`preserveAspectRatio="xMidYMid slice"`

还用上面的例子,它会把图片按比较放到和短的边一样, 这里是100X1000,然后把100X100的框放中间,多余的裁掉,剩下的就是显示的部分

{% img /images/svg/preverse-aspect-ratio.svg %}

这样只要保存图片的比例和设置的很接近就可以基本上解决这个问题

更多`preserveAspectRatio`的信息请看[这里](http://www.w3.org/TR/SVG/coords.html)

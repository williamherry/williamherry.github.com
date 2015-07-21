---
layout: post
title: "简单方法实现拖拽排序"
date: 2015-07-21 07:01
comments: true
categories: 
---

拖拽排序就是类似下面图的效果

{% img /images/sortable.gif %}

要实现以上的效果，有很多的选择，最流行的是使用[act_as_list](https://github.com/swanandp/acts_as_list)gem，它的问题是要修改一条记录的位置，所有跟在它后面的记录都要修改位置。这样如果你有很多条数据，排序就会变的消耗极大和很慢

另一个办法是使用[ranked-model](https://github.com/mixonic/ranked-model)这个gem，它只需要执行一条操作就可以了，它是每次给要排序的记录找一个前后两个数的中间值，它使用的数范围很宽，-8388607 to 8388607，使用的过程中也发现了问题，就是有时候排序是失败的，我没找到具体的原因，猜测可能是找的中间值有问题

后来我发现了一个更简单的方式实现这样的效果，甚至都不用第三方gem（jQuery都sortable还是要用都，前面两个也用）。它都原理是：任何两个浮点数中间都包括无数个浮点数，简单说就是浮点数可以一直二分下去：1 > 0.5 > 0.25 -> 0.125

看一下如何实现

首先再要排序都model里加一个字段用来记录位置，类型为**float**

``` ruby db/migrate/20140908010519_add_position_to_things.rb
class AddRowOrderToThings < ActiveRecord::Migration
  def change
    add_column :things, :position, :float
  end
end
```

然后再该model都controller里添加更新都action

``` ruby controllers/admin/things_controller.rb
class ThingsController < ApplicationController

def update
  @thing = Thing.find(thing_params[:thing_id])
  @thing.update thing_params

  render nothing: true # this is a POST action, updates sent via AJAX, no view rendered
end

private

def thing_params
  params.require(:thing).permit(:position)
end
```

后端都工作以及做完里，下面就是前端了，首先引入jquery-ui/sortable

``` ruby assets/javascripts/admin/application.js
//= require jquery-ui/sortable
```

下来就是最重要都部分了，在客户断直接计算好要排记录它断位置值，传给后台更新

``` javascript assets/javascripts/admin/update_things_position.js.coffee
if ($('#sortable').length > 0) {
  $('#sortable').sortable({
    axis: 'y',
    cursor: 'move',

    sort: function(e, ui) {
      ui.item.addClass('active-item-shadow');
    },

    update: function(e, ui) {
      itemId = ui.item.data('item-id');
      prevPosition = parseFloat(ui.item.prev().data('position'));
      nextPosition = parseFloat(ui.item.next().data('position'));

      if (isNaN(prevPosition)) {
        position = nextPosition - 1;
      } else if (isNaN(nextPosition)) {
        position = prevPosition + 1;
      } else {
        position = (nextPosition + prevPosition) / 2
      }

      $.ajax({
        type: 'POST',
        url: "/things/" + itemId,
        method: "PATCH",
        data: { thing: { position: position } },
        success: function() {
          ui.item.attr('data-position', position);
        }
      });
    }
  });
}
```

页面渲染断时候我把记录当前的position值放在attr里，当移动一条记录后，找到这条记录前后记录的position值，如果前面没有记录，就是移到第一个了，那么把它的position设为后面一条记录position值减1，如果后面没有记录了，就是移动到最后一个，那么把它到position设为前面一条记录position值加1，如果前后都要，前后记录position值相加除2

这样position值到变化好像是指数级的，每次都除以2，会不会除到小到没有呢？我们来看一下

如果每次都移动到最前面或最后吗，都是加1减1到操作，完全不用担心

看一下每次都移动到第二个位置到情况，假设有4条数据，它们到初始position值为0，1，2

第一次把第三个数移动到第二个位置，它到值就是(0+1)/2 = 0.5，现在前两个数到position为0，0.5

第二次把第三个数移动到第二个位置，它到值就是(0+0.5)/2 = 0.25, 现在前两个数到position为0，0.25

第三次把第三个数移动到第二个位置，它到值就是(0+0.25)/2 = 0.125, 现在前两个数到position为0，0.125

应该找到规律了，第n次移动之后第二个数为0.5的n次方，那么排序多少次会出现溢出的问题呢？我把这个问题留给读者 :)


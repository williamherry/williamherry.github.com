---
layout: post
title: "Memcached Cache Analysis"
date: 2014-09-10 22:55
comments: true
categories: 
---

利用munin这样的工具可以查看到Memcached整个缓存的命中率,有时候可能会需要知道不同的缓存各自的命中率如何,这时候可以利用Rails的[ActiveSupport::Notifications](http://edgeguides.rubyonrails.org/active_support_instrumentation.html#active-support"")来帮助我们分析

```
CacheLogger = Logger.new("log/cache.log")

ActiveSupport::Notifications.subscribe /cache_\S*\.active_support/ do |name, start, finish, id, payload|
  CacheLogger.info "#{name} #{start} #{payload[:key]}"
end
```

上面的代码会把所有的缓存的请求记录到一个日志文件里面,可以用下面的脚本去生产分析报告(from @soffolk)

```
#! /bin/sh

# ./parse_cache cache.log courses

read=`cat $1 | grep $2 | grep cache_read.active_support | wc -l`
write=`cat $1 | grep $2 | grep cache_write.active_support | wc -l`
read_multi=`cat $1 | grep $2 | grep cache_read_multi.active_support | grep -o '[as|to]-json' | wc -l`
fetch_hit=`cat $1 | grep $2 | grep cache_fetch_hit.active_support | wc -l`
all=$((read+read_multi))
if [ $all = 0 ]; then
  ratio=0
else
  ratio=$(echo "scale=6; $(($all-$write))/$all*100" | bc -l)
fi

echo "Read:         $read"
echo "Write:        $write"
echo "Read Multi:   $read_multi"
echo "Fetch Hit:    $fetch_hit"
echo "All:          $all"
echo "Hit Ratio:    $ratio%"
```

另外你可能想看一下是不是分给Memcached的内存已经用光了,不断有少访问的item被挤出去

可以用memcdump命令dump出所有的keys

```
memcdump --servers=localhost | sort > keys1.dump
```

过几秒在dump一次

```
memcdump --servers=localhost | sort > keys2.dump
```

使用comm命令可以查看是否有变化

```
comm -23 keys1.dump keys2.dump > only_in_one.dump
comm -13 keys1.dump keys2.dump > only_in_two.dump
```

实际上从memcached的内置命令stats就可以看到内存已经不够用的,一直有item被挤出去

```
...
STAT hash_is_expanding 0
STAT expired_unfetched 4383
STAT evicted_unfetched 7335517 #已驱逐但未获取的对象数目 
STAT bytes 253815528
...
```

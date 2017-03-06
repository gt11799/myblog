title: Python使用datetime报AttributeErrors的解决办法
date: 2016-12-16 23:44:54
tags: Python
---

最近写了一个多线程写数据的脚本，其中使用`datetime.strptime`的时候，报了一个`AttributeError: _strptime_time`的错误。
<!--more-->

对于这个错误，本地无论如何都不能重现，百思不得其解。经过小容容的指点，发现了原来`datetime.strptime`线程不安全，文章[在此](http://bugs.python.org/issue7980)

这个错误很容易复现，多线程中只调用`datetime.strptime`，错误是必现的。代码如下(原谅我懒，直接借用了别人的代码)

```
import time
import thread

def f():
for m in xrange(1, 13):
for d in xrange(1,29):
    time.strptime("2010%02d%02d"%(m,d),"%Y%m%d")

for _ in xrange(10):
thread.start_new_thread(f, ())
time.sleep(3)
```

`datetime.strptime`是基于`time.strptime`的，代码中就都使用`time.strptime`来代替。解决的办法也很简单，在线程外先使用一次`datetime.strptime`，或者`import _strptime`

```
import time
import thread
import _strptime

def f():
for m in xrange(1, 13):
for d in xrange(1,29):
    time.strptime("2010%02d%02d"%(m,d),"%Y%m%d")

for _ in xrange(10):
thread.start_new_thread(f, ())
time.sleep(3)
```

代码跟过去，发现`_strptime`这个module里，有几个可疑的地方。一是定义了几个全局变量，第一个是线程锁，一个是正则的缓存

```
_cache_lock = _thread_allocate_lock()
_TimeRE_cache = TimeRE()
```

而`_strptime`函数内部，首先就用到了这两个变量

```
global _TimeRE_cache, _regex_cache
with _cache_lock:
    if _getlang() != _TimeRE_cache.locale_time.lang:
        pass
```

我怀疑是多线程使用`_strptime`的时候，初始化撞在了一起，产生了两个线程锁，而后面的操作，又是线程不安全的。这次就不讨论了。

让我大跌眼镜的是，`__builtin__`的一个使用很普遍的function，竟然是线程不安全的。

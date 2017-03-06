title: Python如何在函数内获取调用者信息
date: 2016-02-04 00:00:44
tags: Python
---

写了一个公用的生成Excel的函数，如何根据调用者不同，生成不同名字的文件呢？当然最好的办法还是显式的传入函数

<!--more-->

函数的调用其实是一个堆栈，可以用traceback的extract_stack或者print_stack方法去查看

```py
import traceback

def func(x):
  print "result: %s" % x
  print traceback.extract_stack()    

def func1(x):
  func(x)

def func2(x):
  func1(x)

if __name__ == '__main__':
  func2(90)
```

以上会输出

    [('traceback_p.py', 18, '<module>', 'func2(90)'), ('traceback_p.py', 14, 'func2', 'func1(x)'), ('traceback_p.py', 10, 'func1', 'func(x)'), ('traceback_p.py', 6, 'func', 'print traceback.extract_stack()')]

可以看出列表每个元素分别是文件名，行数，函数名，调用方式，而且顺序恰好是堆栈的底到顶端
我们修改一下代码，就可以知道当前的函数名，以及调用者的函数名称了。

```python
import traceback

def func(x):
  print "result: %s" % x
  stack = traceback.extract_stack()
  print "where i am"
  item = stack[-1]
  print "which file: %s" % item[0]
  print "line: %s" % item[1]
  print "function name: %s" % item[2]
  print "how call: %s" % item[3]

  print "\n"

  print "where call the function"
  item = stack[-1]
  print "which file: %s" % item[0]
  print "line: %s" % item[1]
  print "function name: %s" % item[2]
  print "how call: %s" % item[3]

def func1(x):
  func(x)

def func2(x):
  func1(x)

if __name__ == '__main__':
  func2(90)
```


结果如下：

    result: 90
    where i am
    which file: traceback_p.py
    line: 5
    function name: func
    how call: stack = traceback.extract_stack()


    where call the function
    which file: traceback_p.py
    line: 5
    function name: func
    how call: stack = traceback.extract_stack()

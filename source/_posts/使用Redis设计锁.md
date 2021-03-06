title: 使用Redis设计锁
date: 2017-11-15 23:04:22
tags: Python
---

使用Redis来实现分布式锁，发现自己的设计，还是有漏洞 <!--more-->

## 完美做法之SETNX

`SETNX` 是`set if not exists`的缩写，会返回整数值

- 1，当值未被设置
- 0，值已经被设置

于是我们可以这么设置锁

- 进程1`SETNX KEY`返回1，则获得锁
- 进程2`SETNX KEY`返回0，则未获得锁，继续等
- 进程1结束，`DEL KEY`

进程死锁怎么办?

在获得锁之后，设置一个redis key的过期时间。


上面的方法有个问题，一个进程一旦超时，删除的锁其实是别的进程的锁，这个时候要怎么办呢？

再添加个定时器，知晓自己是否超时，决定是否执行`DEL`


## 最完美做法之SETNX EXPIRE

来自[这里](http://blog.csdn.net/lihao21/article/details/49104695)

把expire记在value上

- 进程1`SETNX KEY`返回1，则获得锁，把值设为expire
- 进程2`SETNX KEY`返回0，说明锁在，此时`GET KEY`判断expire是否超时
- 进程2判断已经超时，则`DEL KEY`

这里也有个问题，假设进程3同时也判断超时，会执行跟进程2一样的操作，会同样获得锁

- 进程2判断已经超时，则`GETSET KEY`，`GETSET`会返回之前的值，如果`now > value`才是拿到锁
- 进程3判断已经超时，则`GETSET KEY`， 此时拿回的值，比目前的时间大，则不会获得锁。因为这是同时并发的行为，重新设置了一个稍大的expire是可以接受的

## 次完美做法之集合

可以使用redis的集合的唯一性来做，集群也应该没问题

- 进程1拿到锁，`SADD KEY 1`，往集合中添加元素1
- 进程2`SADD KEY 1`，返回0，则未获得锁
- 进程1做完，`SREM KEY 1`
- 此时进程2`SADD KEY 1`，返回1，则获得锁

至于进程死锁的问题，依然只能通过给key设置expire的方式来做，依然需要在进程结束时，判断是否超时

## 一般做法INCR

- 进程1拿到锁，`INCR KEY`得到1
- 进程2`INCR KEY`得到2，则未获得锁
- 进程1释放锁，`DEL KEY`

问题很多，锁对应的值会不断的增大，依然要在进程中判断是否超时

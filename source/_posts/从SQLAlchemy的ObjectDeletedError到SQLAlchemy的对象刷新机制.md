title: 从SQLAlchemy的ObjectDeletedError到SQLAlchemy的对象刷新机制
date: 2016-10-25 23:14:51
tags: [Python,数据库]
---

我们的系统使用的是Flask+SQLAlchemy，为了使用MySQL主从，我们使用了数据库代理。自此，ObjectDeletedError开始越来越多。
<!--more-->

假设现在我们定义了一个model对象Staff，我们取出两个对象

        s1 = Staff.query.get(1)

现在我们把s1从数据库直接删掉，然后执行

        db.session.commit()  # db是连接数据库的对象

再在命令行里引用s1，就会报ObjectDeletedError。

那SQLAlchemy是怎么判定什么时候会去重新取这个对象呢？

我们继续上述操作

        s1 = Staff.query.get(1)
        s2 = Staff.query.get(2)
        db.session.commit()

如果我们打开数据库的log，会发现，引用s1对象会重新去数据库里查询。
我们保留了s2对象，查看一下s2对象有什么问题。

        dir(s2)

有个_sa_instance_state很可疑，然后我们继续查看

        dir(s2._sa_instance_state)

发现有个expired属性，查看了一下，果然是True。表示这个对象已经过期了。
其中s2._sa_instance_state._expire 是个对象，里面应该有对象过期的机制，我猜是SQLAlchemy在内存里包含了一个版本，commit了之后，会将落后的版本设置为过期。

- - -

因此可以判定，当创建一个对象，commit了之后，创建的时候init的对象已经过期，会重新去数据库取，读取的请求落到了从库，有一定概率，从库并未同步过来，返回NULL，也就导致了ObjectDeletedError错误

- - -

怎么解决这个问题呢？

        db = SQLAlchemy(session_options={'expire_on_commit': False})

在实例化db对象的时候，传入expire_on_commit=False，这样就可以让内存中的对象不过期，就可以防止ObjectDeletedError了。

虽然会在某些情境下导致数据不一致，但是这样更符合直觉，毕竟，取出的对象，就保持他取出来的时候的样子就行，如果我想要此时此刻最新的，重新query一次好咯。

- - -

感谢小容容，老司机

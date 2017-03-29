title: 如何配置svn
date: 2017-03-29 18:30:08
tags: 运维
---

设计师想要一个svn来存放设计材料，于是开搞。<!--more-->

这篇文章不是手把手说怎么弄svn，只是记录下自己遇见的坑。

首先安装svn

        sudo apt install subversion apache2 libapache2-svn

再创建一个svn的home，然后创建版本库

        svnadmin create /var/svn/project

虽然svn给出了官方的权限，实测要改成777
然后进入项目目录的conf目录下，假设是上面的例子

        cd /var/svn/project/conf

编辑`svnserve.conf`，把下面几项的注释去去掉

        anon-access = none
        auth-access = write
        password-db = passwd

编辑`passwd`，加上用户名和密码

        username = password

编辑`authz`，加上组和组权限

```
[groups]
root = username

[/]
@root = rw
```

我就是这里出错的，定义目录权限的时候，原来给出的是

```
[repository:/foo/bar]
```

实测，我如果没去掉repository，会一直提示认证失败

然后就可以起server了

        svnserve -d -r /var/svn

-r 后面跟的是根目录

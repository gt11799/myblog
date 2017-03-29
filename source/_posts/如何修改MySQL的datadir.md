title: 如何修改MySQL的datadir
date: 2017-03-29 18:44:46
tags: 运维
---

部署一个mysql，应该算是基本功吧？可是我搞很多次才成功的 <!--more-->

我在网上尝试了很多教程，觉得[这个](https://www.digitalocean.com/community/tutorials/how-to-move-a-mysql-data-directory-to-a-new-location-on-ubuntu-16-04)最好

MySQL安装完毕之后，会自动启动。先停机，然后修改配置。新版的目录一般都是 `/etc/mysql/my.cnf`，修改datadir到相应的目录

        sudo service mysql stop

在配置文件中修改

        datadir   = /data/mysql/mysql

修改目标目录的权限

        sudo chown -R mysql:mysql /data/mysql

然后把之前的数据转移到这个目录。推荐使用rsync，可以保留之前的文件的权限。我第一次不成功，就是因为有部分文件应该root用户的，而我给改成了mysql

        sudo rsync -av /var/lib/mysql /data/mysql

然后备份

        sudo mv /var/lib/mysql /var/lib/mysql.bak

然后需要修改apparmor的配置，apparmor是ubuntu上安全相关的模块，有的教程推荐修改白名单，我之前尝试失败了，感觉设置别名更好

        sudo vim /etc/apparmor.d/tunables/alias

然后加入这样一行，注意逗号

        alias /var/lib/mysql/ -> /data/mysql/mysql/,

然后重启apparmor服务

        sudo service apparmor restart

因为mysql启动的时候会去检查`/var/lib/mysql`目录，所以我们依然要创建这个目录

        sudo mkdir /var/lib/mysql/mysql -p

我忘记要不要改权限了，改了也不麻烦。
然后就可以重启mysql了。

        sudo service mysql start
        sudo service mysql status

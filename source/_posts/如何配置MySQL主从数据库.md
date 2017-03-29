title: 如何配置MySQL主从数据库
date: 2017-03-29 18:58:36
tags: 运维
---

运维这种东西，时间只要久一点就会忘的一干二净，要把好的教程给记下来，也要自己记下来。万一教程的网站不在了呢 <!--more-->

继续上面的教程，配置MySQL的主从。试过几个教程，[这个](https://www.digitalocean.com/community/tutorials/how-to-set-up-master-slave-replication-in-mysql)最好。
首先自然是配置好两台MySQL数据库。

我们假设主从数据库分别是

        10.0.0.1 - Master
        10.0.0.2 - Slave

### 配置主库

虽然教程推荐绑定IP，但是我是如下配置的，让任意IP都可以访问

        bind-address   = 0.0.0.0

配置server_id和binlog的目标，主从是依赖binlog来同步的

        server-id         = 1
        log_bin           = /var/log/mysql/mysql-bin.log
        binlog_do_db      = sync_db

然后重启mysql

        sudo service mysql reload

连接上mysql，然后新建一个用户，用于从库访问

        GRANT REPLICATION SLAVE ON *.* TO 'slave_user'@'%' IDENTIFIED BY 'password';
        FLUSH PRIVILEGES;

然后切换到要同步的库，查看binlog的版本

        use sync_db;
        show master sttus;

看到目前的binlog版本是`mysql-bin.000001`

```
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 | newdatabase  |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

导出目前的数据库的所有数据

        mysqldump -u root -p --opt sync_db > sync_db.sql

### 配置从库

导入在主库导出的数据库数据

        create database sync_db;
        source sync_db.sql;

然后修改数据库的配置

        server-id     = 2
        relay-log     = /var/log/mysql/mysql-relay-bin.log
        log_bin       = /var/log/mysql/mysql-bin.log
        binlog_do_db  = sync_db

然后重启MySQL，连接到MySQL上之后，修改Master的配置

        CHANGE MASTER TO MASTER_HOST='10.0.0.1',MASTER_USER='slave_user', MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=107;
        start slave;

查看Slave的状态

        show slave status;

我一般都是看`seconds_behind_master`一栏不是`NULL`就ok了

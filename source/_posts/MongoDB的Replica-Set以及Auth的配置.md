title: MongoDB的Replica Set以及Auth的配置
date: 2017-01-13 00:08:50
tags: [运维,Python]
---

MongoDB事件出现后，公司要给MongoDB加Auth，于是我就调研了一番。
<!--more-->

现在MongoDB在生产中一般使用Replica Set的方式部署，如果一台宕机，另外一台Secondary会变成Master继续服务，提高可用性。

使用docker搭个集群测试，首先建个network bridge

```
docker network new mongo-network
```

然后就是运行MongoDB的容器，集群名为test-rep

```
docker run --rm -it --name mongo1 --net=mongo-network mongo --replSet test-rep
docker run --rm -it --name mongo2 --net=mongo-network mongo --replSet test-rep
docker run --rm -it --name mongo3 --net=mongo-network mongo --replSet test-rep
```

然后再运行一个连接到上述三个MongoDB的容器

```
docker run --rm -it --name mongo-client --net=mongo-network mongo /bin/bash
```

然后在容器中执行

```
mongo --host mongo1
```

发现连接上了，说明MongoDB的配置没有问题，然后是配置Replica Set。Replica Set要求配置的members中不能有localhost，而我配置为mongo2，mongo3这种一直都报类似的错误，我索性找出了几个容器的IP，配置上去

```
docker network inspect mongo-network
```

可以看到几个容器的IP

![容器的IP地址](http://7xlo8n.com1.z0.glb.clouddn.com/WX20170113-002639.png)

然后就可以使用IP地址配置了

```
config = {_id:"test-rep", version:1, members:[{_id:0, host:"172.19.0.5:27017", priority:5}, {_id:1, host:"172.19.0.3:27017", priority:2}, {_id:2, host:"172.19.0.4:27017", priority:3}]}
rs.initiate(config)
```

再去看mongod的log，发现集群同步成功

![配置成功](http://7xlo8n.com1.z0.glb.clouddn.com/WX20170113-013308.png)

把mongo1停掉，mongo2会成为primary

![mongo2的log](http://7xlo8n.com1.z0.glb.clouddn.com/WechatIMG1551.jpeg)
![mongoshell](http://7xlo8n.com1.z0.glb.clouddn.com/WechatIMG1562.jpeg)

然后按照MongoDB的文档增加用户

```
db.createUser(
{
user: "admin",
pwd: "admin",
roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
}
)
```

重启mongod的进程，增加--auth参数，表示启用权限校验

```
docker run --rm -it --name mongo1 --net=mongo-network mongo --replSet test-rep --auth
docker run --rm -it --name mongo2 --net=mongo-network mongo --replSet test-rep --auth
docker run --rm -it --name mongo3 --net=mongo-network mongo --replSet test-rep --auth
```

发现一直报`Error in heartbeat request to 172.19.0.5:27017; Unauthorized: not authorized on admin to execute command`的错误，查了很久，发现Replica Set要使用keyFile的校验方式，让集群的member之间同步，也就是说，通过keyFile获得__system用户在local上的权限。local存放着Replica Set的配置和同步信息。
MongoDB官方推荐的keyFile的生产方式

```
openssl rand -base64 756 > <path-to-keyfile>
chmod 400 <path-to-keyfile>
```

先结束掉mongod的进程，因为要放入keyFile，于是我启动docker的时候，默认不启动mongod

```
docker run --rm -it --name mongo1 --net=mongo-network mongo /bin/bash
```

容器里没有装openssl，我偷懒使用了以下命令

```
echo 'I8au1RERvEQkIiIB7vhTMhfceA8oH/L0mT6xxeVgaJg/mYnnZe89dGWjMrQSXI7A' > /data/key_file
chmod 400 /data/key_file
```

然后启动mongod进程

```
mongod --replSet test-rep --auth --keyFile=/data/key_file
```

在mongo2，mongo3上按照上述命令，依次启动。发现漂亮的同步成功的标志

![Auth启动成功](http://7xlo8n.com1.z0.glb.clouddn.com/WX20170113-005044.png)

收工

------

生产上的MongoDB，切换到需要Auth，是否可以在不停机的状况下进行呢？

某同学猜想，mongod不采用Auth的时候，客户端使用密码，可不可以呢？Python连接MongoDB的代码很简单

        pymongo.MongoClient('mongodb://user:user1@mongo1:27017,mongo2:27017,mongo3:27017/mydb?authMechanism=SCRAM-SHA-1')

访问mydb的时候，直接就抛`Authentication failed`错误了。如果我先添加了user呢？在mongo shell中执行

```
db.createUser(
{
user: "user",
pwd: "user1",
roles: [ { role: "readWrite", db: "mydb" },
     { role: "readWrite", db: "mydb2" } ]
}
)
```

刚刚的admin，只是访问admin库的用户名和密码，可以管理用户信息，user用户可以用来读写相应的库。此时，mongod依然没有使用`--auth`启动，因此是没有权限检查的，再次连接，一切正常。

因此配置步骤如下

- 创建MongoDB用户

```
use admin
db.createUser(
{
user: "admin",
pwd: "admin",
roles: [ { role: "userAdminAnyDatabase", db: "admin" },
{ role: "clusterAdmin", db: "admin" },
 ]
}
)

use mydb
db.createUser(
{
user: "user",
pwd: "user1",
roles: [ { role: "dbOwner", db: "mydb" },
     { role: "dbOwner", db: "mydb2" } ]
}
)
```

- 修改应用，更改MongoDB的URI

```
pymongo.MongoClient('mongodb://user:user1@mongo1:27017,mongo2:27017,mongo3:27017/mydb?authMechanism=SCRAM-SHA-1')
```

- mongod增加keyFile

```
openssl rand -base64 756 > /data/key_file
chmod 400 /data/key_file
```

- 把key_file上传到其他mongod服务器上，修改mongod配置，一般是/etc/mongodb.conf

```
security:
authorization: enabled
keyFile: /data/key_file
```

- 然后同时重启三台mongod

这样只有重启的那一刹那不可用

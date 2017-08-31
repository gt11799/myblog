title: docker的一系列操作
date: 2017-08-31 22:12:51
tags: docker
---

记录一些常用的操作，备忘<!--more-->

docker-compose-file里volumes, hosts, 老是忘

```
volumes:
 - host_dir:container_dir
ports:
 - host_port:container_port
```

接入一个已经存在的network

```
docker run -it --rm --network=postgres_default postgres psql -h postgres1 -U postgres db
```

导入postgres的数据, 创建一个临时镜像，然后在里面操作

```
docker run -it --rm --network=postgres_default -v host_dir:/tmp postgres /bin/bash
```

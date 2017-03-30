title: 使用docker配置sentry
date: 2016-08-10 22:54:46
tags: [运维,Docker]
---

最近sentry需要迁移服务器，偶然发现使用docker部署sentry更方便，日后维护起来也更容易。<!--more-->

基本就是按照[官方](https://hub.docker.com/_/sentry/)的操作

启动一个redis

        docker run -d --name sentry-redis redis

启动一个postgres,使用了默认密码。。如果想要更改数据的存储位置，可以加上 -v /data/postgres:/var/lib/postgresql/data

         docker run -d --name sentry-postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=sentry postgres

建表，更新初始数据, 生成一个secret key，因为postgres使用的是默认密码，所以这里不用传 postgres的密码，这里注意postgres和redis的名称对应

        docker run -it --rm -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-postgres:postgres --link sentry-redis:redis sentry upgrade

启动sentry，这里我也传了email的配置，结果总是发邮件超时。这里我把docker中的9000端口映射到了宿主的9000端口

        docker run -d --name my-sentry -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-redis:redis --link sentry-postgres:postgres -p 9000:9000 sentry

启动sentry的cron 和 celery worker

        docker run -d --name sentry-cron -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-postgres:postgres --link sentry-redis:redis sentry run cron
        docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY='<secret-key>' --link sentry-postgres:postgres --link sentry-redis:redis sentry run worker -c 4

celery worker可以传入 `CELERYD_MAX_TASKS_PER_CHILD`,`CELERYD_TASK_SOFT_TIME_LIMIT`,`CELERYD_PREFETCH_MULTIPLIER`等等环境变量用来优化celery(还未测试是否会生效)

如果要重启某个服务

        docker restart <docker name>

如果要查询数据

        docker run -it --rm --link sentry-redis:redis redis redis-cli -h redis
        docker run -it --rm --link sentry-postgres:postgres postgres psql -U sentry -d sentry -h postgres -p

docker把很多服务都封装的很好，不用考虑具体实现，真是吾等福音。

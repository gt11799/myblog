title: 如何一键建立v p N
date: 2015-12-27 19:34:15
tags: 运维
---
之前配过pptp, 一直都不成功。左耳朵耗子分享了一个docker镜像，可以直接pull下来，就可以一键搭v p N
<!--more-->

首先，找一台国外的VPS，装个最新版的Ubuntu/CentOS，然后执行：

        docker run -d -p 500:500/udp -p 4500:4500/udp -p 1701:1701/tcp -e PSK=共享密码 -e USERNAME=用户名 -e PASSWORD=密码 siomiz/softethervpn

（完）

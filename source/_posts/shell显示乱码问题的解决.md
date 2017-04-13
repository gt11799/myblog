title: shell显示乱码问题的解决
date: 2017-04-13 18:03:39
tags: 运维
---

今天遇见了一件离奇事，我的远端服务器突然显示不出一些常见字符了，或者给显示成别的字符了。<!--more-->特征是这样的

![](http://7xlo8n.com1.z0.glb.clouddn.com/WechatIMG1959.jpeg)
![](http://7xlo8n.com1.z0.glb.clouddn.com/WechatIMG1978.jpeg)
![](http://7xlo8n.com1.z0.glb.clouddn.com/WechatIMG1992.jpeg)

试过修改locale，甚至是重启过都不好用。最后经师兄指点，

        reset

使用这个命令就搞定了。
[出处](https://www.oschina.net/question/12_150975)

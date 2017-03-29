title: 如何安装cx_Oracle库
date: 2017-03-29 19:17:30
tags: Python
---

Oracle的Python客户端确实有点烦人，直接`pip install`是搞不定的，需要安装SDK和client。<!--more-->

首先去[这里](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)下载`instantclient-sdk-linux-x86-64` 和 `instantclient-basic-linux-x86-64`

然后要选择Oracle的目录，我一般都放在`/var/www/oracle`，在这个目录解压这两个压缩包

        unzip instantclient-sdk-linux-x86-64-11.2.0.2.0.zip
        unzip instantclient-basic-linux-x86-64-11.2.0.2.0.zip

解压后的文件都会存放在`instantclient_11_2`，进入这个目录

        ln -s libclntsh.so.11.1 libclntsh.so
        ln -s libocci.so.11.1 libocci.so

以上都是不同的版本的名字也不一样。Oracle不默认指定一个，也是醉了

然后需要指定Oracle的Home, `vim ~/.bashrc`，在末尾加上

        # oracle
        export ORACLE_HOME=/var/www/oracle/instantclient_11_2
        export LD_LIBRARY_PATH=$ORACLE_HOME
        export PATH=$ORACLE_HOME:$PATH

如果要立刻装，则需要`source ~/.bashrc`，然后就可以安装`cx_Oracle`了

        pip install cx_Oracle

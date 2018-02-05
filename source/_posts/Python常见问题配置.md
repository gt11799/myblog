title: Python常见问题配置
date: 2016-01-03 09:20:12
tags: Python
---

> 每次配置环境都会有问题，记下来备查

Python的配置有时候挺烦人的。我的环境是Mac，Python2.7，Python3.6，使用的包管理工具是Homebrew, pip, pipenv
<!--more-->

## 安装Mysql

比较坑的是，我们线上的Mysql是5.6，brew直接装的是5.7
我记得之前解决了这个问题，但是这次并没有解决掉。

```
brew install mysql
brew install mysql56
```

mysql-client的版本无所谓，但是mysql-server的版本一定得是5.6的。我之前把mysql@56直接给替换成了mysql，这次没有冲动的这么做。于是我这么启动mysql

```
brew services start mysql56
```

## 安装MongoDB

```
brew install mongodb
mkdir -p /data/db
sudo chown <yourname> /data/db
mongod --fork --logpath=/tmp/mongod.log
```

## pip安装的命令找不到的问题

跟之前用过conda，或者改过Path有关系。表现为，sudo pip装的东西，在shell中不起作用。

我先后做了几件事

- 重装Python

```
brew install python
brew unlink python
brew info python
ls /usr/local/Cellar/python
brew switch python 2.7.14_2
rm '/usr/local/bin/2to3-2'
brew switch python 2.7.14_2
brew link --overwrite python
```

- 编辑.zshrc，增加Path，感觉是这个生效的

```
export PATH=/usr/bin/:$PATH
```

然后之前装的包都要重装一次

## 安装MySQL-python

网上众说纷纭，首先

```
xcode-select --install
```

这个基本是必须装的了。其他的几个办法中，最好用的是

```
brew install openssl
LDFLAGS=-L/usr/local/opt/openssl/lib pip install MySQL-python
```

## 安装pycrypto

一直报错，基本的情况如下

```
/usr/include/stdlib.h:342:6: note: insert '_Nonnull' if the pointer should never be null
void    *valloc(size_t) __alloc_size(1);
       ^
         _Nonnull
src/_fastmath.c:36:11: fatal error: 'gmp.h' file not found
# include <gmp.h>
         ^~~~~~~
337 warnings and 1 error generated.
error: command 'clang' failed with exit status 1
```

解决方案找了好久，最好用的如下。

```
brew install gmp
pipenv shell # switch to virtualenv
env "CFLAGS=-I/usr/local/include -L/usr/local/lib" pip install pycrypto
```

## 安装pymssql

我一直怀疑是我们的版本不好，报错如下

```
creating build
creating build/temp.macosx-10.13-x86_64-2.7
clang -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -Qunused-arguments -Qunused-arguments -I/usr/local/include -I/usr/local/Cellar/python/2.7.14_2/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c _mssql.c -o build/temp.macosx-10.13-x86_64-2.7/_mssql.o -DMSDBLIB
_mssql.c:266:10: fatal error: 'sqlfront.h' file not found
#include "sqlfront.h"
         ^~~~~~~~~~~~
1 error generated.
error: command 'clang' failed with exit status 1

----------------------------------------
Failed building wheel for pymssql
Running setup.py clean for pymssql
Failed to build pymssql
Installing collected packages: pymssql
Running setup.py install for pymssql ... error
  Complete output from command /Users/bruce/.local/share/virtualenvs/falcon-r6zJiT-7/bin/python2.7 -u -c "import setuptools, tokenize;__file__='/private/var/folders/fd/cmrw1qt528xb5mfdvl29s9hc0000gp/T/pip-build-UQpepn/pymssql/setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record /var/folders/fd/cmrw1qt528xb5mfdvl29s9hc0000gp/T/pip-tKVp62-record/install-record.txt --single-version-externally-managed --compile --install-headers /Users/bruce/.local/share/virtualenvs/falcon-r6zJiT-7/bin/../include/site/python2.7/pymssql:
  Traceback (most recent call last):
    File "<string>", line 1, in <module>
    File "/private/var/folders/fd/cmrw1qt528xb5mfdvl29s9hc0000gp/T/pip-build-UQpepn/pymssql/setup.py", line 88, in <module>
      from Cython.Distutils import build_ext as _build_ext
  ImportError: No module named Cython.Distutils
```

解决方案

```
brew install freetds
```

错误变成了

```
_mssql.c:18814:15: error: use of undeclared identifier 'DBVERSION_80'
    __pyx_r = DBVERSION_80;
              ^
1 error generated.
error: command 'clang' failed with exit status 1
```

搜了半天，是freetds的版本问题，于是来了一套组合拳。其中`uninstall`与`link --force`都是必须的

```
brew uninstall --force freetds
brew install freetds@0.91
brew link --force freetds@0.91
pip install pymssql
```

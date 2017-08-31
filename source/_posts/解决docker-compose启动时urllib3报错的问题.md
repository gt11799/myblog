title: 解决docker-compose启动时urllib3报错的问题
date: 2017-08-31 20:34:31
tags: docker
---

使用pip安装了docker-compose之后，启动时报了如下如下错误<!--more-->

```
Traceback (most recent call last):
  File "/usr/local/bin/docker-compose", line 7, in <module>
    from compose.cli.main import main
  File "/usr/local/lib/python2.7/dist-packages/compose/cli/main.py", line 17, in <module>
    from . import errors
  File "/usr/local/lib/python2.7/dist-packages/compose/cli/errors.py", line 11, in <module>
    from docker.errors import APIError
  File "/usr/local/lib/python2.7/dist-packages/docker/__init__.py", line 2, in <module>
    from .api import APIClient
  File "/usr/local/lib/python2.7/dist-packages/docker/api/__init__.py", line 2, in <module>
    from .client import APIClient
  File "/usr/local/lib/python2.7/dist-packages/docker/api/client.py", line 11, in <module>
    from .build import BuildApiMixin
  File "/usr/local/lib/python2.7/dist-packages/docker/api/build.py", line 9, in <module>
    from .. import utils
  File "/usr/local/lib/python2.7/dist-packages/docker/utils/__init__.py", line 2, in <module>
    from .build import tar, exclude_paths
  File "/usr/local/lib/python2.7/dist-packages/docker/utils/build.py", line 5, in <module>
    from .utils import create_archive
  File "/usr/local/lib/python2.7/dist-packages/docker/utils/utils.py", line 18, in <module>
    from .. import tls
  File "/usr/local/lib/python2.7/dist-packages/docker/tls.py", line 5, in <module>
    from .transport import SSLAdapter
  File "/usr/local/lib/python2.7/dist-packages/docker/transport/__init__.py", line 3, in <module>
    from .ssladapter import SSLAdapter
  File "/usr/local/lib/python2.7/dist-packages/docker/transport/ssladapter.py", line 22, in <module>
    urllib3.connection.match_hostname = match_hostname
AttributeError: 'module' object has no attribute 'connection'
```

在谷歌上查了一下，发现[GitHub](https://github.com/docker/docker-py/issues/1054)上有人报过类似issue，结果发现要升级urllib3的版本，还要需改一下path

```
pip install urllib3==1.14
```

and

```
export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:/usr/lib/python2.7/dist-packages
```

另外，如果是系统初装，urllib3会报insecure的错误

```
apt-get install python-openssl
```

完美解决

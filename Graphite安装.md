---
title: Graphite 安装
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - monitor
tags:
  - graphite
---
> 参考：
> http://www.jsxubar.info/category/system-administration/graphite/
> http://www.jsxubar.info/category/system-administration/page/3/

> 更多集成 grafana 及 Diamond 请看：
> http://dongweiming.github.io/blog/archives/shi-yong-grafanahe-diamondgou-jian-graphitejian-kong-xi-tong/

# 使用 yum 安装环境
    $ yum install bitmap bitmap-fonts Django pycairo python-devel python-ldap python-memcached mod_wsgi python-sqlite2 glibc-devel gcc gcc-c++ git openssl-devel python-zope-interface httpd memcached python-hashlib  django-tagging python-twisted python-simplejson httpd mod_wsgi

    # 其它的使得 pip 安装就可以
    $ pip install whisper
    $ pip install carbon
    $ pip install graphite-web

# 安装并升级为最新的 zope.interface 及 twisted
    # ( 确保这个版本要到 3.6.0 以上，如果不行，下载安装 ) 及 twisted
    $ wget https://pypi.python.org/simple/zope.interface/zope.interface-4.1.2-py2.6-win-amd64.egg
    $ easy_install zope.interface-4.1.2-py2.6-win-amd64.egg

    # test twisted
    [root@bigdata twisted]# python
    Python 2.6.6 (r266:84292, Jan 22 2014, 09:42:36)
    [GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> from twisted.python.compat import _PY3
    >>>

# Check for missing dependencies
    $ python /tmp/pip-build-me/graphite-web/check-dependencies.py

# 设置权限
    $ chown -R apache:apache /opt/graphite/storage/

# 配置文件
    $ cp /opt/graphite/webapp/graphite/local_settings.py.example /opt/graphite/webapp/graphite/local_settings.py
    $ cp /opt/graphite/conf/carbon.conf.example /opt/graphite/conf/carbon.conf
    $ cp /opt/graphite/conf/storage-schemas.conf.example /opt/graphite/conf/storage-schemas.conf

# 同步数据库
    # 如果需要用其它数据库，请修改 /opt/graphite/webapp/graphite/local_settings.py
    $ python /opt/graphite/webapp/graphite/manage.py syncdb

    $ cd /opt/graphite/conf/
    $ cp graphite.wsgi.example graphite.wsgi
    $ cp storage-schemas.conf.example storage-schemas.conf
    $ cp carbon.conf.example carbon.conf

    $ cd /opt/graphite/webapp/graphite
    $ cp local_settings.py.example local_settings.py

# 在 httpd 中启用 virtualHost
    $ cd ../../
    $ cp examples/example-graphite-vhost.conf /etc/httpd/conf.d/vhost-graphite.conf

    # 并修改
    $ vi /etc/httpd/conf.d/vhost-graphite.conf
    WSGISocketPrefix /var/run/httpd/wsgi

# 修改已知的 bug
    $ vi /opt/graphite/webapp/graphite/storage.py

    def fetch(self, startTime, endTime, now=None):
        return whisper.fetch(self.fs_path, startTime, endTime)   # 去掉最后一个 now

# 替换 graphite 的界面
    graph-index-master
    graphite-web-master


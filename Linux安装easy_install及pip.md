---
title: linux 安装 easy_install 及 pip
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:16:45
updated:
tags:
  - easy_install
  - pip
categories:
  - linux
---

> 需要先将 python2.6 升级至 2.7，请参见： [python2.6 升级至 python2.7][1]


# 安装 easy_install
    # 获取软件包
    $ wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-3.6.tar.gz
    # 解压 :
    $ tar -xvf setuptools-3.6.tar.gz && cd setuptools-3.6
    # 使用 Python 2.7.x 安装 setuptools
    /usr/local/bin/python2.7 setup.py install

    更多 easy_install 用法见： easy_install 用法

# 安装 PIP
    # 下载方法 1
    $ curl https://bootstrap.pypa.io/get-pip.py | python -
    # 下载方法 2
    $ wget https://bootstrap.pypa.io/get-pip.pyy -O - | python

    # 安装方法 1
    $ yum install python-pip

    # 安装方法 2
    $ /usr/local/bin/easy_install pip  ( 建议 , 如果升级使用此方法！ )

    # 升级 pip
    $ /usr/local/bin/easy_install --upgrade pip

    # 升级 distribute(distribute 是 setuptools 的替代方案，pip 是 easy_install 的替代方案 )
    $ pip install -U distribute

    # 升级 ez_setup
    $ curl https://bootstrap.pypa.io/ez_setup.py | python -

    > PIP 使用指定源，且不缓存到目录
    $ pip install --no-cache-dir -r requirements.txt -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com

    > 安装墙内 PIP
    http://realfavicongenerator.net/

# easy_install 用法
    # 1 直接安装：
      easy_install SQLObject
    # 2 使用下载页面地址安装
      easy_install -f http://pythonpaste.org/package_index.html SQLObject
    # 3 直接给出下载 url 安装
      easy_install http://example.com/path/to/MyPackage-1.2.3.tgz
    # 4 .egg 文件的安装
      easy_install /my_downloads/OtherPackage-3.2.1-py2.3.egg
    # 5 跟新安装过的包
      easy_install --upgrade PyProtocols
    # 6 安装已经下载和提取在当前目录下的包
      easy_install .
    # 7 安装特定特定版本的库
      easy_install "SomePackage==2.0"
    # 8 大于某个版本
      easy_install "SomePackage>2.0
    # 9 卸载库
    easy_install -m PackageName

  [1]: http://blog.dongcj.com/linux/python2.6%E5%8D%87%E7%BA%A7%E8%87%B3python2.7/
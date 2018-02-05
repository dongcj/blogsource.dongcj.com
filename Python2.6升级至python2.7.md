---
title: Python2.6 升级至 python2.7
author: dongcj <ntwk@163.com>
date: 2016/11/28 17:44:33
updated: 2017-08-09 09:03:50
tags:
  - python 升级
categories:
  - linux
---

# 升级 python
## install
    # 一定要先安装 zlib zlib-devel openssl openssl-devel，ncurses-devel，不然安装好后没有 zlib, 和 HTTPlib
    $ yum -y install zlib-devel openssl-devel ncurses-devel libxml2-devel

    $ yum -y install mysql mysql-devel

    $ yum -y install libxml2-devel  libxslt-devel

## 编译安装 python 新版
    $ wget http://www.python.org/ftp/python/2.7.13/Python-2.7.13.tgz

    # 加上 --enable-shared 解决 "Cannot build PL/Python because libpython is not a shared library"
    $ tar -xzf Python-2.7.13.tgz && cd Python-2.7.13 && ./configure  --enable-shared
    $ make all && make install && make clean && make distclean

    $ echo "/usr/local/lib" >>/etc/ld.so.conf
    $ ldconfig
    $ mv /usr/bin/python /usr/bin/python2.6.bak
    $ ln -s /usr/local/bin/python /usr/bin/python

    # copy bz2 module
    $ yes | cp -rLfap /usr/lib64/python2.*/lib-dynload/bz2.so /usr/local/lib/python2.*/
    $ python -V

# 安装 easy_install 和 pip
    # 参见：[《linux 安装 easy_install 及 pip》][1]

## 修改 yum 中的 python 版本为 **python2.6**
    $ vi /usr/bin/yum
    $ vi /usr/bin/yum-config-manager

## 安装一些基础软件
    $ pip install readline

    $ pip install MySQL-python pssh flask
    $ pip install lxml virtualenv virtualenvwrapper websockify

  [1]: http://blog.dongcj.com/linux/linux%E5%AE%89%E8%A3%85easy_install%E5%8F%8Apip/


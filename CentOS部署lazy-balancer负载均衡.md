---
title: Centos 部署 lazy-balancer 负载均衡
author: dongcj <ntwk@163.com>
date: 2016/08/28 17:44:33
updated:
categories:
  - linux
tags:
  - nginx
  - tengine
  - lazy-balancer
  - 负载均衡
---

# clone soft
    $ git clone https://github.com/v55448330/lazy-balancer.git /usr/local/lazy_balancer
    $ cd /usr/local/lazy_balancer

# install tengine
    $ git submodule update --init --recursive

    $ cd resource/nginx/tengine
    $ yum -y install pcre-devel

    $ ./configure --user=nginx --group=nginx --prefix=/etc/nginx --sbin-path=/usr/sbin --error-log-path=/var/log/nginx/error.log --conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx.pid
    $ make && make install

    # add nginx user
    $ adduser --system --no-create-home --shell /bin/false   nginx

# install supervisor
    $ yum -y install supervisor

    # add to supervisor config file
    $ echo "[include]" >>/etc/supervisord.conf
    $ echo "files = /etc/supervisor/conf.d/*.conf" >>/etc/supervisord.conf

    $ cp -rf /usr/local/lazy_balancer/service/* /etc/supervisor/
    $ vi /etc/supervisor/conf.d/supervisor_balancer.conf

# start service
    $ service supervisor restart








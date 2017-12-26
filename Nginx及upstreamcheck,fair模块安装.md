---
title: Nginx及upstreamcheck,fair模块安装
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - web
tags:
  - nginx
  - upstreamcheck
---

# download nginx & module

```bash
cd /tmp
# main software
wget http://nginx.org/download/nginx-1.11.5.tar.gz

# dependency(if necessary)
wget https://nchc.dl.sourceforge.net/project/pcre/pcre/8.41/pcre-8.41.tar.gz

# module nginx_upstream_check_module
git clone https://github.com/yaoweibin/nginx_upstream_check_module.git

# module nginx-upstream-fair
git clone https://github.com/gnosek/nginx-upstream-fair.git

```

# make & install

```bash
# install dependency(if necessary)
tar xzf pcre-8.41.tar.gz
cd pcre-8.41
./configure
make && make install
ldconfig

# install nginx
tar xzf nginx-1.11.5.tar.gz
cd nginx-1.11.5
./configure --user=nobody --group=nobody --prefix=/usr/local/nginx/ \
--with-http_stub_status_module \
--add-module=../nginx_upstream_check_module \
--add-module=../nginx-upstream-fair

make && make install
```

# nginx.conf & proxy.conf
```bash
vi /usr/local/nginx/conf/nginx.conf

user  nobody;
worker_processes  4;

# security
# server_tokens   off;
# proxy_hide_header   X-Powered-By;

error_log     /var/log/nginx/error.log  warn;
#error_log     /var/log/nginx/error.log  notice;
#error_log      /var/log/nginx/error.log  debug;

pid        /var/run/nginx.pid;

events {
    worker_connections  8192;
}

http {
    include       /usr/local/nginx/conf/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    keepalive_timeout  65;

    # 设定负载均衡的服务器列表
    upstream lb.xxx.com {
        server xxxx:8088;
        server xxxx:8088;

        # upstream check
        check interval=6000 rise=1 fall=3 timeout=4000;

        # fairly scheduler
        fair;


    }

    # fastcgi config
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    # gzip config
    gzip on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;



    server {
        listen 80;
        include proxy.conf;

        # check_status_module
        # usage: curl http://localhost/status?format=json
        # 好像无法生效
        location /status?format=json {
            check_status;
            access_log  off;
            allow 127.0.0.1;
            deny all;
       }

        # http_stub_status
        location /ng_status {
            stub_status on;
            access_log  off;
            allow 127.0.0.1;
            deny all;
        }

        # upstream
        location / {
            proxy_pass http://lb.xxx.com;
        }


        # static file expires
            location ~ .*\.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)($|/) {
            expires      30d;
        }

        # js & css expires
            location ~ .*\.(js|css)($|/) {
            expires      8h;
        }
    }

}

```


```nginx
vi /usr/local/nginx/conf/proxy.conf

proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size 10m;
client_body_buffer_size 128k;
proxy_connect_timeout 90;
proxy_send_timeout 90;
proxy_read_timeout 90;
proxy_buffer_size 4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;

```

## start & auto-start
```bash
# autostart
echo "/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf" >>/etc/rc.local
# make log dir
mkdir -p /var/log/nginx/
# start
/etc/rc.local
# stop
pkill nginx
# reload
nginx -s reload
```


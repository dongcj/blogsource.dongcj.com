---
title: Curl、resty、httpie、httpstat、wuzz 使用介绍
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - curl
  - resty
  - httpie
  - httpstat
  - wizz
---

# curl
## 下载文件
```bash
export DOCKERIZE_VERION=latest && \
export DOCKERIZE_ARCH=dockerize-linux-amd64
DOCKERIZE_DOWNLOADURL=$(curl -s https://api.github.com/repos/jwilder/dockerize/releases/${DOCKERIZE_VERION} | jq -r ".assets[] | select(.name | test(\"${DOCKERIZE_ARCH}\")) | .browser_download_url")
curl -sSOL http://$DOCKERIZE_DOWNLOADURL
# or 
curl -sSL ${DOCKERIZE_DOWNLOADURL} | tar zxvf - -C ${MODULE_HOME}   
```

## 详细信息
    curl -v www.sina.com
    curl --trace output.txt www.sina.com
    curl --trace-ascii output.txt www.sina.com
    curl  -s  --head  www.sina.com

## 上传文件
    curl --form upload=@localfilename --form press=OK [URL]
    curl -T file.txt

    curl --cookie "name=xxx" www.example.com
    curl --user name@DOMAIN:password http://example.com

    curl -fsSL ...

## 下载网页保存为文件名
    curl -o [文件名] www.sina.com

    curl -L www.sina.com

    curl --limit-rate 1000B -O http://www.gnu.org/software/gettext/manual/gettext.html

## 对于 https 的方法，添加 --insecure
    curl --insecure -LJO https://packages.gitlab.com/gitlab

    curl example.com/form.cgi?data=xxx

## 增加头信息 (-H)
    curl -i -X GET --header "Content-Type:application/json" http://example.com
    # 模拟服务地址（例如有些网站只能用域名访问的）
    curl – H Host:web-test.proxy-test.local http://192.168.1.2:8080

    POST：(-d)
    curl -i -X POST --data "data=xxx"           example.com/form.cgi
    curl -i -X POST --data-urlencode "date=April 1"     example.com/form.cgi

## Referer(-e)
    curl --referer http://www.example.com http://www.example.com

## User Agent 字段 (-A)
    curl --user-agent "[User Agent]" http://www.example.com

## 配置压缩 (-I means "Fetch the HTTP-header only")
    curl -I http://www.111cn.net/ -H Accept-Encoding:gzip,defaltefrom

## 超时设置
    --connect-timeout 3     # 3 秒连接时间
    -m | --max-time   5     # 连接 5 秒后自动断开

## 本地 socket
    curl --unix-socket /var/run/docker.sock http:/images/json
    or
    curl --unix-socket /var/run/docker.sock http://localhost/images/json

## curl examples
    # zabbix 使用 API 进行认证
    # curl -i -X POST -H 'Content-Type:application/json' -d '{"jsonrpc": "2.0","method":"user.authenticate","params":{"user":"admin","password":"passw0rd"},"auth": null,"id":0}' http://192.168.0.54/api_jsonrpc.php

    HTTP/1.1 200 OK
    Date: Mon, 12 Aug 2013 05:53:05 GMT
    Server: Apache/2.2.21 (Linux/SUSE)
    X-Powered-By: PHP/5.3.8
    Content-Length: 68
    Content-Type: application/json

    {"jsonrpc":"2.0","result":"c12f74265ea3cfb772b5e1d56957645b","id":0}

    # zabbix 利用 tokern 和 id 进行注册主机

    2. echo `cat /tmp/123.txt` --- 这样也可以 ...

    3. curl -i -X POST -H 'Content-Type: application/json' -d ' 格式化后的内容 ' http://192.168.0.54/api_jsonrpc.php

    -i|--include :  在输出中包含 HTTP 头 ( 如服务器名，日期，HTTP 版本等 )
    -s|--silent  :  静默模式
    -X|--request :  请求 HTTP 服务，默认为 GET

  ---

# resty 模拟请求

    # install resty
    curl -L http://github.com/micha/resty/raw/master/resty > resty
    # 导入变量
    . resty
    $ resty http://127.0.0.1:8080/data
      http://127.0.0.1:8080/data*

    $ GET /blogs.json
    [ {"id" : 1, "title" : "first post", "body" : "This is the first post"}, ... ]

    $ PUT /blogs/2.json '{"id" : 2, "title" : "updated post", "body" : "This is the new."}'
    {"id" : 2, "title" : "updated post", "body" : "This is the new."}

    $ DELETE /blogs/2

    $ POST /blogs.json '{"title" : "new post", "body" : "This is the new new."}'
    {"id" : 204, "title" : "new post", "body" : "This is the new new."}

  ---

# 使用 httpie
    # install
    $ apt-get install httpie

    # usage
    $http httpie.org
    $ http PUT example.org X-API-Token:123 name=John

    $ http -f POST example.org hello=World
    $ http -v example.org
    $ http -a USERNAME POST https://api.github.com/repos/jakubroztocil/httpie/issues/83/comments body='HTTPie is awesome! :heart:'

    $ http example.org < file.json
    $ http example.org/file > file

    $ http --download example.org/file

    $ http --session=logged-in -a username:password httpbin.org/get API-Key:123
    $ http --session=logged-in httpbin.org/headers
    $ http localhost:8000 Host:example.com
    $ http DELETE example.org/todos/7

  ---

# httpstat
![httpstat](http://i.imgur.com/xR2OHZ4.png)

  ---

# wuzz
![wuzz](http://i.imgur.com/fdWQco8.gif)


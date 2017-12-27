---
title: Yum,wget,apt-get,npm,gem,yast,git,goAgent 代理 .md
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:22:41
updated: 2016/08/11 15:22:42
categories:
- linux
tags:
  - yum
  - wget
  - apt-get
  - npm
  - gem
  - yast
  - git
  - goAgent
  - 代理
  - proxy
---


# yum 配置代理
---
    $ vi /etc/yum.conf
      proxy=http://mycache.mydomain.com:808
      用于 yum 连接的帐户细节
      proxy_username=yum-user
      proxy_password=qwerty


    # yum 出现以下错误：
    > UnicodeDecodeError: 'ascii' codec can't decode byte 0xd1 in position 72: ordinal not in range(128)

    # 解决方法：
    使用光盘中的 GPGKEY：
    $ rpm --import RPM-GPG-KEY-redhat-release


# wget 配置代理
---
    # 配置 .wgetrc
      use_proxy=on
      http_proxy=http://proxy.yoyodyne.com:18023/
      #ftp_proxy=http://proxy.yoyodyne.com:18023/

    # 或者：
    $ wget -Y on "http_proxy=http://192.168.88.57:808" "www.baidu.com"

    # 进行断点下载示例
    $ wget -c [-O master.zip] --no-check-certificate https://xxx.com/file.zip



# apt-get 配置代理
---
    # 方法 1：将以下代码写入 /etc/apt/apt_proxy.conf （位置自定，临时使用）
      Acquire {
       Retries "0 ″ ;
       HTTP {
       Proxy "http://192.168.0.246:808";
       };
       };

    $ apt-get install postfix -c /etc/apt/apt.proxy.conf


    # 方法 2：将以下代码写入 /etc/apt/apt.conf( 永久使用 )
      Acquire::http::proxy  "http:// 用户名 : 密码 @ 代理地址 : 端口 ";
      Acquire::https::proxy  "https:// 用户名 : 密码 @ 代理地址 : 端口 ";
      Acquire::ftp::proxy  "ftp:// 用户名 : 密码 @ 代理地址 : 端口 ";


# curl 配置代理
---

    $ curl -x 192.168.88.57:808 "www.baidu.com"

    # 或者
    $ vi /root/.bashrc
      export http_proxy = http://PROXYSERVER:18023




# git 配置代理
---

    # 如果是 git clone http:// 或 git clone https:// 的话直接把代理服务器加到环境变量就可以了：
    $ export http_proxy="http://username:password@squid.vpsee.com:3128/"
    $ export https_proxy="http://username:password@squid.vpsee.com:3128/"

    ----------------------------------------------------------------------

    # 这样 git 就会自动使用环境变量里的代理服务器了。http 方式正常，但是 https 方式 git 就会提示 CA 证书不受信任了，
    # 可以通过以下方式把 goagent 的 CA 加到系统信任列表里：
    1、$ sudo cp path/to/goagent/local/CA.crt /usr/share/ca-certificates/goagent.crt
    2、$ sudo chmod a+r /usr/share/ca-certificates/goagent.crt
    3、$ sudo dpkg-reconfigure ca-certificates

    # 最后一个命令会有一个图形界面，在里面勾选 goagent 的 CA 就可以了。

    ----------------------------------------------------------------------

    # 如果是 git clone git:// 的话麻烦一些（可能有的 git 源不提供 http/https 的方式 )
    # 下载一个 connect.c
    $ gcc -o connect connect.c
    $ cp connect /bin/
    $ echo "connect -H http://192.168.88.57:808 $@" >/root/git_proxy
    $ chmod a+x /root/git_proxy

    $ git config --global core.gitproxy /root/git_proxy

    # 然后就可以使用 git 了

----------------------------------------------------------------------


# yast 配置代理
---

    $ vi /etc/sysconfig/proxy

      PROXY_ENABLED="on"
      HTTP_PROXY="http://192.168.11.35:808"


# GoAgent 代理
---

    # 重建证书目录：
    （如果不建，则会报：certutil: could not authenticate to token NSS Certificate DB.: An I/O error occurred during security authorization.）
    $ modutil -changepw "NSS Certificate DB" -dbdir $HOME/.pki/nssdb


    # 查看证书
    $ certutil -d sql:$HOME/.pki/nssdb -L


    # 导入证书
    $ pk12util -d sql:$HOME/.pki/nssdb -i XXXX.pfx


    # GoAgent 证书导入
    $ certutil -d sql:$HOME/.pki/nssdb -A -t TC -n "GoAgent" -i /tmp/CA.crt


    # 删除证书
    $ certutil -d sql:$HOME/.pki/nssdb -D -t "C,," -n GoAgent


# npm 代理：
---

    # 设置 :
    $ npm config set proxy=http://proxy.mysite.com:8080
      或在 /root/.npmrc 中设置：
    $ proxy=http://192.168.109.1:808

    # 取消 :
    $ npm config delete proxy


# gem 代理：
---

    # 安装时加上 --http-proxy 参数
    $ gem install --http-proxy http://proxy.mysite.com:8080 sass

    # 取消 :
    # 安装时不加上 --http-proxy 参数
    $ gem install sass






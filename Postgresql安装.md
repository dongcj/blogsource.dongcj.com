---
title: Postgresql 安装
author: dongcj <ntwk@163.com>
date: 2016/06/13 09:25:58
updated: 2016/08/04 17:26:08
categories:
  - database
tags:
  - postgres
---

    $ yum -y install readline-devel zlib-devel make gcc gcc-c++ openldap openldap-devel \
      openssl openssl-devel  perl perl-devel perl-ExtUtils* python-devel tcl-devel pam-devel libxslt-devel
    $ yum -y install wget gcc systemtap systemtap-sdt-devel \
      sgml-common docbook stylesheets openjade  sgml-tools  xsltproc \
      libxml2 libxml2-devel bison flex libreadline6-devel

# 安装
```bash
# 配置用户、环境变量
$ useradd postgres

$ vim /etc/profile
  export PATH=$PATH:$HOME/bin:/usr/local/pgsql/bin

$ source  /etc/profile

$ wget https://ftp.postgresql.org/pub/source/v9.5.1/postgresql-9.5.1.tar.bz2
$ bzcat postgresql-9.5.1.tar.bz2 | tar xBpf -
$ cd postgresql-9.5.1
$ ./configure --with-perl --with-python --with-tcl  --with-openssl  --without-ldap \
  --with-libxml --with-libxslt --enable-thread-safety  --with-wal-blocksize=64  \
  --with-blocksize=32 --with-wal-segsize=64 -enable-dtrace  --with-pam

$ make && make install
```

# 初始化数据库
```bash
mkdir -p /pgdata
chown postgres /pgdata
su - postgres
/usr/local/pgsql/bin/initdb -D /pgdata/

    $ vim /pgdata/pg_hba.conf
      host    all             all             0.0.0.0/0               trust

    $ vim /pgdata/postgresql.conf
      listen_addresses = '*'
      max_connections = 1500
```

# 启动数据库 [快速]
    $ /usr/local/pgsql/bin/pg_ctl [-m fast] -D /pgdata/ -l ~/pg_logfile start

# 创建并导入库
```bash
su postgres
psql
postgres=# create database <DB_NAME>;
postgres=# \c <DB_NAME>;
\i <DB_FILE>.sql
```


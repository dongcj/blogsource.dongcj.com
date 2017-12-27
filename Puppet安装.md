---
title: Puppet 安装
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - puppet
  - devops
---
### /etc/hosts 可以 ping 通对方域名

### 安装 ntp 做时间同步

### 安装 puppet

    $ yum install ruby ruby-libs ruby-rdoc
    $ wget http://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm  ( 这个版本可以修改为最新的）
    $ yum update

    # 服务端安装 puppet-server
    $ yum install puppet-server

    # 服务端安装 puppet 图形界面（可选）
    $ yum install ruby-mysql mysql  mod_passenger puppet-dashboard

    # 客户端安装 puppet
    $ yum install puppet

### service puppetmaster start

    # 启动期间会生成 ca 证书，如果不能生成，可以使用以下命令手动生成：
    $ puppet master --verbose --no-daemonize --cert_name "Puppet CA: `hostname -f`"

    # 查看生成的证书信息
    $ openssl x509 -text -noout -in /var/lib/puppet/ssl/certs/ca.pem

### 服务端允许请求证书

    $ cat > /etc/puppet/autosign.conf <<EOF
    *.xxx.com
    EOF

    $ service puppetmaster restart
    $ puppet cert list --all	--> 查看下证书，此时是没有客户机的，只有自己

### 客户端请求证书

    在 /etc/puppet/puppet.conf 中添加 server = bigdata.xxxx.com 一行，为了方便命令都可以不用 --server bigdata.xxxx.com

    $ puppetd --server bigdata.xxxx.com --test
    或者
    $ puppet agent --no-daemonize --onetime --verbose --debug

    # 再到服务端查看一下证书
    $ puppet cert list --all	--> 查看下证书，此时生成了客户端的，开头为 + 号

### 安装 dashboard

    $ vi /usr/share/puppet-dashboard/config/database.yml
    production:
    database: puppet
    username: root
    password: passw0rd
    encoding: utf8
    adapter: mysql

    $ vi /usr/share/puppet-dashboard/config/environment.rb
    config.time_zone = 'Beijing'


### 初始化数据库
    $ cd /usr/share/puppet-dashboard/
    rake RAILS_ENV=production db:migrate


### 整合 Passenger 和 apache
    $ vi /etc/httpd/conf.d/passenger.conf

    LoadModule passenger_module modules/mod_passenger.so
    PassengerRoot /usr/lib/ruby/gems/1.8/gems/passenger-3.0.21
    PassengerRuby /usr/bin/ruby
    PassengerHighPerformance on
    PassengerMaxPoolSize 12
    PassengerPoolIdleTime 1500
    PassengerStatThrottleRate 120
    RailsAutoDetect On

    <VirtualHost *>
      ServerName bigdata.xxxx.com
      DocumentRoot "/usr/share/puppet-dashboard/public/"
      ErrorLog /var/log/httpd/puppet-dashboard_error.log
      LogLevel warn
      CustomLog /var/log/httpd/puppet-dashboard_access.log combined
    </VirtualHost>


### 让 Dashboard 使用 Reports
    $ vi /etc/puppet/puppet.conf
    [master]
    reports = store, http
    reporturl = http://bigdata.xxxx.com:80/reports/upload


### 重启 puppetmaster 服务
    $ /etc/init.d/puppetmaster restart


### 导入报告
    $ cd /usr/share/puppet-dashboard
    $ rake RAILS_ENV=production reports:import


### 执行导入的 reports
    $ rake jobs:work RAILS_ENV="production"


### 下载及文档地址
    http://code.google.com/p/puppet-manifest-share/downloads/list
    http://www.vpsee.com/2012/03/install-puppet-on-centos-6-2/
    http://dongxicheng.org/cluster-managemant/puppet/
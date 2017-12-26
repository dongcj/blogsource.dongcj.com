---
title: Puppet ��װ
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - puppet
  - devops
---
### /etc/hosts ���� ping ͨ�Է�����

### ��װ ntp ��ʱ��ͬ��

### ��װ puppet

    $ yum install ruby ruby-libs ruby-rdoc
    $ wget http://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm  ( ����汾�����޸�Ϊ���µģ�
    $ yum update

    # ����˰�װ puppet-server
    $ yum install puppet-server

    # ����˰�װ puppet ͼ�ν��棨��ѡ��
    $ yum install ruby-mysql mysql  mod_passenger puppet-dashboard

    # �ͻ��˰�װ puppet
    $ yum install puppet

### service puppetmaster start

    # �����ڼ������ ca ֤�飬����������ɣ�����ʹ�����������ֶ����ɣ�
    $ puppet master --verbose --no-daemonize --cert_name "Puppet CA: `hostname -f`"

    # �鿴���ɵ�֤����Ϣ
    $ openssl x509 -text -noout -in /var/lib/puppet/ssl/certs/ca.pem

### �������������֤��

    $ cat > /etc/puppet/autosign.conf <<EOF
    *.xxx.com
    EOF

    $ service puppetmaster restart
    $ puppet cert list --all	--> �鿴��֤�飬��ʱ��û�пͻ����ģ�ֻ���Լ�

### �ͻ�������֤��

    �� /etc/puppet/puppet.conf ����� server = bigdata.xxxx.com һ�У�Ϊ�˷���������Բ��� --server bigdata.xxxx.com

    $ puppetd --server bigdata.xxxx.com --test
    ����
    $ puppet agent --no-daemonize --onetime --verbose --debug

    # �ٵ�����˲鿴һ��֤��
    $ puppet cert list --all	--> �鿴��֤�飬��ʱ�����˿ͻ��˵ģ���ͷΪ + ��

### ��װ dashboard

    $ vi /usr/share/puppet-dashboard/config/database.yml
    production:
    database: puppet
    username: root
    password: passw0rd
    encoding: utf8
    adapter: mysql

    $ vi /usr/share/puppet-dashboard/config/environment.rb
    config.time_zone = 'Beijing'


### ��ʼ�����ݿ�
    $ cd /usr/share/puppet-dashboard/
    rake RAILS_ENV=production db:migrate


### ���� Passenger �� apache
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


### �� Dashboard ʹ�� Reports
    $ vi /etc/puppet/puppet.conf
    [master]
    reports = store, http
    reporturl = http://bigdata.xxxx.com:80/reports/upload


### ���� puppetmaster ����
    $ /etc/init.d/puppetmaster restart


### ���뱨��
    $ cd /usr/share/puppet-dashboard
    $ rake RAILS_ENV=production reports:import


### ִ�е���� reports
    $ rake jobs:work RAILS_ENV="production"


### ���ؼ��ĵ���ַ
    http://code.google.com/p/puppet-manifest-share/downloads/list
    http://www.vpsee.com/2012/03/install-puppet-on-centos-6-2/
    http://dongxicheng.org/cluster-managemant/puppet/
---
title: Gearman 安装
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - cluster
tags:
  - gearman
---
> 参考
    http://www.php-oa.com/2010/09/05/perl-gearman-distributed.html

    python gearman library home page
    http://samuelks.com/python-gearman/docs/

    Tim Yang：利用 Gearman 来实现远程监控与管理
    http://timyang.net/linux/gearman-monitor/



# 一、安装 grarmand 及 libgearman-devel
    $ yum install gearmand libgearman-devel  php-pecl-gearman  ( 直接安装这个 php-pecl-gearman 就不用第二步：安装 PHP 的 API 了 )

# 二、安装 PHP 的 API
    $ yum install php php-devel boost boost-devel libuuid libuuid-devel

    $ tar -xzf gearman-1.1.2.tgz   # 下载地址：https://pecl.php.net/package/gearman
    $ cd gearman-1.1.2
    $ phpize
    $ ./configure; make; make install		// 实际上是给 PHP 增加了个 gearman 的 extension: /usr/lib64/php/modules/gearman.so

    # 增加 php 扩展
    $ vi /etc/php.d/gearman.ini
    ; Enable gearman extension module
    extension=gearman.so

# 三、启动 gearmand
    # ( 如果使用默认参数，需要开启 IPV6)

    # 可以设置成服务启动：
    $ vi /etc/sysconfig/gearmand
        OPTIONS="-L 0.0.0.0 --verbose=DEBUG"	--> 根据自己需求设置 --verbose

    $ service gearmand start

    # 或者自己启动
    $ gearmand -d --http-port 8090 -L 0.0.0.0 -p 4730

    # 如果需要使用数据库
    gearmand -d --http-port=8080 -L 0.0.0.0 -p 4730 --mysql-user=root --mysql-password=YOUR_PASS --mysql-db gearman

    # 命令行 API 用法：
    $ gearman -w -f wc -- wc -l	// 开启一个 worker（-w）, 监听一个函数 wc(-f wc)， 函数的内容为 wc -l

    # 将 /etc/passwd 文件给函数 wc 进行处理
    $ gearman -f wc < /etc/passwd


    $ gearadmin --show-jobs
          --status
          --workers
          -- ...


# PHP API 用法：

## 1. Worker:
    <?php

    $worker = new GearmanWorker();
    $worker->addServer();
    $worker->addFunction("reverse", "reverse_fn");

    while (1) {
        print "Waiting for job...\n";
        $ret = $worker->work();
        if($worker->returnCode() != GEARMAN_SUCCESS) {
           break;
        }
    }

    function reverse_fn(GearmanJob $job) {
        $workload = $job->workload();
        echo "Received job: ". $job->handle(). "\n";
        echo "Workload: $workload\n";
        $result = strrev($workload);
        echo "Result: $result\n";
        return $result;
    }

    ?>


## 2. client:
    <?php

    $client = new GearmanClient();
    $client->addServer();

    echo "sending job\n";

    $result = $client->doNormal("reverse", "Hello World!");		--> 这里可以换成 doBackground 就可以后台

    if($result) {
        echo "Success: $result\n";
    } else {
        echo "Failed!";
    }

    ?>





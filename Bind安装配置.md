---
title: Bind 安装配置
author: dongcj <ntwk@163.com>
date: 2016/07/11 15:40:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - bind
  - dns
---

> 参考：
http://linux.vbird.org/linux_server/0350dns.php#server_settings
DDNS 的安装见：http://wenku.baidu.com/view/31f263d233d4b14e8524687e.html

# 安装
    $ rpm -ivh bind bind-chroot

# 配置

    1. 生成 rndc key
    $ rndc-confgen -r /dev/random >/etc/rndc.conf
    $ chown root:named /etc/rndc.conf

    2. 配置 named.conf
    $ vi /etc/named.conf
    options
    {
            // Put files that named is allowed to write in the data/ directory:
            directory               "/var/named";           // "Working" directory
            dump-file               "data/cache_dump.db";
            statistics-file         "data/named_stats.txt";
            memstatistics-file      "data/named_mem_stats.txt";
            allow-query             { any; };
            recursion               yes;
            listen-on port 53       { any; };
            allow-transfer          { none; }; // 不允许别人进行 zone 转移 , 如果有 slave DNS，则可以开启
    };

    acl intranet { 192.168.1.0/24; }; // 本地来源 IP
    acl internet { ! 192.168.1.0/24; any; }; // 外部来源 IP. 惊叹号表示反向选择

            match-clients { "intranet"; };  // 吻合的才使用底下的 zone
            zone "." IN {
                    type hint;
                    file "named.ca";
            };

            zone "xdol.vicp.net" IN {
                    type master;
                    file "named.xdol.vicp.net";
            };

            zone "1.168.192.in-addr.arpa" IN {
                    type master;
                    file "named.192.168.1";
            };
    };

    view "wan" {
            match-clients { "internet"; };  // 吻合的才使用底下的 zone
            zone "." IN {
                    type hint;
                    file "named.ca";
            };

            zone "xdol.vicp.net" IN {
                    type master;
                    file "named.xdol.vicp.net.inter";
            };
    };

    key "rndc-key" {			// 这里是 rndc 的密钥，需要修改为 /etc/rndc.conf 中一样的 secret
          algorithm hmac-md5;
          secret "wFLLOzcaq3T2CFNvbT3d7g==";
    };

    controls {
          inet 127.0.0.1 port 953
                  allow { 127.0.0.1; } keys { "rndc-key"; };
    };

    logging {                               // 防止外部服务器错误导致 log 记录
            category lame-servers { null; };
    };

# zone 配置
    1. 正向 zone 内网配置
    $ vi /var/named/named.xdol.vicp.net

    $TTL    600
    @                       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D );
    @                       IN      NS      master.xdol.vicp.net.   ;DNS 服务器名称
    master.xdol.vicp.net.   IN      A       192.168.1.109           ;DNS 服务器 IP
    @                       IN      MX      10 www.xdol.vicp.net.   ; 邮件服务器

    www.xdol.vicp.net.      IN      A       192.168.1.109		; 内部网卡的 IP
    ftp.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.
    ssh.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.

    client.xdol.vicp.net.   IN      A       192.168.1.246

    2. 正向 zone 外网配置
    $ vi /var/named/named.xdol.vicp.net.inter

    $TTL    600
    @                       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D );
    @                       IN      NS      master.xdol.vicp.net.   ;DNS 服务器名称
    master.xdol.vicp.net.   IN      A       11.11.11.11		;DNS 服务器 IP
    @                       IN      MX      10 www.xdol.vicp.net.   ; 邮件服务器

    www.xdol.vicp.net.      IN      A       11.11.11.11		; 外部网卡的 IP
    ftp.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.
    ssh.xdol.vicp.net       IN      CNAME   www.xdol.vicp.net.

    client.xdol.vicp.net.   IN      A       192.168.1.246

    3. 反向 zone 内、外网设置 ( 外网不需要反向 zone)
    $ vi /var/named/named.192.168.1

    $TTL    600
    @       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D )
    @       IN      NS      master.xdol.vicp.net.
    109     IN      PTR     master.xdol.vicp.net. ; 将原来的 A 改为 PTR 标志而已

    109     IN      PTR     www.xdol.vicp.net.
    246     IN      PTR     client.xdol.vicp.net.

# 启动服务
    service named start
    chkconfig named on
    

# 配置 DDNS( 可选 )
    1. DNS 服务端生成主机端的 key( 在当前目录下会生成一个公钥及一个私钥 )
    dnssec-keygen -r /dev/urandom -a HMAC-MD5 -b 512 -n HOST greatwall

    $ vi /etc/named.conf
    // 先在任意地方加入這個 Key 的相關密碼資訊！ ( 加而不是更新其它的 )
    key "greatwall" {
            algorithm hmac-md5;
            secret "xZmUo8ozG8f2OSg/cqH8Bqxk59Ho8....3s9IjUxpFB4Q==";  // 这里换成算出的公钥 cat 出来的内容
    };

    zone "centos.vbird" IN {
            type master;
            file "named.centos.vbird";
            update-policy {					// 这个 update-policy 及后面就是添加的
                    grant greatwall name greatwall.xdol.vicp.net. A;
            };
    };

    $ chmod g+w /var/named
    $ chown named /var/named/named.centos.vbird
    $ /etc/init.d/named restart
    $ setsebool -P named_write_master_zones=1		// selinux 的

    将 ddns 的 key 传给客户端，并在客户端 crontab 中添加以下自动运行的脚本

    #!/bin/bash
    #
    # Update your Dynamic IP by using BIND 9 's tools
    #
    ###############################################
    # History
    # 2004/10/27	VBird	First time release
    #
    ##############################################
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    export PATH

    # 0. keyin your parameters
    basedir="/usr/local/ddns"			# working directory
    keyfile="$basedir"/"Kgreatwall.+157+60932.key"	# your ddns' key (filename)
    ttl=600						# the ttl time ( 10 min. )
    outif="eth0"					# Your interface (connect to internet)
    hostname="greatwall.xdol.vicp.net"		# Your hostname
    servername="192.168.1.109"			# The update primary DNS server name (or IP)
    showmesg=no                                     # if yes then show messages

    # Get your new IP
    newip=`ifconfig "$outif" | grep 'inet addr' | \
        awk '{print $2}' | sed -e "s/addr\://"`
    checkip=`echo $newip | grep "^[0-9]"`
    if [ "$checkip" == "" ]; then
        echo "$0: The interface can't connect internet...."
        exit 1
    fi

    # check if the DNS is the same with your IP
    dnsip=`host $hostname | head -n 1 | awk '{print $4}'`
    if [ "$newip" == "$dnsip" ]; then
            if [ "$showmesg" == "yes" ]; then
                    echo "$0: The IP is the same with DNS, Don't change it."
            fi
            exit 0
    fi

    # create the temporal file
    tmpfile=$basedir/ns_auto_update.txt
    cd $basedir
    echo "server $servername" 			>  $tmpfile
    echo "update delete $hostname A " 		>> $tmpfile
    echo "update add    $hostname $ttl A $newip" 	>> $tmpfile
    echo "send" 					>> $tmpfile

    # send yo

# 附录： 名词解释
---

TLD: 	Top Level Domain
ccTLD: 	Country code TLD
TTL: Time to live

申請 DNS 領域查詢授權

NS 记录： NameServer
A 记录：  Address
MX 记录： Mail
CNAME       實際代表這個主機別名的主機名字

PTR：就是指向 (PoinTeR) 的縮寫，後面記錄的資料就是反解到主機名稱囉！

記錄 . 的 zone 的類型，就被我們稱為 hint 類型

在 domain 的部分，若可能的話，請盡量使用 FQDN，亦即是主機名稱結尾加上一個小數點

SOA 主要是與領域有關，所以前面當然要寫 ksu.edu.tw 這個領域名。而 SOA 後面共會接七個參數，這七個參數的意義依序是：
1.Master DNS 伺服器主機名稱：這個領域主要是哪部 DNS 作為 master 的意思。在本例中， dns1.ksu.edu.tw 為 ksu.edu.tw 這個領域的主要 DNS 伺服器囉；

2. 管理員的 email：那麼管理員的 email 為何？發生問題可以聯絡這個管理員。要注意的是， 由於 @ 在資料庫檔案中是有特別意義的，因此這裡就將 abuse@mail.ksu.edu.tw 改寫成 abuse.mail.ksu.edu.tw ，這樣看的懂了嗎？

Refresh >= Retry *2
Refresh + Retry < Expire
Expire >= Rrtry * 10
Expire >= 7Days


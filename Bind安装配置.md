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

    view "lan" {                    // lan 只是一个名字而已，代表的是内网
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

    ; 关于 192.168.1.109 这部主机的正解设定
    www.xdol.vicp.net.      IN      A       192.168.1.109		; 内部网卡的 IP
    ftp.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.
    ssh.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.

    ; 其它主机的正确设定
    client.xdol.vicp.net.   IN      A       192.168.1.246


    2. 正向 zone 外网配置
    $ vi /var/named/named.xdol.vicp.net.inter

    $TTL    600
    @                       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D );
    @                       IN      NS      master.xdol.vicp.net.   ;DNS 服务器名称
    master.xdol.vicp.net.   IN      A       11.11.11.11		;DNS 服务器 IP
    @                       IN      MX      10 www.xdol.vicp.net.   ; 邮件服务器

    ; 关于 192.168.1.109 这部主机的正解设定
    www.xdol.vicp.net.      IN      A       11.11.11.11		; 外部网卡的 IP
    ftp.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.
    ssh.xdol.vicp.net       IN      CNAME   www.xdol.vicp.net.

    ; 其它主机的正确设定
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


    2. 将公钥加入到配置文件中
    $ vi /etc/named.conf
    // 先在任意地方加入這個 Key 的相關密碼資訊！ ( 加而不是更新其它的 )
    key "greatwall" {
            algorithm hmac-md5;
            secret "xZmUo8ozG8f2OSg/cqH8Bqxk59Ho8....3s9IjUxpFB4Q==";  // 这里换成算出的公钥 cat 出来的内容
    };

    // 然後將你原本的 zone 加入底下這一段宣示
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



    3. 客户端更新
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

SOA：就是開始驗證 (Start of Authority) 的縮寫


PTR：就是指向 (PoinTeR) 的縮寫，後面記錄的資料就是反解到主機名稱囉！


記錄 . 的 zone 的類型，就被我們稱為 hint 類型

因為使用 DHCP 時，系統會主動的使用 DHCP 伺服器傳來的資料進行系統設定檔的修訂。因此，你必須告知系統，不要使用 DHCP 傳來的伺服器設定值。 此時，你得要在 /etc/sysconfig/network-scripts/ifcfg-eth0 等相關檔案內，增加一行：『PEERDNS=no』，然後重新啟動網路即可。


在 domain 的部分，若可能的話，請盡量使用 FQDN，亦即是主機名稱結尾加上一個小數點





SOA 主要是與領域有關，所以前面當然要寫 ksu.edu.tw 這個領域名。而 SOA 後面共會接七個參數，這七個參數的意義依序是：
1.Master DNS 伺服器主機名稱：這個領域主要是哪部 DNS 作為 master 的意思。在本例中， dns1.ksu.edu.tw 為 ksu.edu.tw 這個領域的主要 DNS 伺服器囉；


2. 管理員的 email：那麼管理員的 email 為何？發生問題可以聯絡這個管理員。要注意的是， 由於 @ 在資料庫檔案中是有特別意義的，因此這裡就將 abuse@mail.ksu.edu.tw 改寫成 abuse.mail.ksu.edu.tw ，這樣看的懂了嗎？


3. 序號 (Serial)：這個序號代表的是這個資料庫檔案的新舊，序號越大代表越新。 當 slave 要判斷是否主動下載新的資料庫時，就以序號是否比 slave 上的還要新來判斷，若是則下載，若不是則不下載。 所以當你修訂了資料庫內容時，記得要將這個數值放大才行！ 為了方便使用者記憶，通常序號都會使用日期格式『YYYYMMDDNU』來記憶，例如崑山科大的 2010080369 序號代表 2010/08/03 當天的第 69 次更新的感覺。不過，序號不可大於 2 的 32 次方，亦即必須小於 4294967296 才行喔。


4. 更新頻率 (Refresh)：那麼啥時 slave 會去向 master 要求資料更新的判斷？ 就是這個數值定義的。崑山科大的 DNS 設定每 1800 秒進行一次 slave 向 master 要求資料更新。那每次 slave 去更新時， 如果發現序號沒有比較大，那就不會下載資料庫檔案。


5. 失敗重新嘗試時間 (Retry)：如果因為某些因素，導致 slave 無法對 master 達成連線， 那麼在多久的時間內，slave 會嘗試重新連線到 master。在崑山科大的設定中，900 秒會重新嘗試一次。意思是說，每 1800 秒 slave 會主動向 master 連線，但如果該次連線沒有成功，那接下來嘗試連線的時間會變成 900 秒。若後來有成功，則又會恢復到 1800 秒才再一次連線。


6. 失效時間 (Expire)：如果一直失敗嘗試時間，持續連線到達這個設定值時限， 那麼 slave 將不再繼續嘗試連線，並且嘗試刪除這份下載的 zone file 資訊。崑山科大設定為 604800 秒。意思是說，當連線一直失敗，每 900 秒嘗試到達 604800 秒後，崑山科大的 slave 將不再更新，只能等待系統管理員的處理。


7. 快取時間 (Minumum TTL)：如果這個資料庫 zone file 中，每筆 RR 記錄都沒有寫到 TTL 快取時間的話，那麼就以這個 SOA 的設定值為主。

除了 Serial 不可以超過 2 的 32 次方之外，有沒有其它的限制啊針對這幾個數值？是有的，基本上就是這樣：
Refresh >= Retry *2
Refresh + Retry < Expire
Expire >= Rrtry * 10
Expire >= 7Days

一般來說，如果 DNS RR 資料變更情況頻繁的，那麼上述的相關數值可以訂定的小一些，如果 DNS RR 是很穩定的， 為了節省頻寬，則可以將 Refresh 設定的較大一些。


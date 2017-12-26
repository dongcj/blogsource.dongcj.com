---
title: Bind ��װ����
author: dongcj <ntwk@163.com>
date: 2016/07/11 15:40:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - bind
  - dns
---

> �ο���
http://linux.vbird.org/linux_server/0350dns.php#server_settings
DDNS �İ�װ����http://wenku.baidu.com/view/31f263d233d4b14e8524687e.html

# ��װ
    $ rpm -ivh bind bind-chroot


# ����

    1. ���� rndc key
    $ rndc-confgen -r /dev/random >/etc/rndc.conf
    $ chown root:named /etc/rndc.conf

    2. ���� named.conf
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
            allow-transfer          { none; }; // ��������˽��� zone ת�� , ����� slave DNS������Կ���
    };

    acl intranet { 192.168.1.0/24; }; // ������Դ IP
    acl internet { ! 192.168.1.0/24; any; }; // �ⲿ��Դ IP. ��̾�ű�ʾ����ѡ��

    view "lan" {                    // lan ֻ��һ�����ֶ��ѣ������������
            match-clients { "intranet"; };  // �ǺϵĲ�ʹ�õ��µ� zone
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
            match-clients { "internet"; };  // �ǺϵĲ�ʹ�õ��µ� zone
            zone "." IN {
                    type hint;
                    file "named.ca";
            };

            zone "xdol.vicp.net" IN {
                    type master;
                    file "named.xdol.vicp.net.inter";
            };
    };

    key "rndc-key" {			// ������ rndc ����Կ����Ҫ�޸�Ϊ /etc/rndc.conf ��һ���� secret
          algorithm hmac-md5;
          secret "wFLLOzcaq3T2CFNvbT3d7g==";
    };

    controls {
          inet 127.0.0.1 port 953
                  allow { 127.0.0.1; } keys { "rndc-key"; };
    };

    logging {                               // ��ֹ�ⲿ������������ log ��¼
            category lame-servers { null; };
    };



# zone ����
    1. ���� zone ��������
    $ vi /var/named/named.xdol.vicp.net

    $TTL    600
    @                       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D );
    @                       IN      NS      master.xdol.vicp.net.   ;DNS ����������
    master.xdol.vicp.net.   IN      A       192.168.1.109           ;DNS ������ IP
    @                       IN      MX      10 www.xdol.vicp.net.   ; �ʼ�������

    ; ���� 192.168.1.109 �ⲿ�����������趨
    www.xdol.vicp.net.      IN      A       192.168.1.109		; �ڲ������� IP
    ftp.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.
    ssh.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.

    ; ������������ȷ�趨
    client.xdol.vicp.net.   IN      A       192.168.1.246


    2. ���� zone ��������
    $ vi /var/named/named.xdol.vicp.net.inter

    $TTL    600
    @                       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D );
    @                       IN      NS      master.xdol.vicp.net.   ;DNS ����������
    master.xdol.vicp.net.   IN      A       11.11.11.11		;DNS ������ IP
    @                       IN      MX      10 www.xdol.vicp.net.   ; �ʼ�������

    ; ���� 192.168.1.109 �ⲿ�����������趨
    www.xdol.vicp.net.      IN      A       11.11.11.11		; �ⲿ������ IP
    ftp.xdol.vicp.net.      IN      CNAME   www.xdol.vicp.net.
    ssh.xdol.vicp.net       IN      CNAME   www.xdol.vicp.net.

    ; ������������ȷ�趨
    client.xdol.vicp.net.   IN      A       192.168.1.246

    3. ���� zone �ڡ��������� ( ��������Ҫ���� zone)
    $ vi /var/named/named.192.168.1

    $TTL    600
    @       IN      SOA     master.xdol.vicp.net.   dongchaojun.gmail.com. ( 2012040702 3H 15M 1W 1D )
    @       IN      NS      master.xdol.vicp.net.
    109     IN      PTR     master.xdol.vicp.net. ; ��ԭ���� A ��Ϊ PTR ��־����

    109     IN      PTR     www.xdol.vicp.net.
    246     IN      PTR     client.xdol.vicp.net.


# ��������
    service named start
    chkconfig named on
    

# ���� DDNS( ��ѡ )
    1. DNS ��������������˵� key( �ڵ�ǰĿ¼�»�����һ����Կ��һ��˽Կ )
    dnssec-keygen -r /dev/urandom -a HMAC-MD5 -b 512 -n HOST greatwall


    2. ����Կ���뵽�����ļ���
    $ vi /etc/named.conf
    // ��������ط������@�� Key �����P�ܴa�YӍ�� ( �Ӷ����Ǹ��������� )
    key "greatwall" {
            algorithm hmac-md5;
            secret "xZmUo8ozG8f2OSg/cqH8Bqxk59Ho8....3s9IjUxpFB4Q==";  // ���ﻻ������Ĺ�Կ cat ����������
    };

    // Ȼ�ጢ��ԭ���� zone ��������@һ����ʾ
    zone "centos.vbird" IN {
            type master;
            file "named.centos.vbird";
            update-policy {					// ��� update-policy �����������ӵ�
                    grant greatwall name greatwall.xdol.vicp.net. A;
            };
    };

    $ chmod g+w /var/named
    $ chown named /var/named/named.centos.vbird
    $ /etc/init.d/named restart
    $ setsebool -P named_write_master_zones=1		// selinux ��



    3. �ͻ��˸���
    �� ddns �� key �����ͻ��ˣ����ڿͻ��� crontab ����������Զ����еĽű�

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







# ��¼�� ���ʽ���
---

TLD: 	Top Level Domain
ccTLD: 	Country code TLD
TTL: Time to live

��Ո DNS �I���ԃ�ڙ�

NS ��¼�� NameServer
A ��¼��  Address
MX ��¼�� Mail
CNAME       ���H�����@�����C�e�������C����

SOA�������_ʼ��C (Start of Authority) �Ŀs��


PTR������ָ�� (PoinTeR) �Ŀs��������ӛ䛵��Y�Ͼ��Ƿ��⵽���C���Q�ӣ�


ӛ� . �� zone ����ͣ��ͱ��҂��Q�� hint ���

���ʹ�� DHCP �r��ϵ�y�����ӵ�ʹ�� DHCP �ŷ���������Y���M��ϵ�y�O���n����ӆ����ˣ����횸�֪ϵ�y����Ҫʹ�� DHCP ������ŷ����O��ֵ�� �˕r�����Ҫ�� /etc/sysconfig/network-scripts/ifcfg-eth0 �����P�n���ȣ�����һ�У���PEERDNS=no����Ȼ�������ӾW·���ɡ�


�� domain �Ĳ��֣������ܵ�Ԓ��Ո�M��ʹ�� FQDN���༴�����C���Q�Yβ����һ��С���c





SOA ��Ҫ���c�I�����P������ǰ�殔ȻҪ�� ksu.edu.tw �@���I�������� SOA ���湲�����߂��������@�߂����������x�����ǣ�
1.Master DNS �ŷ������C���Q���@���I����Ҫ���Ĳ� DNS ���� master ����˼���ڱ����У� dns1.ksu.edu.tw �� ksu.edu.tw �@���I�����Ҫ DNS �ŷ����ӣ�


2. ����T�� email�����N����T�� email ��Σ��l�����}�����j�@������T��Ҫע����ǣ� ��� @ ���Y�ώ�n���������؄e���x�ģ�����@�e�͌� abuse@mail.ksu.edu.tw �Č��� abuse.mail.ksu.edu.tw ���@�ӿ��Ķ��ˆ᣿


3. ��̖ (Serial)���@����̖��������@���Y�ώ�n�������f����̖Խ�����Խ�¡� �� slave Ҫ�Д��Ƿ��������d�µ��Y�ώ�r��������̖�Ƿ�� slave �ϵ�߀Ҫ���Д࣬���Ǆt���d�������Ǆt�����d�� ���Ԯ�����ӆ���Y�ώ���ݕr��ӛ��Ҫ���@����ֵ�Ŵ���У� ���˷���ʹ����ӛ����ͨ����̖����ʹ�����ڸ�ʽ��YYYYMMDDNU����ӛ�������獋ɽ�ƴ�� 2010080369 ��̖���� 2010/08/03 ����ĵ� 69 �θ��µĸ��X�����^����̖���ɴ�� 2 �� 32 �η����༴���С� 4294967296 ����ม�


4. �����l�� (Refresh)�����Nɶ�r slave ��ȥ�� master Ҫ���Y�ϸ��µ��Дࣿ �����@����ֵ���x�ġ���ɽ�ƴ�� DNS �O��ÿ 1800 ���M��һ�� slave �� master Ҫ���Y�ϸ��¡���ÿ�� slave ȥ���r�� ����l�F��̖�]�б��^���ǾͲ������d�Y�ώ�n����


5. ʧ�����Lԇ�r�g (Retry)��������ĳЩ���أ����� slave �o���� master �_���B���� ���N�ڶ�õĕr�g�ȣ�slave ���Lԇ�����B���� master���ڍ�ɽ�ƴ���O���У�900 ������Lԇһ�Ρ���˼���f��ÿ 1800 �� slave �������� master �B���������ԓ���B���]�гɹ����ǽ���Lԇ�B���ĕr�g��׃�� 900 �롣������гɹ����t�֕��֏͵� 1800 �����һ���B����


6. ʧЧ�r�g (Expire)�����һֱʧ���Lԇ�r�g�����m�B�����_�@���O��ֵ�r�ޣ� ���N slave �������^�m�Lԇ�B�����K�҇Lԇ�h���@�����d�� zone file �YӍ����ɽ�ƴ��O���� 604800 �롣��˼���f�����B��һֱʧ����ÿ 900 ��Lԇ���_ 604800 ���ᣬ��ɽ�ƴ�� slave �����ٸ��£�ֻ�ܵȴ�ϵ�y����T��̎��


7. ��ȡ�r�g (Minumum TTL)������@���Y�ώ� zone file �У�ÿ�P RR ӛ䛶��]�Ќ��� TTL ��ȡ�r�g��Ԓ�����N�����@�� SOA ���O��ֵ������

���� Serial �����Գ��^ 2 �� 32 �η�֮�⣬�Л]�����������ư�ᘌ��@�ׂ���ֵ�����еģ������Ͼ����@�ӣ�
Refresh >= Retry *2
Refresh + Retry < Expire
Expire >= Rrtry * 10
Expire >= 7Days

һ����f����� DNS RR �Y��׃����r�l���ģ����N���������P��ֵ����ӆ����СһЩ����� DNS RR �Ǻܷ����ģ� ���˹�ʡ�l�����t���Ԍ� Refresh �O�����^��һЩ��


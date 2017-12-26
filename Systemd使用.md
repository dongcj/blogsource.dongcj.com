---
title: Systemd,systemctl ����
author: dongcj <ntwk@163.com>
date: 2016/06/13 09:25:58
updated: 2016/08/04 17:26:08
categories:
  - linux
tags:
  - systemd
  - systemctl
---
# Systemd �� Systemctl ��������

## �������õķ������
    �鿴Ŀǰ״̬��      systemctl status <SERVICE_NAME>

    [ ���� | ֹͣ ]:    systemctl [ start | stop ] <SERVICE_NAME>

    ���Ʋ� chkconfig:    systemctl list-unit-files | grep <SERVICE_NAME>
                        systemctl is-enabled <SERVICE_NAME>

    [��]�Զ�������      systemctl [ disable | enable ] <SERVICE_NAME>

    # ϵͳ���� systemd �� /etc/systemd/system
    # �û����� systemd �� /usr/lib/systemd/system/ ��main��
    # ���м��� systemd �� /run/systemd/system

    # ��ɾ��������Ҫ��
    systemctl stop <SERVICE_NAME>
    systemctl disable <SERVICE_NAME>
    rm -rf /[etc|run]/systemd/system/<SERVICE_NAME>
    systemctl daemon-reload
    `systemctl reset-failed`   # ֻ�����������ſ��Կ���������ʧ


### 1. ���ȼ�����ϵͳ���Ƿ�װ�� systemd ��ȷ����ǰ��װ�İ汾

    [root@manage ~]# systemctl --version
    systemd 215
    +PAM +AUDIT +SELINUX +IMA +SYSVINIT +LIBCRYPTSETUP +GCRYPT +ACL +XZ -SECCOMP -APPARMOR
    �����к�����ر��������ǰ�װ�� 215 �汾�� systemd��

### 2. ��� systemd �� systemctl �Ķ������ļ��Ϳ��ļ��İ�װλ��

    [root@manage ~]# whereis systemd
    systemd: /usr/lib/systemd /etc/systemd /usr/share/systemd /usr/share/man/man1/systemd.1.gz
    [root@manage ~]# whereis systemctl
    systemctl: /usr/bin/systemctl /usr/share/man/man1/systemctl.1.gz

### 3. ��� systemd �Ƿ�����
    [root@manage ~]# ps -eaf | grep [s]ystemd
    root         1     0  0 16:27 ?        00:00:00 /usr/lib/systemd/systemd --switched-root --system --deserialize 23
    root       444     1  0 16:27 ?        00:00:00 /usr/lib/systemd/systemd-journald
    root       469     1  0 16:27 ?        00:00:00 /usr/lib/systemd/systemd-udevd
    root       555     1  0 16:27 ?        00:00:00 /usr/lib/systemd/systemd-logind
    dbus       556     1  0 16:27 ?        00:00:00 /bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
    ע�⣺systemd ����Ϊ�����̣�PID=1�����еġ����������-e�������� ps ��������У�ѡ�����н��̣���-a��ѡ����Ựǰ��������н��̣���ʹ�ã�-f���������������ʽ�б��� -eaf����

    Ҳ��ע�������к���ķ����ź�������ʣ�ಿ�֡������ű��ʽ�� grep ���ַ�����ʽ��һ���֡�

### 4. ���� systemd ��������

    [root@manage ~]# systemd-analyze
    Startup finished in 487ms (kernel) + 2.776s (initrd) + 20.229s (userspace) = 23.493s


### 5. ��������ʱ�������̻��ѵ�ʱ��

    [root@manage ~]# systemd-analyze blame
    8.565s mariadb.service
    7.991s webmin.service
    6.095s postfix.service
    4.311s httpd.service
    3.926s firewalld.service
    3.780s kdump.service
    3.238s tuned.service
    1.712s network.service
    1.394s lvm2-monitor.service
    1.126s systemd-logind.service
    ....
### 6. ��������ʱ�Ĺؼ���

    [root@manage ~]# systemd-analyze critical-chain
    The time after the unit is active or started is printed after the "@" character.
    The time the unit takes to start is printed after the "+" character.
    multi-user.target @20.222s
    ���� mariadb.service @11.657s +8.565s
      ���� network.target @11.168s
        ���� network.service @9.456s +1.712s
          ���� NetworkManager.service @8.858s +596ms
            ���� firewalld.service @4.931s +3.926s
              ���� basic.target @4.916s
                ���� sockets.target @4.916s
                  ���� dbus.socket @4.916s
                    ���� sysinit.target @4.905s
                      ���� systemd-update-utmp.service @4.864s +39ms
                        ���� auditd.service @4.563s +301ms
                          ���� systemd-tmpfiles-setup.service @4.485s +69ms
                            ���� rhel-import-state.service @4.342s +142ms
                              ���� local-fs.target @4.324s
                                ���� boot.mount @4.286s +31ms
                                  ���� systemd-fsck@dev-disk-by\x2duuid-79f594ad\x2da332\x2d4730\x2dbb5f\x2d85d19608096
                                    ���� dev-disk-by\x2duuid-79f594ad\x2da332\x2d4730\x2dbb5f\x2d85d196080964.device @4
    ��Ҫ��Systemctl ���ܷ���.service�������ص㣨.mount�����׽ӿڣ�.socket�����豸��.device����Ϊ��Ԫ��


### 7. �г����п��õ�Ԫ state =[ static | enabled | disabled | invalid ]
                    type  =[ service  | target  |  mount  | socket | slice | scope | timer | path ]

    [root@manage ~]# systemctl list-unit-files [ --type=TYPE ] [ --state=STATE ]
` ������������� chkconfig`

    UNIT FILE                                   STATE
    proc-sys-fs-binfmt_misc.automount           static
    dev-hugepages.mount                         static
    dev-mqueue.mount                            static
    proc-sys-fs-binfmt_misc.mount               static
    sys-fs-fuse-connections.mount               static
    sys-kernel-config.mount                     static
    sys-kernel-debug.mount                      static
    tmp.mount                                   disabled
    brandbot.path                               disabled
    .....



### 8. �г����������е�Ԫ

    [root@manage ~]# systemctl list-units
    UNIT                                        LOAD   ACTIVE SUB       DESCRIPTION
    proc-sys-fs-binfmt_misc.automount           loaded active waiting   Arbitrary Executable File Formats File Syste
    sys-devices-pc...0-1:0:0:0-block-sr0.device loaded active plugged   VBOX_CD-ROM
    sys-devices-pc...:00:03.0-net-enp0s3.device loaded active plugged   PRO/1000 MT Desktop Adapter
    sys-devices-pc...00:05.0-sound-card0.device loaded active plugged   82801AA AC'97 Audio Controller
    sys-devices-pc...:0:0-block-sda-sda1.device loaded active plugged   VBOX_HARDDISK
    sys-devices-pc...:0:0-block-sda-sda2.device loaded active plugged   LVM PV Qzyo3l-qYaL-uRUa-Cjuk-pljo-qKtX-VgBQ8


### 9. �г�����ʧ�ܵ�Ԫ

    [root@manage ~]# systemctl --failed
    UNIT          LOAD   ACTIVE SUB    DESCRIPTION
    kdump.service loaded failed failed Crash recovery kernel arming
    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.
    1 loaded units listed. Pass --all to see loaded but inactive units, too.
    To show all installed unit files use 'systemctl list-unit-files'.



### 10. ���ĳ����Ԫ���� cron.service���Ƿ�����

    [root@manage ~]# systemctl is-enabled crond.service
    enabled


### 11. ��μ������������ʱ���û���÷��񣨼�ϵͳ����ʱ�Զ���������

    [root@manage ~]# systemctl is-active httpd.service
    [root@manage ~]# systemctl enable    httpd.service
    [root@manage ~]# systemctl disable   httpd.service


### 12. ���ĳ����Ԫ������Ƿ�����

    [root@manage ~]# systemctl status firewalld.service
    firewalld.service - firewalld - dynamic firewall daemon
       Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled)
       Active: active (running) since Tue 2015-04-28 16:27:55 IST; 34min ago
     Main PID: 549 (firewalld)
       CGroup: /system.slice/firewalld.service
               ���� 549 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
    Apr 28 16:27:51 tecmint systemd[1]: Starting firewalld - dynamic firewall daemon...
    Apr 28 16:27:55 tecmint systemd[1]: Started firewalld - dynamic firewall daemon.
    ʹ�� Systemctl ���Ʋ��������



### 14. Linux �����������������ֹͣ�����ط����Լ��������� httpd.service��״̬

    [root@manage ~]# systemctl start httpd.service
    [root@manage ~]# systemctl restart httpd.service
    [root@manage ~]# systemctl stop httpd.service
    [root@manage ~]# systemctl reload httpd.service
    [root@manage ~]# systemctl status httpd.service
    [root@manage ~]# systemctl show -p LoadState vdsm.service

    httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled)
       Active: active (running) since Tue 2015-04-28 17:21:30 IST; 6s ago
      Process: 2876 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)
     Main PID: 2881 (httpd)
       Status: "Processing requests..."
       CGroup: /system.slice/httpd.service
               ���� 2881 /usr/sbin/httpd -DFOREGROUND
               ���� 2884 /usr/sbin/httpd -DFOREGROUND
               ���� 2885 /usr/sbin/httpd -DFOREGROUND
               ���� 2886 /usr/sbin/httpd -DFOREGROUND
               ���� 2887 /usr/sbin/httpd -DFOREGROUND
               ���� 2888 /usr/sbin/httpd -DFOREGROUND
    Apr 28 17:21:30 tecmint systemd[1]: Starting The Apache HTTP Server...
    Apr 28 17:21:30 tecmint httpd[2881]: AH00558: httpd: Could not reliably determine the server's fully q...ssage
    Apr 28 17:21:30 tecmint systemd[1]: Started The Apache HTTP Server.
    Hint: Some lines were ellipsized, use -l to show in full.
    ע�⣺������ʹ�� systemctl �� start��restart��stop �� reload ����ʱ�����ǲ�����ն˻�ȡ���κ�������ݣ�ֻ�� status ������Դ�ӡ�����



### 15. ������Σ�������������������ʾ������ httpd.service��

    # ע�⣺����һ����ʾ�������һ������
    [root@manage ~]# systemctl mask httpd.service
    ln -s '/dev/null' '/etc/systemd/system/httpd.service'

    # ɾ������
    [root@manage ~]# systemctl unmask httpd.service
    rm '/etc/systemd/system/httpd.service'


### 16. ʹ�� systemctl ����ɱ������

    [root@manage ~]# systemctl kill httpd
    [root@manage ~]# systemctl status httpd
    httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled)
       Active: failed (Result: exit-code) since Tue 2015-04-28 18:01:42 IST; 28min ago
     Main PID: 2881 (code=exited, status=0/SUCCESS)
       Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
    Apr 28 17:37:29 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:29 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:39 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:39 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:49 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:49 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:59 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 17:37:59 tecmint systemd[1]: httpd.service: Got notification message from PID 2881, but recepti...bled.
    Apr 28 18:01:42 tecmint systemd[1]: httpd.service: control process exited, code=exited status=226
    Apr 28 18:01:42 tecmint systemd[1]: Unit httpd.service entered failed state.
    Hint: Some lines were ellipsized, use -l to show in full.
    ʹ�� Systemctl ���Ʋ�������ص�

### 17. �г�����ϵͳ���ص�

    [root@manage ~]# systemctl list-unit-files --type=mount
    UNIT FILE                     STATE
    dev-hugepages.mount           static
    dev-mqueue.mount              static
    proc-sys-fs-binfmt_misc.mount static
    sys-fs-fuse-connections.mount static
    sys-kernel-config.mount       static
    sys-kernel-debug.mount        static
    tmp.mount                     disabled


### 18. ���ء�ж�ء����¹��ء�����ϵͳ���ص㲢���ϵͳ�й��ص�״̬

    [root@manage ~]# systemctl start tmp.mount
    [root@manage ~]# systemctl stop tmp.mount
    [root@manage ~]# systemctl restart tmp.mount
    [root@manage ~]# systemctl reload tmp.mount
    [root@manage ~]# systemctl status tmp.mount
    tmp.mount - Temporary Directory
       Loaded: loaded (/usr/lib/systemd/system/tmp.mount; disabled)
       Active: active (mounted) since Tue 2015-04-28 17:46:06 IST; 2min 48s ago
        Where: /tmp
         What: tmpfs
         Docs: man:hier(7)
    http://www.freedesktop.org/wiki/Software/systemd/APIFileSystems
      Process: 3908 ExecMount=/bin/mount tmpfs /tmp -t tmpfs -o mode=1777,strictatime (code=exited, status=0/SUCCESS)
    Apr 28 17:46:06 tecmint systemd[1]: Mounting Temporary Directory...
    Apr 28 17:46:06 tecmint systemd[1]: tmp.mount: Directory /tmp to mount over is not empty, mounting anyway.
    Apr 28 17:46:06 tecmint systemd[1]: Mounted Temporary Directory.



### 19. ������ʱ������û���ù��ص㣨ϵͳ����ʱ�Զ����أ�

    [root@manage ~]# systemctl is-active tmp.mount
    [root@manage ~]# systemctl enable tmp.mount
    [root@manage ~]# systemctl disable  tmp.mount



### 20. �� Linux �����Σ������������ã���ɼ����ص�

    [root@manage ~]# systemctl mask tmp.mount
    ln -s '/dev/null' '/etc/systemd/system/tmp.mount'
    [root@manage ~]# systemctl unmask tmp.mount
    rm '/etc/systemd/system/tmp.mount'
    ʹ�� Systemctl ���Ʋ������׽ӿ�

### 21. �г����п���ϵͳ�׽ӿ�

    [root@manage ~]# systemctl list-unit-files --type=socket
    UNIT FILE                    STATE
    dbus.socket                  static
    dm-event.socket              enabled
    lvm2-lvmetad.socket          enabled
    rsyncd.socket                disabled
    sshd.socket                  disabled
    syslog.socket                static
    systemd-initctl.socket       static
    systemd-journald.socket      static
    systemd-shutdownd.socket     static
    systemd-udevd-control.socket static
    systemd-udevd-kernel.socket  static
    11 unit files listed.



### 22. �� Linux ��������������ֹͣ�������׽ӿڲ������״̬

    [root@manage ~]# systemctl start cups.socket
    [root@manage ~]# systemctl restart cups.socket
    [root@manage ~]# systemctl stop cups.socket
    [root@manage ~]# systemctl reload cups.socket
    [root@manage ~]# systemctl status cups.socket
    cups.socket - CUPS Printing Service Sockets
       Loaded: loaded (/usr/lib/systemd/system/cups.socket; enabled)
       Active: active (listening) since Tue 2015-04-28 18:10:59 IST; 8s ago
       Listen: /var/run/cups/cups.sock (Stream)
    Apr 28 18:10:59 tecmint systemd[1]: Starting CUPS Printing Service Sockets.
    Apr 28 18:10:59 tecmint systemd[1]: Listening on CUPS Printing Service Sockets.


### 23. ������ʱ�����׽ӿڣ������û��������ϵͳ����ʱ��������

    [root@manage ~]# systemctl is-active cups.socket
    [root@manage ~]# systemctl enable cups.socket
    [root@manage ~]# systemctl disable cups.socket



# ����� CPU �����ʣ�����

### 25. ��ȡ��ǰĳ������� CPU ������ httpd��

    [root@manage ~]# systemctl show -p CPUShares httpd.service
    CPUShares=1024
    [root@manage ~]# ע�⣺���������Ĭ�� CPU ����ݶ� =1024����������� / ����ĳ�����̵� CPU ����ݶ

### 26. ��ĳ������httpd.service���� CPU ����ݶ�����Ϊ 2000 CPUShares/

    [root@manage ~]# systemctl set-property httpd.service CPUShares=2000
    [root@manage ~]# systemctl show -p CPUShares httpd.service
    CPUShares=2000
    [root@manage ~]# ע�⣺����Ϊĳ���������� CPUShares�����Զ�����һ���Է�����������Ŀ¼���� httpd.service�������������һ����Ϊ 90-CPUShares.conf ���ļ������ļ����� CPUShare ������Ϣ�������ͨ�����·�ʽ�鿴���ļ���

    [root@manage ~]# vi /etc/systemd/system/httpd.service.d/90-CPUShares.conf
    [Service]
    CPUShares=2000



### 27. ���ĳ���������������ϸ��

    [root@manage ~]# systemctl show httpd
    Id=httpd.service
    Names=httpd.service
    Requires=basic.target
    Wants=system.slice
    WantedBy=multi-user.target
    Conflicts=shutdown.target
    Before=shutdown.target multi-user.target
    After=network.target remote-fs.target nss-lookup.target systemd-journald.socket basic.target system.slice
    Description=The Apache HTTP Server
    LoadState=loaded
    ActiveState=active
    SubState=running
    FragmentPath=/usr/lib/systemd/system/httpd.service
    ....


### 28. ����ĳ������httpd���Ĺؼ���

    [root@manage ~]# systemd-analyze critical-chain httpd.service
    The time after the unit is active or started is printed after the "@" character.
    The time the unit takes to start is printed after the "+" character.
    httpd.service +142ms
    ���� network.target @11.168s
      ���� network.service @9.456s +1.712s
        ���� NetworkManager.service @8.858s +596ms
          ���� firewalld.service @4.931s +3.926s
            ���� basic.target @4.916s
              ���� sockets.target @4.916s
                ���� dbus.socket @4.916s
                  ���� sysinit.target @4.905s
                    ���� systemd-update-utmp.service @4.864s +39ms
                      ���� auditd.service @4.563s +301ms
                        ���� systemd-tmpfiles-setup.service @4.485s +69ms
                          ���� rhel-import-state.service @4.342s +142ms
                            ���� local-fs.target @4.324s
                              ���� boot.mount @4.286s +31ms
                                ���� systemd-fsck@dev-disk-by\x2duuid-79f594ad\x2da332\x2d4730\x2dbb5f\x2d85d196080964.service @4.092s +149ms
                                  ���� dev-disk-by\x2duuid-79f594ad\x2da332\x2d4730\x2dbb5f\x2d85d196080964.device @4.092s


### 29. ��ȡĳ������httpd�����������б�

    [root@manage ~]# systemctl list-dependencies httpd.service
    httpd.service
    ���� system.slice
    ���� basic.target
      ���� firewalld.service
      ���� microcode.service
      ���� rhel-autorelabel-mark.service
      ���� rhel-autorelabel.service
      ���� rhel-configure.service
      ���� rhel-dmesg.service
      ���� rhel-loadmodules.service
      ���� paths.target
      ���� slices.target
      �� ���� -.slice
      �� ���� system.slice
      ���� sockets.target
      �� ���� dbus.socket
    ....



### 30. ���ȼ��г�������

    [root@manage ~]# systemd-cgls
    ���� 1 /usr/lib/systemd/systemd --switched-root --system --deserialize 23
    ���� user.slice
    �� ���� user-0.slice
    ��   ���� session-1.scope
    ��     ���� 2498 sshd: root@pts/0
    ��     ���� 2500 -bash
    ��     ���� 4521 systemd-cgls
    ��     ���� 4522 systemd-cgls
    ���� system.slice
      ���� httpd.service
      �� ���� 4440 /usr/sbin/httpd -DFOREGROUND
      �� ���� 4442 /usr/sbin/httpd -DFOREGROUND
      �� ���� 4443 /usr/sbin/httpd -DFOREGROUND
      �� ���� 4444 /usr/sbin/httpd -DFOREGROUND
      �� ���� 4445 /usr/sbin/httpd -DFOREGROUND
      �� ���� 4446 /usr/sbin/httpd -DFOREGROUND
      ���� polkit.service
      �� ���� 721 /usr/lib/polkit-1/polkitd --no-debug
    ....


### 31. �� CPU���ڴ桢���������г�������

    [root@manage ~]# systemd-cgtop
    Path                                                              Tasks   %CPU   Memory  Input/s Output/s
    /                                                                    83    1.0   437.8M        -        -
    /system.slice                                                         -    0.1        -        -        -
    /system.slice/mariadb.service                                         2    0.1        -        -        -
    /system.slice/tuned.service                                           1    0.0        -        -        -
    /system.slice/httpd.service                                           6    0.0        -        -        -
    /system.slice/NetworkManager.service                                  1      -        -        -        -
    /system.slice/atop.service                                            1      -        -        -        -
    /system.slice/atopacct.service                                        1      -        -        -        -
    /system.slice/auditd.service                                          1      -        -        -        -
    /system.slice/crond.service                                           1      -        -        -        -
    /system.slice/dbus.service                                            1      -        -        -        -
    /system.slice/firewalld.service                                       1      -        -        -        -
    /system.slice/lvm2-lvmetad.service                                    1      -        -        -        -
    /system.slice/polkit.service                                          1      -        -        -        -
    /system.slice/postfix.service                                         3      -        -        -        -
    /system.slice/rsyslog.service                                         1      -        -        -        -
    /system.slice/system-getty.slice/getty@tty1.service                   1      -        -        -        -
    /system.slice/systemd-journald.service                                1      -        -        -        -
    /system.slice/systemd-logind.service                                  1      -        -        -        -
    /system.slice/systemd-udevd.service                                   1      -        -        -        -
    /system.slice/webmin.service                                          1      -        -        -        -
    /user.slice/user-0.slice/session-1.scope                              3      -        -        -        -
    ����ϵͳ���еȼ�

### 32. ����ϵͳ��Ԯģʽ

    [root@manage ~]# systemctl rescue
    Broadcast message from root@tecmint on pts/0 (Wed 2015-04-29 11:31:18 IST):
    The system is going down to rescue mode NOW!


### 33. �������ģʽ

    [root@manage ~]# systemctl emergency
    Welcome to emergency mode! After logging in, type "journalctl -xb" to view
    system logs, "systemctl reboot" to reboot, "systemctl default" to try again
    to boot into default mode.


### 34. �г���ǰʹ�õ����еȼ�

    [root@manage ~]# systemctl get-default
    multi-user.target


### 35. �������еȼ� 5����ͼ��ģʽ

    [root@manage ~]# systemctl isolate runlevel5.target
    ��
    [root@manage ~]# systemctl isolate graphical.target


### 36. �������еȼ� 3�������û�ģʽ�������У�

    [root@manage ~]# systemctl isolate runlevel3.target
    ��
    [root@manage ~]# systemctl isolate multiuser.target


### 37. ���ö��û�ģʽ��ͼ��ģʽΪĬ�����еȼ�

    [root@manage ~]# systemctl set-default runlevel3.target
    [root@manage ~]# systemctl set-default runlevel5.target


### 38. ������ֹͣ����������ϵͳ��ʹϵͳ������˯��

    [root@manage ~]# systemctl reboot
    [root@manage ~]# systemctl halt
    [root@manage ~]# systemctl suspend
    [root@manage ~]# systemctl hibernate
    [root@manage ~]# systemctl hybrid-sleep



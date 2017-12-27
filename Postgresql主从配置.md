---
title: Postgresql 主从配置
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - database
tags:
  - postgresql
  - pg recovery
---

# 安装好二台 postgresql
> 参见：http://blog.dongcj.com/database/Postgresql%E5%AE%89%E8%A3%85/
```bash
# 主服务器上
vi /etc/profile
  export PGDATA=/pgdata

source /etc/profile

# 从服务器上
vi /etc/profile
  export PGDATA=/pgdata_backup

source /etc/profile
```

# 配置主服务器 192.168.10.11
```bash
# 使用 root 帐户
mkdir /pgdata
chown -R postgres:postgres /pgdata/

# 配置一个账号进行主从同步
su - postgres
psql [ -p  54321 ]     # 如果修改过端口，需要指定端口
psql$ CREATE ROLE replica login replication encrypted password 'replica';

# 增加 replica 用户
cd /pgdata
vi pg_hba.conf
  host    replication     replica       192.168.10.12/32               md5

# 配置复制参数
vi postgresql.conf
  wal_level = hot_standby     # 主为 wal 的主机
  max_wal_senders = 32        # 最多几个流复制
  wal_keep_segments = 256     # 设置流复制保留最多的 log 数目，每个 log 16M
  wal_sender_timeout = 60s    # 流复制主机发送数据的超时时间
  max_connections = 100       # 注意：从库的 max_connections 必须要大于主库的

# 修改权限
chmod -R 700 /pgdata/

# 重启 pg
pg_ctl restart
```

# 配置从服务器 192.168.10.12
```bash
# 使用 root 帐户
mkdir /pgdata_backup
chown -R postgres:postgres /pgdata_backup/

su - postgres
pg_basebackup -F p --progress -D /pgdata_backup -h 192.168.10.11 -p 5432 -U replica --password
# password is: replica

cd /pgdata_backup

# 编辑配置
vi recovery.conf
# 如果找不到 recovery.conf，可以新建，只需要下面三行即可
recovery_target_timeline = 'latest'
standby_mode = on
primary_conninfo = 'host=192.168.10.11 port=5432 user=replica password=replica'

# 编辑配置
vi postgresql.conf
max_connections = 1000  # ( 要大于主库的 )
hot_standby = on        # 说明这台机器不仅仅是用于数据归档，也用于数据查询
max_standby_streaming_delay = 30s   # 多久向主报告一次从的状态，最长的间隔时间
wal_receiver_status_interval = 5s   # 间隔多久将状态信息发送给主
hot_standby_feedback = on           # 如果有错误的数据复制，是否向主进行反馈

chmod -R 700 /pgdata_backup/

# 启动从 pg
pg_ctl start
```

# 检测配置是否成功
    # 使用 ps -ef 在主从二台上检测进程状态
    # 在主上应该有 sender 进程，在从上有 receiver 进程

    # 在主上查看复制状态
    $ psql$ select * from pg_stat_replication;

    # 可以看到 sender 的进程信息

# 主从切换命令

```bash 
# node1 上模拟主库故障
[postgres@node1 ~]$ pg_ctl stop -m f

# node2 提升备库状态
[postgres@node2 ~]$ pg_ctl promote

# node2 更新备数据
[postgres@node2 ~]$ createdb pgbench
[postgres@node2 ~]$ pgbench -i -s 10 pgbench

# node1 将以前 master 恢复为 standby
pg_rewind -D /opt/pgsql/data/ --source-server='host=node2 user=postgres port=5432'

vi /opt/pgsql/data/recovery.conf
standby_mode = 'on'
primary_conninfo = 'host=node2 user=postgres port=5432'
recovery_target_timeline = 'latest'

# 启动 node1 上的数据库：
[postgres@node1 ~]$ pg_ctl start

# 将 node1 恢复为 master
[postgres@node2 ~]$ pg_ctl stop -m f
[postgres@node1 ~]$ pg_ctl promote
[postgres@node1 ~]$ pgbench -s 10 -T 60 pgbench

# 恢复 node2 为 standby：
[postgres@node2 ~]$ pg_rewind -D /opt/pgsql/data/ --source-server='host=node1 user=postgres port=5432'
[postgres@node2 ~]$ mv /opt/pgsql/data/recovery.done /opt/pgsql/data/recovery.conf

# 修改 node2 为 node1
vi /opt/pgsql/data/recovery.conf 

# 启动 node2 上的数据库：
[postgres@node2 ~]$ pg_ctl start
```


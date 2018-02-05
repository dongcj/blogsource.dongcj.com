---
title: Rancher 安装
author: dongcj <ntwk@163.com>
date: 2017/12/24 15:43:19
updated: 2017/12/24 15:44:39
categories:
  - docker
tags:
  - rancher
  - docker
  - caas
---

# 1. 准备步骤

## 1.1. 服务器时间同步
> 如果不配置时间同步，可能会导致调度服务有可能不正常, 建议使用 **UTC** 时间
```bash
# 直接删除 /etc/localtime 即可使用 UTC 时间
rm -rf /etc/localtime
```

## 1.2. 支持 BBR，请升级内核至 4.9 以上版本（弱网环境建议）
> 如果要支持 aufs，请安装 linux-image-extra 的包

## 1.3. 所有服务器设置外部 DNS
```bash
# stop and disable the local dns generator
systemctl stop systemd-resolved.service
systemctl disable systemd-resolved.service

systemctl stop resolvconf.service
systemctl disable resolvconf.service

systemctl stop NetworkManager.service
systemctl disable NetworkManager.service

# 修改为外部 DNS 服务器
vi /etc/network/interfaces
  dns-nameservers 114.114.114.114

# set the /etc/resolv.conf
unlink /etc/resolv.conf
vi /etc/resolv.conf
nameserver 114.114.114.114

# show the service status
systemctl status systemd-resolved.service
systemctl status resolvconf.service
systemctl status NetworkManager

systemctl restart docker

# If has dnsmasq service
# Comment out the dns=dnsmasq line
# vi /etc/NetworkManager/NetworkManager.conf
  # dns=dnsmasq

# restart network and docker service
#systemctl restart network-manager
```


## 1.4. 设置 hostname
    [root@rancher-server] echo "<HOSTNAME>" >/etc/hostname && hostname <HOSTNAME>
    [root@rancher-server] hostnamectl set-hostname svi1r01n08


## 1.5. install Docker
```bash
# 国内 DaoCloud 镜像安装
curl -sSL https://get.daocloud.io/docker | sh

# Rancher 官方安装脚本
git clone https://github.com/rancher/install-docker.git
cd ./install-docker && bash <LATEST__XX>.sh

# Docker 开机自启动
systemctl  enable docker.service
```

## 1.6. 配置 docker 加速器（用于国内加速）

```bash
vi /etc/docker/daemon.json
# 编辑完后重启 Docker 生效
```

```json
{
  "registry-mirrors": [
     "https://2lqq34jg.mirror.aliyuncs.com",
     "https://pee6w651.mirror.aliyuncs.com",
     "https://registry.docker-cn.com",
     "http://hub-mirror.c.163.com"
  ]
}
```


# 2. 安装 Rancher
## 2.1. 单机安装

```bash
# 在一台服务器上执行命令

# 下载 rancher/server 镜像
docker pull rancher/server

# 启动
docker run -d --restart=always \
-v /opt/svicloud/rancher/server_db:/var/lib/mysql \
--name rancher-server -p 8080:8080  rancher/server

# 然后，浏览器打开 http://<SERVER_IP>:8080 即可
```

# 3. 其它安装选项

## 3.1. HA 模式安装（可选）
1. 安装一个外部**主从模式**的 MySQL
2. 在主备两边执行

```bash
docker run -d --restart=unless-stopped -p 8080:8080 -p 9345:9345 --name rancher-server rancher/server \
  --db-host myhost.example.com \
  --db-port 3306 \
  --db-user username \
  --db-pass password \
  --db-name cattle \
  --advertise-address <IP_of_the_Node>

# The following are default values: 
db.cattle.database=mysql
db.cattle.username=cattle
db.cattle.password=cattle
db.cattle.mysql.host=localhost
db.cattle.mysql.port=3306
db.cattle.mysql.name=cattle
# All values above are the defaults except db.cattle.database. If the defaults are fine, then all you need to set is db.cattle.database=mysql.
```

## 3.2. 启动集群模式（可选）
```bash
# 手动启动主
docker run -d --restart=unless-stopped -p 8080:8080 -p 9345:9345 --name rancher-server rancher/server --db-host 192.168.1.174 --db-port 3306 --db-user cattle --db-pass cattle --db-name cattle --advertise-address 192.168.1.174

# 手动启动备
docker run -d --restart=unless-stopped -p 8080:8080 -p 9345:9345 --name rancher-server rancher/server --db-host 192.168.1.174 --db-port 3306 --db-user cattle --db-pass cattle --db-name cattle --advertise-address 192.168.1.184
```

## 3.3. 设置 Rancher SSL（可选）

```bash
mkdir -p /opt/svicloud/rancher/cert
cp cert.pem key.pem /opt/svicloud/rancher/cert

docker run -d \
--restart=always \
--name rancher-server-ssl \
--link rancher-server \
-p 80:80 -p 443:443 \
-e 'RANCHER_URL=<YOUR_DOMAIN>' \
-e 'RANCHER_CONTAINER_NAME=rancher-server' \
-e 'RANCHER_PORT=8080' \
-v /opt/svicloud/rancher/cert:/etc/nginx/external/ \
codedevote/nginx-ssl-proxy-rancher
```


## 3.4. registry v2 版本的查询所有镜像
> https://github.com/docker/distribution/blob/master/docs/spec/api.md#deleting-an-image

```bash
# 1. 先查出所有的镜像
curl https://192.168.1.111/v2/_catalog

# 2. 查出该镜像的 tag 列表
curl https://192.168.1.111/v2/dongcj/webserver/tags/list

# 3. 查看详细的镜像信息
curl https://192.168.1.111/v2/dongcj/webserver/manifests/v0.1

# 4、删除所给的镜像
https://github.com/burnettk/delete-docker-registry-image

# 或者使用 ui 进行操作
```



# 4. 自定义加入 rancher 网络
    在启动参数中加入 --label io.rancher.container.network=true，这样网络就会有 rancher 的网络 IP



# 5. 查询 DNS 的所有记录

```bash
# external 的 DNS 设置方法相同
# 进入 network-services-metadata-dns-X.
cat /etc/rancher-dns/answers.json
```

> 注意：在独立的容器中（自建的），DNS 只保留最后一个


# 6. Rancher LB `http` redirect to `https`

  - 按照如下模式，自定义请求头信息

![](https://i.imgur.com/TEg1m2a.png)

  - 在 `自定义 Haproxy.cfg` 中增加以下内容

```apache
frontend http-frontend
    bind *:80
    mode http
    redirect scheme https code 301 if !{ ssl_fc }

```

# 7. Rancher LB 使用 IP Hash
在 ` 自定义 Haproxy.cfg` 中增加以下内容 :
```
balance source
```












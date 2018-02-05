---
title: Docker 安装及命令
author: dongcj <ntwk@163.com>
date: 
updated: 
categories: 
  -
tags:  
  - 
---

> 为了支持 bbr、aufs，必须安装 extra, 详见：[Linux 升级 kernel 及 tcp_BBR](http://blog.dongcj.com/linux/Linux%E5%8D%87%E7%BA%A7kernel%E5%8F%8Atcp_BBR/)

# 安装 Docker

```bash
# clone 后直接脚本安装
git clone https://github.com/rancher/install-docker.git
```

    docker info | grep WARN
    
> WARNING: No swap limit support
> 解决方法见第 1 步，配置 `grub` 参数

> WARNING: bridge-nf-call-ip6tables is disabled
> 解决方法：

    vim /etc/sysctl.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        net.bridge.bridge-nf-call-arptables = 1

    sysctl -p

<!-- more -->

# CentOS 安装 Docker
> ( 本方法只适用于 `centos6` 及以下 )

    yum -y install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

    yum install docker-io

> ( 本方法只适用于 `CentOS7` 及以上 )

```bash
# remove existing docker if exist
yum remove docker \
    docker-common \
    container-selinux \
    docker-selinux \
    docker-engine \
    docker-engine-selinux

curl -sSL -O https://get.docker.com/builds/Linux/x86_64/docker-1.10.1 && chmod +x docker-1.10.1 && sudo mv docker-1.10.1 /usr/local/bin/docker
```

> 针对 "Error starting daemon: Devices cgroup isn't mounted" 的解决方法：<br>
在 grub.conf 中添加 "cgroup_enable=memory swapaccount=1"

```bash
# 配置源
yum install -y yum-utils

yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum-config-manager --enable docker-ce-edge

# 查找指定版本
yum list docker-ce --showduplicates

# 安装指定版本
yum install 17.03.1.ce-1.el7.centos
yum list installed | grep docker
```

# 配置 Docker 加速器（用于国内加速）
> 以下操作需重启 Docker 服务生效

vi /etc/docker/daemon.json

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

# 配置 Docker 代理（用于国内加速）

```bash
vi /etc/docker/daemon.json

http_proxy=<IP_ADDR>:<PORT> https_proxy=<IP_ADDR>:<PORT> dockerd
```

# 安装 docker-compose

```bash
# docker-compose VERSION:
# https://github.com/docker/compose/releases

# get the latest info
DOCKER_COMPOSE_ARCH=docker-compose-Linux
curl -s https://api.github.com/repos/docker/compose/releases/latest | \
    jq -r ".assets[] | select(.name | test(\"${DOCKER_COMPOSE_ARCH}\")) | .browser_download_url"

# download the latest version
dockerComposeVersion=1.18.0

curl -L https://github.com/docker/compose/releases/download/$dockerComposeVersion/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

# 安装 docker-machine

```bash
curl -L https://github.com/docker/machine/releases/download/v0.9.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine
chmod +x /tmp/docker-machine
cp /tmp/docker-machine /usr/local/bin/docker-machine

# 使用 docker-machine( 支持 virtualbox、阿里云等 )
# 所有的 Available driver plugins：
> https://github.com/docker/docker.github.io/blob/master/machine/AVAILABLE_DRIVER_PLUGINS.md

# create aws machine
docker-machine create --driver amazonec2 --amazonec2-access-key AKI******* --amazonec2-secret-key 8T93C*******  aws-sandbox

docker-machine create --driver none --url=tcp://192.168.1.112:2376 svi1r01n02
```

# Docker 常用配置
> 如果是 systemctl 启动的 docker, 需要在 /lib/systemd/system/docker.service 中修改

```bash
## 配置 Docker 启动参数：（在 /etc/default/docker 中）
DOCKER_OPTS="--storage-driver=aufs --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.1.112:2376"

other_args="--exec-driver=lxc \
    --selinux-enabled [-H tcp://0.0.0.0:2376] [-b=br0]"   # 如果用了 -H，连接时也要用 -H 指定 !!
```

```bash
# 前台启动
docker -d -b br0

# 加入默认选项以使 service 生效
other_args="--exec-driver=lxc --selinux-enabled -b=br0"     
```

> 映射为 container 内部盘，可以自定义使用 read、write、mknode 操作

    docker run --device=/dev/sdc:/dev/xvdc:[rwm] --device=/dev/sdd --device=/dev/zero:/dev/nulo -i -t ubuntu ls -l /dev/{xvdc,sdd,nulo}

## Restart policies
    docker run --restart=[always|no|on-failure|unless-stopped] redis

## 向容器中增加主机名与 IP 对应 (add /etc/hosts entry)
    docker run --add-host=docker:10.180.0.1 --rm -it debian

## 设置容器的 ulimit
    docker run --ulimit nofile=1024:1024 --rm debian sh -c "ulimit -n"

## Docker 重启 daemon 不重启 container
```bash
# 以下二种方法任一种都可以
  - 将 /etc/docker/daemon.json 中的 "live-restore" 设置为 true，然后 SIGHUP（kill -HUP PID）
  - sudo dockerd --live-restore
```

> 可以使用 --config-file 指定，默认位置为 /etc/docker/daemon.json
```json
{
    "authorization-plugins": [],
    "bridge": "",
    "cluster-advertise": "",
    "cluster-store": "",
    "debug": true,
    "default-ulimits": {},
    "disable-legacy-registry": false,
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "fixed-cidr": "",
    "graph": "",
    "group": "",
    "hosts": [],
    "insecure-registries": [],
    "labels": [],
    "live-restore": true,
    "log-driver": "",
    "log-level": "",
    "mtu": 0,
    "pidfile": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "storage-driver": "",
    "storage-opts": [],
    "swarm-default-advertise-addr": "",
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "tlsverify": true
}
```

# Docker 的 API 操作
```bash
# 镜像
curl --unix-socket /var/run/docker.sock http://localhost/images/json

# events
curl --no-buffer -XGET --unix-socket /var/run/docker.sock http://localhost/events

# container 信息
curl --unix-socket /var/run/docker.sock "http://localhost/containers/json?all=1&before=8dfafdbc3a40&size=1"
```

## set metadata on container

```bash
# ( 如下设置了一个 my-label="" 和 com.example.foo=bar)
docker run -l my-label --label com.example.foo=bar ubuntu bash
docker run --label-file ./labels ubuntu bash

# cat ./labels
com.example.label1="a label"

# this is a comment
com.example.label2=another\ label
com.example.label3
```

## 直接设置 sysctl( 不能与 --network=host 同用 )
    docker run --sysctl net.ipv4.ip_forward=1 ubuntu

## 从 container 的变化中新建一个 image
    docker commit $CONTAINER_ID [REPOSITORY[:TAG]]

# 常用 Docker 命令

    docker run --rm --volume-from dbdata -v ${pwd}:/backup  ubuntu tar cvf /backup/backup.tar /dbdata

## 已运行容器通过 iptbles 来 nat

    iptables -t nat -A  DOCKER -p tcp --dport 3306 -j DNAT --to-destination 172.17.0.2:3306

# Docker Inspect
```bash
# 如果镜像和实例名字重名，使用 --type 区分
docker inspect --type=image rhel7

container_ip=$(docker inspect --format '{{.NetworkSettings.IPAddress}}' ${container_id})

docker inspect `docker ps -q` | grep IPAddress | cut -d '"' -f 4
# 或者
docker inspect `dl` | jq -r '.[0].NetworkSettings.IPAddress'

# get the IP address of a container
docker inspect \
  --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  d2cc496561d6

172.17.0.2

docker inspect --format '{{json .Mounts}}' pensive_blackwell

docker inspect --format "{{lower .Name}}" pensive_blackwell

# Listing all port bindings
docker inspect \
  --format='{{range $p, $conf := .NetworkSettings.Ports}} \
  {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' \
  d2cc496561d6

  80/tcp -> 80

# Getting size information on a container
docker inspect -s d2cc496561d6 |　grep -i Size

# 找到 container 的 pid
PID=$(docker inspect --format {{.State.Pid}} <container_name_or_ID>)

docker inspect -f '{{.Config.Image}}' 5cf58382b2f0

# logging driver
docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

sut_ip=${BOOT_2_DOCKER_HOST_IP}

template='{{ range $key, $value := .NetworkSettings.Ports }}{{ $key }}='"${BOOT_2_DOCKER_HOST_IP}:"'{{ (index $value 0).HostPort }} {{ end }}'

tomcat_host_port=$(docker inspect --format="${template}" ${container_id})

for line in ${tomcat_host_port} ; do
    echo "${line}" >> ${work_dir}/docker_container_hosts.properties
done
```
> 详见：[https://docs.docker.com/engine/admin/logging/overview/](https://docs.docker.com/engine/admin/logging/overview/)


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

# 1. Ubuntu 安装 Docker

```bash
# clone 后直接脚本安装 , CentOS 没有测试过
git clone https://github.com/rancher/install-docker.git
cd ./install-docker
# install the latest docker release, eg: 
bash 17.12.sh
```

# 2. 配置 Docker 加速器（用于国内加速，可选）
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

<!-- more -->

# 3. 配置 Docker 代理（国内无法访问国外某些网站的 https，可选）

```
# 临时使用直接启动 dockerd（当然也可以写入 systemd 长期生效）
http_proxy=PROXY_ADDR:PORT https_proxy=PROXY_ADDR:PORT dockerd
```

# 4. 安装 docker-compose（可选）

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

# 5. 安装 docker-machine（可选）

```bash
curl -L https://github.com/docker/machine/releases/download/v0.9.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine
chmod +x /tmp/docker-machine
cp /tmp/docker-machine /usr/local/bin/docker-machine

# 使用 docker-machine( 支持 virtualbox、阿里云等 )
# 所有的 Available driver plugins：
> https://github.com/docker/docker.github.io/blob/master/machine/AVAILABLE_DRIVER_PLUGINS.md

# create aws machine
docker-machine create --driver amazonec2 --amazonec2-access-key AKI******* --amazonec2-secret-key 8T93C*******  aws-sandbox

# 直接使用本地服务器
docker-machine create --driver none --url=tcp://192.168.1.112:2376 svi1r01n02
```

# 6. Docker 常用配置
> 如果是 systemctl 启动的 docker, 需要在 /lib/systemd/system/docker.service 中修改

```bash
## 配置 Docker 启动参数：（在 /etc/default/docker 中）
DOCKER_OPTS="--storage-driver=aufs --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.1.112:2376"

other_args="--exec-driver=lxc \
    --selinux-enabled [-H tcp://0.0.0.0:2376] [-b=br0]"   # 如果用了 -H，连接时也要用 -H 指定 !!
```

## 6.1. Docker 使用不同的桥接方式

```bash
# 前台启动
docker -d -b br0

# 加入默认选项以使 service 生效
other_args="--exec-driver=lxc --selinux-enabled -b=br0"     
```

## 6.2. Docker 直接使用外部块存储
> 映射为 container 内部盘，可以自定义使用 read、write、mknode 操作

    docker run --device=/dev/sdc:/dev/xvdc:[rwm] --device=/dev/sdd --device=/dev/zero:/dev/nulo -i -t ubuntu ls -l /dev/{xvdc,sdd,nulo}

## 6.3. Restart policies
    docker run --restart=[always|no|on-failure|unless-stopped] redis

## 6.4. 向容器中增加主机名与 IP 对应 (add /etc/hosts entry)
    docker run --add-host=docker:10.180.0.1 --rm -it debian

## 6.5. 设置容器的 ulimit
    docker run --ulimit nofile=1024:1024 --rm debian sh -c "ulimit -n"

## 6.6. Docker 重启 daemon 不重启 container
```bash
# 以下二种方法任一种都可以
  - 将 /etc/docker/daemon.json 中的 "live-restore" 设置为 true，然后 SIGHUP（kill -HUP PID）
  - sudo dockerd --live-restore
```

## 6.7. Docker Daemon 的配置文件
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

# 7. Docker 的 API 操作
```bash
# 镜像
curl --unix-socket /var/run/docker.sock http://localhost/images/json

# events
curl --no-buffer -XGET --unix-socket /var/run/docker.sock http://localhost/events

# container 信息
curl --unix-socket /var/run/docker.sock "http://localhost/containers/json?all=1&before=8dfafdbc3a40&size=1"
```

# 8. Docker 小技巧
## 8.1. set metadata on container

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

## 8.2. 直接设置 sysctl( 不能与 --network=host 同用 )
    docker run --sysctl net.ipv4.ip_forward=1 ubuntu

## 8.3. 从 container 的变化中新建一个 image
    docker commit $CONTAINER_ID [REPOSITORY[:TAG]]

# 9. 常用 Docker 命令

## 9.1. 备份容器中的数据（将容器中的数据目录拷贝至当前目录下）
    docker run --rm --volume-from dbdata -v ${pwd}:/backup  ubuntu tar cvf /backup/backup.tar /dbdata

## 9.2. 已运行容器通过 iptbles 来 nat

    # 相当于端口映射
    iptables -t nat -A  DOCKER -p tcp --dport 3306 -j DNAT --to-destination 172.17.0.2:3306

# 10. Docker Inspect
```bash
# 如果镜像和实例名字重名，使用 --type 区分
docker inspect --type=image rhel7

# 获取正在运行的容器 IP 的示例
container_ip=$(docker inspect --format '{{.NetworkSettings.IPAddress}}' ${container_id})

# docker inspect 输出结果的解析
docker inspect `docker ps -q` | grep IPAddress | cut -d '"' -f 4
# 或者
docker inspect `dl` | jq -r '.[0].NetworkSettings.IPAddress'

# get the IP address of a container
docker inspect \
  --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  d2cc496561d6

172.17.0.2

# 以 json 的格式展示
docker inspect --format '{{json .Mounts}}' pensive_blackwell

# 大小写
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

# 找到 container 使用的镜像
docker inspect -f '{{.Config.Image}}' 5cf58382b2f0

# logging driver
docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

# 格式 host:port , 并且把他们输入一个 java properties 文件：
sut_ip=${BOOT_2_DOCKER_HOST_IP}

template='{{ range $key, $value := .NetworkSettings.Ports }}{{ $key }}='"${BOOT_2_DOCKER_HOST_IP}:"'{{ (index $value 0).HostPort }} {{ end }}'

tomcat_host_port=$(docker inspect --format="${template}" ${container_id})

for line in ${tomcat_host_port} ; do
    echo "${line}" >> ${work_dir}/docker_container_hosts.properties
done
```
> 详见：[https://docs.docker.com/engine/admin/logging/overview/](https://docs.docker.com/engine/admin/logging/overview/)

# 11. Docker 清理方案
## 11.1. 清理
```bash
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs --no-run-if-empty docker rm

docker rm $(docker ps -q -f status=exited)

docker images | grep "<none>" | awk '{print $3}' | xargs docker rmi

docker rmi $(docker images -q -f "dangling=true")
```

```
docker-cleanup-volumes
Manage data in containers
remove-orphan-images.sh
Deleting images from a private docker registry
delete-docker-registry-image（support v2）

Cleaning up unused Docker
How to remove old Docker containers
Implement a 'clean' command
docker-cleanup-volumes.sh
https://docs.docker.com/docker-trusted-registry/soft-garbage/
docker-cleanup
docker-gc
```
> https://github.com/yangtao309/yangtao309.github.com/issues/1


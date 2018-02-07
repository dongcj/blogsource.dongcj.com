---
title: Harbor 支持 https 
author: dongcj <ntwk@163.com>
date: 
updated: 
categories: 
  - 
tags: 
  - 
---

2. 新建仓库目录

```bash
mkdir -p /opt/svicloud/tools/harbor/data/cert/
```

```bash
echo -n "svicloud85509336" >/opt/svicloud/tools/harbor/data/cert/secretkey
```

5. 使用 docker-compose 安装 harbor

> docker-compose [下载地址](http://download.svicloud.com/harbor-docker-compose-v1.2.2.yml)

```bash
mv harbor-docker-compose-v1.2.2.yml docker-compose.yml 
docker-compose up -d
```

<!-- more -->

6. 测试下，应该可以成功，如无法成功，运行步骤 7

7. （` 可选 `）修改 `registry` 中的证书 `root.crt` 和 `ui` 的 `private_key.pem`

# 通过 curl 请求测试
```bash
curl -i  https://<HARBOR.DOMAIN.COM>/v2/_catalog

HTTP/1.1 401 Unauthorized
Server: nginx/1.11.5
Date: Fri, 19 May 2017 06:45:06 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 134
Connection: keep-alive
Docker-Distribution-Api-Version: registry/2.0
Www-Authenticate: Bearer realm="https://<HARBOR.DOMAIN.COM>/service/token",service="token-service",scope="registry:catalog:*"

## 会收到一条没有认证的消息，这个正常，返回一定是 401
{"errors":[{"code":"UNAUTHORIZED","message":"authentication required","detail":[{"Type":"registry","Name":"catalog","Action":"*"}]}]}

/service/token?account=test&scope=repository:test/repo:push,pull&service=token-service

```

# 附：jwt 的 claim 格式
```bash
claim = {
    "iss": self.issuer,
    "sub": self.account,
    "aud": self.service,
    "exp": now + self.token_expires,
    "nbf": now,
    "iat": now,
    "jti": base64.b64encode(os.urandom(1024)), # TODO
    "access": self.access
}

# iss 前期通过 registry.yaml 配置文件里的 issuser
# sub 当前操作的帐号或者说用户名
# aud 前期通过 registry.yaml 配置文件里的 service
# exp 、 iat 和 nbf 表示的是 token 时间相关的
# jti 是一段随机字符串
# access 就是上述解析过后的 scope
```

  - 删除 harbor 容器

```bash
docker rm -f -v `docker ps -a | grep harbor | awk '{print $1}'`
```

  - 清理安装目录

```bash
rm -rf /opt/svicloud/tools/harbor
```
  - 清理卷

```bash
docker volume prune
```
  - 清理镜像

```bash
docker rmi dongcj/harbor-setupwrapper:v1.2.2
```


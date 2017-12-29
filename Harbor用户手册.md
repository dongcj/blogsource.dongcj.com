---
title: Harbor 用户手册
author: dongcj <ntwk@163.com>
---

# docker-compose 或自建商店安装
> 如果之前有安装失败记录，需要进行清理，见『清理及故障处理』章节

1. 从阿里云或第三方证书提供商下载免费证书。

2. 新建仓库目录

```bash
mkdir -p /opt/svicloud/tools/harbor/data/cert/
```

3. 将证书的 `server.crt` 和 `server.key` 放置于上述目录


4. 生成一个 `secretkey` 文件，也放置于以上目录

```bash
echo -n "svicloud85509336" >/opt/svicloud/tools/harbor/data/cert/secretkey
```


5. 使用 docker-compose 安装 harbor

> docker-compose [下载地址](http://download.svicloud.com/harbor-docker-compose-v1.2.2.yml)

```bash
mv harbor-docker-compose-v1.2.2.yml docker-compose.yml 
docker-compose up -d
```

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

# 备注：请求格式
/service/token?account=test&scope=repository:test/repo:push,pull&service=token-service

# scope：指定类型（repository），repo（test/repo），请求的操作权限（pull && push）
# service：即 JWT 验证中的 Audience，Token 接收方（即 registry）
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

# 清理及故障处理

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











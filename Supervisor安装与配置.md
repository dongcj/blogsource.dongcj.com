---
title: Supervisor安装与配置
author: dongcj <ntwk@163.com>
date: 2016/06/13 09:25:58
updated: 2016/08/04 17:26:08
categories:
  - linux
tags:
  - supervisor
---

# 安装
```bash
# 建议使用 yum 安装
yum install supervisor

# 也可以使用 pip 安装
pip install supervisor

# 生成配置文件
echo_supervisord_conf >/etc/supervisord.conf
```


# 配置
> 生成的配置文件不用修改，直接在最后加下自己的配置，以下是样例

```bash
[program:mysql]
environment=API_UMBRELLA_CONFIG="/opt/api-umbrella/var/run/runtime_config.yml"  # 可选
directory=/opt/api-umbrella/embedded/apps/web/current   # 可选
command=service mysqld start    # 必选，启动命令
autorestart=true                # 必选，自动启动
redirect_stderr=true            # 可选
stdout_syslog=true              # 可选

[program:opsview]
command=service opsview start
autorestart=true

[program:opsview-web]
command=service opsview-web start
autorestart=true
```

# 启动
    $ service supervisord start

    > 如果使用 pip 安装，需要自已写启动脚本


# 检查安装
    # 如下即表示正常
    supervisorctl status
    remote-launcher `RUNNING`    pid 11087, uptime 9 days, 2:49:14
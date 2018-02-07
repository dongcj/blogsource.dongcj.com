---
title: 修改 npm 安装源为国内源
tags: 
  - 
categories: 
  - 
updated: 
date: 
author: dongcj <ntwk@163.com>
---

> 淘宝源地址：[http://npm.taobao.org/](http://npm.taobao.org/)

# 删除已有 npm 包 ( 可选 )

```bash
# 查看已安装的 npm package
npm -g ls        

## 删除所有 npm
npm ls -gp | awk -F/ '/node_modules/ && !/node_modules.*node_modules/ {print $NF}' | xargs npm -g rm
```

> npm ls -g 如果报错，可以进行以下处理 :

```bash
# （原因是 node-modules 中不允许用软链接，但 inherits 却用了）：
npm ERR! missing: inherits@*, required by undefined@undefined
npm ERR! missing: inherits@*, required by block-stream@0.0.7
npm ERR! missing: inherits@*, required by fstream@0.1.24

cd /usr/lib/node_modules/
unlink inherit
mv inherits@2/ inherits/
```

```bash
npm update npm -g set registry=https://registry.npm.taobao.org
npm config set ca ""

npm install npm -g set registry=https://registry.npm.taobao.org
npm config delete ca
```

<!-- more -->

# 或者做个 cnpm 的 alias
```bash
# install cnpm
npm install -g cnpm --registry=https://registry.npm.taobao.org

# make alias
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"

# Or alias it in .bashrc or .zshrc
echo '\n#alias for cnpm\nalias cnpm="npm --registry=https://registry.npm.taobao.org \
  --cache=$HOME/.npm/.cache/cnpm \
  --disturl=https://npm.taobao.org/dist \
  --userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshrc

# install module / sync / info
cnpm install <NAME>
cnpm sync connect
cnpm info connect
```


---
title: Screen、nohup、setsid、disown 后台运行
author: dongcj <ntwk@163.com>
date: 2016/08/04 16:20:35
updated: 2017-08-18 01:41:14
categories:
  - linux
tags:
  - screen
  - nohup
  - setsid
  - disown
---
# screen 后台

    screen -S   yourname -> 新建一个叫 yourname 的 session，然后进入
    screen -ls -> 列出当前所有的 session
    screen -r yourname -> 回到 yourname 这个 session
    screen -d yourname -> 远程 detach 某个 session
    screen -d -r yourname -> 结束当前 session 并回到 yourname 这个 session

    # 有用的命令，可以做为同步演示用 , 比 script 强大
    screen -x <SESSION_NAME>

    ## 强制退出某个会话
    screen -X -S [session_you_want_to_kill] quit

    # 在在脚本中使用 screen（直接在指定的 screen 中运行命令，并 Detach 出去）
    screen -Sdm   <SCREEN_NAME>   <DO_SOME_COMMAND>

    ctrl A 然后再按 D 退出会话




# nohup 后台
    nohup ping www.ibm.com &

# setsid 后台
    setsid ping www.ibm.com (ctrl+c 后就跑后台去了 )


# subshell 后台
    (ping www.ibm.com &)


# disown 后台
    disown -h JOBID












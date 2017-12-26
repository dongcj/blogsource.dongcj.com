---
title: Screen、nohup、setsid、disown后台运行
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
# screen后台

    screen -S   yourname -> 新建一个叫yourname的session，然后进入
    screen -ls -> 列出当前所有的session
    screen -r yourname -> 回到yourname这个session
    screen -d yourname -> 远程detach某个session
    screen -d -r yourname -> 结束当前session并回到yourname这个session

    # 有用的命令，可以做为同步演示用, 比script强大
    screen -x <SESSION_NAME>

    ## 强制退出某个会话
    screen -X -S [session_you_want_to_kill] quit

    # 在在脚本中使用screen（直接在指定的screen中运行命令，并Detach出去）
    screen -Sdm   <SCREEN_NAME>   <DO_SOME_COMMAND>

    ctrl A 然后再按 D 退出会话




# nohup 后台
    nohup ping www.ibm.com &

# setsid后台
    setsid ping www.ibm.com (ctrl+c后就跑后台去了)


# subshell后台
    (ping www.ibm.com &)


# disown后台
    disown -h JOBID



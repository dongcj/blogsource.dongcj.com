---
title: VirtualBox 报错 E_FAIL0x80004005 解决方案
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - virtualbox
  - vm
---
> VirtualBox 4.3.12 后的版本才会有 E_FAIL (0x80004005) 问题

# 错误信息：

> 返回 代码 : E_FAIL (0x80004005)
> 组件 : MachineWrap
> 界面 : IMachine {f30138d4-e5ea-4b3a-8858-a059de4c93fd}

总而言之这个问题就是 4.3.12 以后的 VirtualBox 全都无法启动 VM，即使是新建的也不行。
这个问题困扰了我很多年，从 VirtualBox 更新到 4.3.14 以后就一直让我头大。

# 寻找原因
直到最近重装系统，我才找到了问题的原因—— MacType。
是的，MacType 与 VirtualBox（4.3.14 以上版本）冲突。
尽管这并不是真正底层的原因，因为同样类似的问题发生了在了很多外国人身上，我相信他们中很多是不用 MacType 的，但我还是因为 MacType 找到了我自己在这个问题上的解决方案。

# 解决方案
在 MacType 的配置文件中排除 VirtualBox 的进程即可。

```ini
[ExcludeModule]
VBoxSVC.exe
VirtualBox.exe
```

# 另一个问题
鼠标点击 VM 窗口底部图标会导致的 0x00000000 内存不能为 written
```bat
sfc /scannow
```
还是不行的话，有可能是你的主题 (theme) 被替换过，使用工具 UniversalThemePatcher 恢复下就好

通过 google 搜索居然解决这种多年顽疾有多么喜悦是很难让外人理解的，但我真的非常开心 ~

> 参考：https://donneryst.com/blog/virtualbox-4-3-12%E4%BB%A5%E5%90%8E%E7%9A%84e_fail-0x80004005%E9%97%AE%E9%A2%98.html

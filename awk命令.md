---
title: awk 命令
tags: 
  - 
categories: 
  - 
author: dongcj <ntwk@163.com>
categories: 

updated: 
date: 
---

```awk
msg=$(awk '      
    BEGIN { IGNORECASE = 1; ORS = "%20" }
    /FAILED|ERROR/ {
        gsub(/[[:blank:]]+(FAILED|ERROR)/, "")
        gsub(/[[:blank:]]+/, "_")
gsub(/[^[:alnum:]_]/, "")
        print
    }' \
    /var/clone/check/check_baseos.log \
    /var/clone/check/check_gemstone_init.log \
    /var/clone/check/check_pe_profile.log \
    /var/clone/check/check_agent.log 2>/dev/null | cut -c1-70)
``` 
    
    
# 计算第三列的总和
```awk
awk '{ x += $3 } END { print x }' myfile
```

# 1. awk 内建变量示例详解之 NR、FNR、NF、FILENAME
>NR: Number of Record
>NF: Number of Field

```awk
cat class1.txt
zhaoyun 85 87
guanyu 87 88
liubei 90 86

cat class2.txt
caocao 92 87 90
guojia 99 96 92

awk '{print FILENAME,"NR="NR,"FNR="FNR,"$"NF"="$NF}' class1.txt class2.txt
class1.txt NR=1 FNR=1 $3=87
class1.txt NR=2 FNR=2 $3=88
class1.txt NR=3 FNR=3 $3=86
class2.txt NR=4 FNR=1 $4=90
class2.txt NR=5 FNR=2 $4=92
```


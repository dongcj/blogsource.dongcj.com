---
title: 使用 printf 进行格式化
author: dongcj <ntwk@163.com>
date: 2016/08/30 17:27:43
updated: 2016/08/30 17:48:08
categories:
  - linux
tags:
  - shell
  - printf
---

## 表格输出
```bash
    #/bin/bash

    divider===============================
    divider=$divider$divider

    header="\n %-10s %8s %10s %11s\n"
    format=" %-10s %08d %10s %11.2f\n"

    width=43

    printf "$header" "ITEM NAME" "ITEM ID" "COLOR" "PRICE"

    printf "%$width.${width}s\n" "$divider"

    printf "$format" \
    Triangle 13  red 20 \
    Oval 204449 "dark blue" 65.656 \
    Square 3145 orange .7
```

```bash
printf "%s\n" "1" "2" "\n3"
1
2
\n3

printf "%b\n" "1" "2" "\n3"
1
2

3
$

printf "%d\n" 255 0xff 0377 3.5
255
255
255
bash: printf: 3.5: invalid number
3

printf "%f\n" 255 0xff 0377 3.5
255.000000
255.000000
377.000000
3.500000

printf "%.1f\n" 255 0xff 0377 3.5
255.0
255.0
377.0
3.5
```

```bash
for i in $( seq 1 10 ); do printf "%03d\t" "$i"; done
001     002     003     004     005     006     007     008     009     010
```


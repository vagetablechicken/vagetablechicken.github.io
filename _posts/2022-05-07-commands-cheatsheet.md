---
title: 常用命令与脚本
categories: cheatsheet
layout: post
---

## Bash

### Git 批量处理相关文件

move or remove them before you merge, use this to get related files:
```
git xxx 2>&1|grep -E '^\s'|cut -f2-|xargs -I {} echo "{}"
```

### 进程/端口

no ps需要`apt install procps`。
```
ps axu | grep xx | awk '{print $2}' | xargs kill -9
netstat -ltnup
```

### 批量运行当前目录下所有文件

```
for f in *; do ./$f; done
```

### Top 资源监控

记录一段时间进程cpu mem等资源的变化，适合抓出程序最大占用内存的时间点。
```bash
while true; do sleep 10 && top -b -p 25440 -n1 | tail -1 ; done >> process.top
# add time
while true; do sleep 10 && date && top -b -p 25440 -n1 | tail -1 ; done >> process.top
```

## Debug

### OOM

怀疑进程被oom kill了，需要查日志，dmesg -T没有权限要求，但它的时间[不准确](https://zhuanlan.zhihu.com/p/619173424?utm_id=0)，甚至可能是未来时间。最好查/var/log/messages，但它可能需要root权限。
Docker内的进程也是被物理机管理的，如果占满内存，也是会被kill，而且在物理机上会有日志。

## Regex

虽然真的很难懂，但是确实用处很多，逃不开它的魔爪。测试工具推荐 https://regex101.com/ ，有解释regex的意思，容易debug。让AI先写，再自己用测试工具验证，是个比较高效的方法。

hive的regexp_replace好像不是遵守java的regex规则，比如dash '-'不需要转义，但是java里需要转义？
'\\\-\'"'这样的写法，hive里还会报错，得把-换回原始-，才能正常运行？

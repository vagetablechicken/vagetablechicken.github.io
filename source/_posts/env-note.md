---
title: 环境note
tags:
  - env
  - os
date: 2022-05-07 00:23:43
---

# 环境note

## About OS

### macOS

#### homebrew

默认源太卡，容易install失败。换tuna的源，设置方法见

[homebrew](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)，[homebrew-bottles](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew-bottles/)。（注意，这两个都要配置）

#### gdb/lldb

mac中使用lldb，不需要指定bin。`lldb -c /cores/xxx`即可。

### linux

### debian源

[debian | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)

apt update

apt install maven

### Windows

Windows下的终端Terminal，如何加入snippet，参考https://github.com/microsoft/terminal/issues/6412#issuecomment-964343941。我个人常为OpenMLDB加的snippet：
```json
        {
            "command": {
                "action": "sendInput",
                "input": "set @@execute_mode='online';\r"
            },
            "name": "on"
        },
        {
            "command": {
                "action": "sendInput",
                "input": "set @@execute_mode='offline';\r"
            },
            "name": "off"
        },
        {
            "command": {
                "action": "sendInput",
                "input": "set @@sync_job=true;\r"
            },
            "name": "sync"
        },
        {
            "command": {
                "action": "sendInput",
                "input": ":set mouse-=a\r"
            },
            "name": "mouse"
        }
```
不需要快捷键，`Ctrl+Shift+P`可以根据name快速选择，快捷键还懒得记。

## vim

vim时用鼠标选择一段文本，可能进入VISUAL模式。VISUAL模式下的复制/粘贴/剪切得用`y`,`p`,`d`。注意，VISUAL模式下复制的文本，不会记录在剪贴板，只能在vim中使用，拷贝不出去。

更习惯不进入VISUAL模式的话，`set mouse-=a`。更改默认配置，把这个设置放在`~/.vimrc`里。

## python

python源也可以用tuna的，可以直接

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <pkg>
```

全局配置方法

```
python -m pip install --upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

详情见[pypi | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)。

## sh

### git

move or remove them before you merge, use this to get related files

```
git xxx 2>&1|grep -E '^\s'|cut -f2-|xargs -I {} echo "{}"
```

### ps and network

no ps需要`apt install procps`。

```
ps axu | grep xx | awk '{print $2}' | xargs kill -9
netstat -ltnup
```

### run bins

运行当前目录下所有文件，测试时常用。

```
for f in *; do ./$f; done
```

### top record

记录一段时间进程cpu mem等资源的变化，适合抓出程序最大占用内存的时间点。
```bash
while true; do sleep 10 && top -b -p 25440 -n1 | tail -1 ; done >> process.top
# add time
while true; do sleep 10 && date && top -b -p 25440 -n1 | tail -1 ; done >> process.top
```

## npm

如果目的是pnpm，不用单独下载npm，直接下载pnpm就行。 https://pnpm.io/zh/installation

如果是npm，下载nvm更合适管理node版本。

## keyboard

rk61键盘配合mac使用，支持很差，还是需要改键。使用karabiner-element做改键。

可以直接修改`~/.config/karabiner/karabiner.json`，注意里面的device是有vendor id和product id的，得填对，可以用karabiner直接查到。

windows下改键，使用PowerToys的键盘管理器。

## docker源

就是`registry-mirrors`这个配置项。linux直接在 `/etc/docker/daemon.json`里改，mac可以在docker desktop设置里找到配置文件，也是一样的修改。

```
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}
```

重启后`docker info`可以查看。

## tmux

tmux如果另一个没退，就又attach，可能分辨率会用另一个的，就会右边下边出现很多点点。tmux命令`choose-client`，选择分辨率，选小的就能撑满屏幕。

默认`choose-client`是shift-b，但我改了tmux的prefix，所以是ctrl-a，再shift-d。自己查自己tmux的keys，用`choose-client` bind的key。

https://unix.stackexchange.com/a/174454 window name highlight

tmux里zsh无法用home end键：zshrc更改配置可以绑键
```
#Rebind HOME and END to do the decent thing:
bindkey '\e[1~' beginning-of-line
bindkey '\e[4~' end-of-line
case $TERM in (xterm*)
bindkey '\eOH' beginning-of-line
bindkey '\eOF' end-of-line
esac

#To discover what keycode is being sent, hit ^v
#and then the key you want to test.

#And DEL too, as well as PGDN and insert:
bindkey '\e[3~' delete-char
bindkey '\e[6~' end-of-history
bindkey '\e[2~' redisplay

#Now bind pgup to paste the last word of the last command,
bindkey '\e[5~' insert-last-word
```

## debug

### oom

怀疑进程被oom kill了，需要查日志，dmesg -T没有权限要求，但它的时间[不准确](https://zhuanlan.zhihu.com/p/619173424?utm_id=0)，甚至可能是未来时间。最好查/var/log/messages，但它可能需要root权限。
Docker内的进程也是被物理机管理的，如果占满内存，也是会被kill，而且在物理机上会有日志。

---
title: 工具与软件配置
date: 2022-05-07 10:00:00 +0800
tags: [tools, config]
categories: [Env, Tools, Cookbook]
pin: true
---

## ShareX

截图自带当前时间戳的做法：

在“任务设置”->“图像”->“特效“->“图像特效配置”中增加预设，起名timestamp，效果text，内容直接填`%y-%mo-%d %h-%m-%s`，可以按下拉菜单参考语法，位置和字体大小颜色自己调。

然后在“截图后的任务”中，“添加图像特效”勾选timestamp即可。这里比较离谱，截图后会触发的任务只会字体加粗，不太明显，仔细点看。

## VPN & git

VPN一般v2ray系列最好用。

github repo clone走ssh通道，有时候走不通，具体错误比如，连不上github 22等。vpn默认只会代理http/https流量，ssh要单独配置ssh代理。有时候http也可能有问题，不过git config set http.proxy或https.proxy比较方便。

```
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

在`.ssh/config`中配置:
```
Host github.com
    HostName github.com
    User git
    # windows style
    # ProxyCommand connect -S 127.0.0.1:10808 %h %p
    # ProxyCommand connect -S 127.0.0.1:7897 %h %p
    # linux style
    # ProxyCommand nc -X 5 -x 127.0.0.1:10808 %h %p
    # ProxyCommand nc -X 5 -x 127.0.0.1:7897 %h %p
```

10808是v2rayN的socks port，开v2rayN就会开启的，其他VPN看具体配置。Clash Verge端口默认是7897，混合端口。

## vim

vim时用鼠标选择一段文本，可能进入VISUAL模式。VISUAL模式下的复制/粘贴/剪切得用`y`,`p`,`d`。注意，VISUAL模式下复制的文本，不会记录在剪贴板，只能在vim中使用，拷贝不出去。

更习惯不进入VISUAL模式的话，`set mouse-=a`。更改默认配置，把这个设置放在`~/.vimrc`里。

## Python

Python源可以用tuna的，有些网络华为源更快。

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <pkg>
# 全局配置方法
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

Python numpy大升级出现很多不兼容，一个通常适用的老版本：
```
pip install numpy==1.26.4  
pip install scipy==1.12.0  
pip install scikit-learn==1.2.2
```

### uv

uv比较适合做复杂的库依赖管理，format支持较好，比较适合团队项目。

uv推荐不要pip，而是直接安装在本地，即standalone模式，见[官方文档](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer)。

vscode可能会自动带一些缓存，导致切换python路径总是失败，具体可能：
- uv python pin了不起作用，可能是vscode内存了`$env:UV_PYTHON`，一般先在普通shell里测试，避免vscode帮倒忙。

## npm

如果目的是pnpm，不用单独下载npm，直接下载pnpm就行。 https://pnpm.io/zh/installation

如果是npm，下载nvm更合适管理node版本。
```bash
# windows https://github.com/coreybutler/nvm-windows?tab=readme-ov-file
# ubuntu https://github.com/nvm-sh/nvm.git/
nvm install lts
nvm on
npm i -g yarn
```

## docker

### 导入导出

**千万不要用docker export/import**，大的image导入导出都慢的可怕。用save/load。

### 换源

配置换源是个非常重的操作，要重启docker。不是必须这么做，一般先考虑给pull的地址加镜像前缀，比如daocloud的，具体去搜。

#### 配置换源

就是`registry-mirrors`这个配置项。linux直接在 `/etc/docker/daemon.json`里改，mac/win可以在docker desktop设置里找到配置文件，也是一样的修改。

```
{
    "registry-mirrors": [
        "..."
    ]
}
```

重启后`docker info`可以查看。

源的地址很容易失效，github有些项目会帮忙检查，即时去查一下，这里不提供了，以免失效。

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

## GO

gvm来管理，但gvm要先下一个早期版本，才能去下一些高版本，依赖关系比较诡异，照着下面安装就行。
```bash
# -B 直接下binary，gcc编1.4可能有一堆c++ warning，编译通不过
gvm install go1.4 -B
# Set $GOROOT_BOOTSTRAP to a working Go tree >= Go 1.17.13.
gvm use go1.4 --default
# 装比1.17还高的时候有需要1.17作为基础，1.4直接装不了1.21。。。有毒
gvm install go1.17.13
gvm use go1.17 --default # 可以就用17这个版本当默认版
gvm install go1.21
```

---
title: 工具与软件配置
date: 2022-05-07 10:00:00 +0800
tags: [tools, config]
categories: [Env, Tools, Cookbook]
---

## ShareX

截图自带当前时间戳的做法：

在“任务设置”->“图像”->“特效“->“图像特效配置”中增加预设，起名timestamp，效果text，内容直接填`%y-%mo-%d %h-%m-%s`，可以按下拉菜单参考语法，位置和字体大小颜色自己调。

然后在“截图后的任务”中，“添加图像特效”勾选timestamp即可。这里比较离谱，截图后会触发的任务只会字体加粗，不太明显，仔细点看。

## VPN & git

VPN一般v2ray系列最好用。

github repo clone走ssh通道，有时候走不通，具体错误比如，连不上github 22等。vpn默认只会代理http/https流量，ssh要单独配置ssh代理。有时候http也可能需要明确set，git config set http.proxy或https.proxy比较方便，例子在下面。

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

```bash
# 如果没在git项目中，必须--global
git config --global http.proxy 'http://127.0.0.1:10809'
git config --global https.proxy 'http://127.0.0.1:10809'
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## Vim

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

#### 换源
临时：
```
$env:UV_INDEX_URL = "https://pypi.tuna.tsinghua.edu.cn/simple"
$env:UV_PYTHON_INSTALL_MIRROR = "https://python-standalone.org/mirror/astral-sh/python-build-standalone"

export UV_INDEX_URL="https://pypi.tuna.tsinghua.edu.cn/simple"
export UV_PYTHON_INSTALL_MIRROR="https://python-standalone.org/mirror/astral-sh/python-build-standalone"
```

个人PC等长期用的，写在uv.toml，windows路径`$env:APPDATA\uv\uv.toml`，linux路径`~/.config/uv/uv.toml`:
```
[[index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

项目里pyproject.toml也能填，但不建议开源项目加，地域限制:
```
[[tool.uv.index]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

## npm

如果目的是pnpm，不用单独下载npm，直接下载[pnpm](https://pnpm.io/zh/installation)。

如果是npm，下载nvm更合适管理node版本。
```bash
# windows https://github.com/coreybutler/nvm-windows?tab=readme-ov-file
# ubuntu https://github.com/nvm-sh/nvm.git/
nvm install lts
nvm on
npm i -g yarn
```

## Go

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

## 键盘改键

rk61键盘配合mac使用，支持很差，还是需要改键。使用karabiner-element做改键。

可以直接修改`~/.config/karabiner/karabiner.json`，注意里面的device是有vendor id和product id的，得填对，可以用karabiner直接查到。

windows下改键，使用PowerToys的键盘管理器。

## Google账号

Google账号注册多个时，可以通过手机验证跳过手机号绑定，但实际它还是建立了关联，只是没有把手机号显示在账号信息里。所以，注册账号超过一定数量后，Google会阻止手机设备继续辅助验证。

Google新注册账号，网页上注册需要手机主动发验证码，+86一般发不了，安卓设备上用Google Play商店注册，手机上不需要发验证码，它可以自动从设备信息里获取验证。

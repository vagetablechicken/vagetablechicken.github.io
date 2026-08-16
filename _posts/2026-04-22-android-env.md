---
title: Android 环境配置
date: 2026-04-22 20:00:00+0800
tags: [devops]
categories: [DevOps]
---

Android 环境基本不用来开发，比如笔记同步，网络，摄像头，电视等等。

## Logseq 笔记同步

GitSync有时候会炸，使用termux作为保底，最基础的git同步很难出问题。参考了[Logseq Android 同步](https://github.com/charliie-dev/Logseq-Git-Sync-101/wiki/For-Android-users)，再加了一些自己的使用习惯。

```
termux-setup-storage
pkg install git vim less
vim ~/.bashrc
```

.bashrc中填，方便跳转，termux内路径跟版本有关，目前我的setup storage后在`~/storage/shared`下，我一般把仓库放Documents里，方便安卓文件系统访问。安卓输入法不是很好用，加一些缩写：
```
#!/bin/bash
cat << 'EOF' >> ~/.bashrc
alias doc='cd ~/storage/shared/Documents'
alias logseq='cd ~/storage/shared/Documents/logseq_storage'
alias sync='bash ~/storage/shared/Documents/logseq_storage/scripts/termux_auto_sync.sh'
alias glg="git log --graph --all --date=format:'%Y-%m-%d %H:%M' --pretty=format:'%C(yellow)%h%Creset %C(green)[%ad]%Creset%C(auto)%d%Creset %s'"
alias gs='git status'
alias gl='git pull'
alias gp='git push'
alias gc='git commit -m'
EOF
source ~/.bashrc
```

```bash
git config --global credential.helper store
# 如果用错了，用git credential-cache exit忘记
# pull等方式提醒输入用户名密码，我用gitlab，可以token配置，注意要带上code download权限
git pull
```

termux widget可以用，但要termux和widget都从F-Droid安装，Google Play版不支持widget。scripts复制到`~/.shortcuts`，然后在桌面添加termux widget小组件（注意不是app本身，是小组件，要给单独脚本加icon，要加png才能自动解析出来，没什么必要），选择脚本即可。

```bash
mkdir ~/.shortcuts
chmod 700 ~/.shortcuts

mkdir -p ~/.shortcuts/icons
chmod -R a-x,u=rwX,go-rwx ~/.shortcuts/icons
# The icon file name must be equal to <script_name>.png, like script.sh.png.
```

autosync脚本内容：
```bash
#!/bin/bash

# ==================== 配置项 ====================
# 你的 Logseq 仓库在手机上的路径
REPO_PATH="$HOME/storage/shared/Documents/logseq_storage"

# 你的远程分支名称
BRANCH_NAME="main"
# ================================================

# 1. 检查本地仓库路径是否存在
if [ ! -d "$REPO_PATH" ]; then
    echo "错误：找不到指定的仓库路径 -> $REPO_PATH"
    echo "请检查："
    echo "1. 路径拼写是否正确（Linux 对大小写敏感，请确认是 documents 还是 Documents）"
    echo "2. 是否已在 Termux 中执行过 termux-setup-storage"
    exit 1
fi

# 进入仓库目录
cd "$REPO_PATH" || exit 1

echo "=== 开始 Android Termux 同步 Logseq: $(date +'%Y-%m-%d %H:%M:%S') ==="

# 2. 拉取远程最新代码
echo "正在拉取远端更新 (pull)..."
if ! git pull --rebase origin "$BRANCH_NAME"; then
    echo "警告：拉取失败，可能存在代码冲突或网络问题，请手动处理。"
    exit 1
fi

# 3. 检查是否有未提交的本地修改
if [[ -n $(git status --porcelain) ]]; then
    echo "检测到本地有修改，正在打包提交..."
    
    # 添加所有变动
    git add .
    
    # 提交改动，明确标注是 Android Termux 同步
    git commit -m "Android Termux sync Logseq: $(date +'%Y-%m-%d %H:%M:%S')"
    
    # 推送到远程仓库
    echo "正在推送至远程仓库 (push)..."
    if git push origin "$BRANCH_NAME"; then
        echo "=== 同步成功推送到远端 ==="
    else
        echo "错误：推送失败，请检查网络或 Git 凭据是否有效。"
        exit 1
    fi
else
    echo "本地暂无需要提交的新修改。"
    echo "=== 同步流程结束 ==="
fi
```

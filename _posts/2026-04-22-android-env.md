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
git config --global core.quotepath false # 文件使用中文名可以，但特殊符号最好不要用，不同平台支持不一样，可能会一个平台能拉一个平台拉不下来
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
REPO_PATH="$HOME/storage/shared/Documents/logseq_storage"
BRANCH_NAME="main"
# ================================================

# 1. 检查路径
if [ ! -d "$REPO_PATH" ]; then
    echo "错误：找不到指定的仓库路径 -> $REPO_PATH"
    exit 1
fi

cd "$REPO_PATH" || exit 1
echo "=== 开始 Android Termux 同步 Logseq: $(date +'%Y-%m-%d %H:%M:%S') ==="

# 2. 检查本地是否有修改，有的话暂存 (Stash)
HAS_LOCAL_CHANGES=false
if [[ -n $(git status --porcelain) ]]; then
    echo "检测到本地修改，正在执行暂存 (git stash)..."
    # 必须加 -u 参数，否则 Logseq 新建的页面（未跟踪文件）不会被收录进去
    git stash -u
    HAS_LOCAL_CHANGES=true
fi

# 3. 拉取远程最新代码
echo "正在拉取远端更新 (pull)..."
if ! git pull --rebase origin "$BRANCH_NAME"; then
    echo "❌ 错误：拉取失败 (远端历史不兼容或网络问题)。"
    # 如果 pull 失败了，尽量把刚才藏起来的代码还给你
    if [ "$HAS_LOCAL_CHANGES" = true ]; then
        echo "尝试恢复本地修改..."
        git stash pop
    fi
    exit 1
fi

# 4. 恢复本地修改 (如果之前有暂存)
if [ "$HAS_LOCAL_CHANGES" = true ]; then
    echo "正在恢复本地修改 (git stash pop)..."
    if ! git stash pop; then
        echo "========================================================"
        echo "🚨 警告：恢复修改时发生冲突 (Merge Conflict)！"
        echo "这说明你在手机和电脑上修改了同一个页面的同一行。"
        echo "Git 已将冲突标记 (<<<<<<<) 写入笔记文件中。"
        echo "👉 解决方法：直接打开 Logseq，找到那条笔记，手动删掉不需要的文本和乱码符号。"
        echo "清理完毕后，再次运行本脚本即可完成推送。"
        echo "========================================================"
        exit 1
    fi
fi

# 5. 提交并推送
if [[ -n $(git status --porcelain) ]]; then
    echo "准备提交已合并的修改..."
    git add .
    git commit -m "Android Termux sync: $(date +'%Y-%m-%d %H:%M:%S')"
    
    echo "正在推送至远程仓库 (push)..."
    if git push origin "$BRANCH_NAME"; then
        echo "=== 🎉 同步成功推送到远端 ==="
    else
        echo "❌ 错误：推送失败，请检查网络。"
        exit 1
    fi
else
    echo "本地暂无需要提交的新修改。"
    echo "=== ✅ 同步流程结束 ==="
fi
```

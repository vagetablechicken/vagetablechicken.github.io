---
title: Android 环境配置
date: 2026-04-22 20:00:00+0800
tags: [devops]
categories: [DevOps]
---

Android 环境基本不用来开发，比如笔记同步，网络，摄像头，电视等等。

## Logseq 笔记

GitSync有时候会炸，使用termux作为保底，最基础的git同步很难出问题。

```
termux-setup-storage
pkg install git vim
vim ~/.bashrc
```

.bashrc中填
```
export GIT_DISCOVERY_ACROSS_FILESYSTEM=1
alias doc='cd /storage/emulated/0/Documents'
alias logseq='cd /storage/emulated/0/Documents/logseq_storage'
```

```bash
git config --global credential.helper store
# 如果用错了，用git credential-cache exit忘记
# pull等方式提醒输入用户名密码，我用gitlab，可以token配置，注意要带上code download权限
git pull
```
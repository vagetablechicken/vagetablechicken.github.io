---
title: 环境note
tags:
  - env
  - os
date: 2022-05-07 00:23:43
---

# 环境note

## macOS

### homebrew

默认源太卡，容易install失败。换tuna的源，设置方法见

[homebrew](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)，[homebrew-bottles](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew-bottles/)。（注意，这两个都要配置）

## linux

## vim

vim时用鼠标选择一段文本，可能进入VISUAL模式。VISUAL模式下的复制/粘贴/剪切得用`y`,`p`,`d`。注意，VISUAL模式下复制的文本，不会记录在剪贴板，只能在vim中使用，拷贝不出去。

更习惯不进入VISUAL模式的话，`set mouse-=a`。更改默认配置，把这个设置放在`~/.vimrc`里。

## python

python源也可以用tuna的，可以直接

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <pkg>
```

全局配置方法见[pypi | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)。

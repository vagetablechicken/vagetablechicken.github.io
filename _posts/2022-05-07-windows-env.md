---
title: Windows 环境配置
date: 2022-05-07 10:00:00 +0800
categories: [Env, Windows]
tags: [windows, wsl2, powershell, terminal]
---

## 安装与分区

盘格式NTFS，创建分区时要注意不要用MBR，直接GPT。windows10/11都支持GPT分区启动。

服务器主机安装时最好把BIOS的AC Recovery打开，防止系统崩溃后无法启动。

## Python

windows python通过pip install是要现编译的，VS的编译输出是中文，terminal就显示乱码。需要win11打开”区域“，管理-更改系统区域设置，打开utf-8支持，就不会乱码了。但这么做，别的地方可能乱码，不是永久方案，需要用时打开一下。可以看下VS能不能改语言，不确定能不能解决。

windows装dlib用pip一般会报编译错误。`conda install -c conda-forge dlib`下载的依赖比较多，能安装成功。

windows conda最麻烦的是scripts相关的，conda init得到的环境下conda命令实际是脚本，很多命令都没有。

## WSL2+Docker

注意windows的Docker Desktop不用wsl2就能跑，docker desktop自带一个轻量级linux内核，但如果有比较复杂的需求，还是建议直接上wsl2，比如：

- gitlab image等对windows文件系统支持差，但如果用wsl2的ext4文件系统就没问题，也就是必须volume bind到wsl2的文件系统。

简而言之，先装wsl2，要不要长期使用另说。

```
wsl --install -d Ubuntu
```

网速正常的话，一般就安装好了。但有时候会卡在诡异的地方，wsl命令分几个版本，默认版本用起来很不顺手，但一般懒得重新下。通常是ubuntu镜像下了，wsl内核没下，多跑几下install命令，可能就好了。

wsl涉及到使用ip就很麻烦，把这部分配置好，方便使用，见[gist](https://gist.github.com/vagetablechicken/1f30f57c178536aa6135ea06b33d66ec)。这部分配置只在2024年12月前测试过，后续windows更新可能会失效。

wsl2默认只有1个T，模型存的多就用完了，expand一下，不会重置环境，只需要shutdown一下，见[官方文档](https://learn.microsoft.com/en-us/windows/wsl/disk-space#how-to-expand-the-size-of-your-wsl-2-virtual-hard-disk)。

wsl2无论什么端口，从win都能用localhost地址访问到，这不代表ip就通了。localhost和127都可能无法等同，估计是wsl2只加工了localhost。

## PowerShell

PowerShell在中文版windows上容易遇到编码问题，但更换英文版Windows很麻烦。

如果目的不是在shell中展示日志，可以使用vscode自动猜编码，使用正确的编码。shell中操作文件，只需要记得读和写都设置一下utf-8即可。

PowerShell自行启动的和vscode内启动的还可能使用不同的profile，在你使用的shell中打印可以知道应该修改哪个profile文件。

```
$profile
```

user profile在文档目录中，自己启动的名字是`Microsoft.PowerShell_profile.ps1`，vscode用的是`Microsoft.VSCode_profile.ps1`。两者都改了，还是可能无法作用于Start-Process重定向。测试不重定向，还是Start-Process，弹出的shell里都不会乱码。重定向的编码确实无法改变，Start-Process加load user profile也不行。

目前还没有找到彻底解决方案。

## Terminal snippet

Windows下的终端Terminal加入snippet，参考https://github.com/microsoft/terminal/issues/6412#issuecomment-964343941。

我个人常为OpenMLDB加的snippet：
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

遇到`git clone error: invalid path`，用wsl git，然后再mv过去，windows git有点坑。

## Windows Antigravity

Google Antigravity 打开的时候会闪现一个窗口迅速关闭，在某些机器上甚至会窗口报`已退出进程，代码为1`。没有证明这两者是同一个东西，但可能性比较大。调试方案：

```powershell
# sudo权限

# 进程创建/退出添加事件监控


# python脚本监听，比较方便

```

监听脚本：
```python
```

Antigravity启动时确实会扫一遍所有terminal，包括powershell、cmd、wsl、git bash等，查版本之类的。确实比较大的可能是wsl的问题，wsl安装本来就是一堆bug，报`已退出进程，代码为1`的可能性也比较大。

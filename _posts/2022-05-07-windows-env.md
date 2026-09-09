---
title: Windows 环境配置
date: 2022-05-07 10:00:00 +0800
categories: [Env, Windows]
tags: [windows, wsl2, powershell, terminal]
---

## 安装与分区

盘格式NTFS，创建分区时要注意不要用MBR，直接GPT。windows10/11都支持GPT分区启动。

服务器主机安装时最好把BIOS的AC Recovery打开，防止系统崩溃后无法启动。

EasyU PE环境，用WinNTSetup安装系统，GPT分区格式就要用UEFI装，具体来讲，引导驱动器选安装盘一个框里的esp分区（小的），efi part的那个灯要绿，黄的没必要装，引导不了，安装驱动器选要当系统盘的分区。

## Windows Server OS WLAN安装

主板驱动下载后安装可能失败，“段落无效”。把WLAN Config功能打开，然后启动这个服务，最后安装WLAN驱动。

```powershell
dism /online /enable-feature /featurename:WirelessNetworking
```

## Python

windows python通过pip install是要现编译的，VS的编译输出是中文，terminal就显示乱码。需要win11打开”区域“，管理-更改系统区域设置，打开utf-8支持，就不会乱码了。
但这么做，别的地方可能乱码，不是永久方案，需要用时打开一下。

可以看下VS能不能改语言，不确定能不能解决。

windows装dlib用pip一般会报编译错误。`conda install -c conda-forge dlib`下载的依赖比较多，能安装成功。

windows conda最麻烦的是scripts相关的，conda init得到的环境下conda命令实际是脚本，很多命令都没有。推荐用uv。

## WSL2+Docker

windows server先`wsl --install`后重启，再继续装ubuntu镜像。docker desktop engine可能起不来，可能是先装的docker，要先wsl再docker的顺序来。

注意windows的Docker Desktop不用wsl2就能跑，docker desktop自带一个轻量级linux内核， mount volume如果有chown和chmod问题，通过启动一个alpine来改一般能行。

但有些更复杂的情况，还是建议直接上wsl2，比如：

- gitlab image 等对 windows 文件系统支持差，用 wsl2 的 ext4 文件系统就没问题，也就是必须 mount volume 到wsl2的文件系统。

简而言之，先装wsl2，要不要长期使用另说。

```
wsl --install -d Ubuntu
```

网速正常的话，一般就安装好了。但有时候会卡在诡异的地方，wsl命令分几个版本，默认版本用起来很不顺手，但一般懒得重新下。通常是ubuntu镜像下了，wsl内核没下，多跑几下install命令，可能就好了。

wsl涉及到使用ip就很麻烦，把这部分配置好，方便使用，见[gist](https://gist.github.com/vagetablechicken/1f30f57c178536aa6135ea06b33d66ec)。
这部分配置只在2024年12月前测试过，后续windows更新可能会失效。

现在有WSL Settings可视化界面来配置，应该不需要wslconfig手写文件了。

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
sudo auditpol /set /subcategory:"进程创建" /success:enable /failure:enable
sudo auditpol /set /subcategory:"进程终止" /success:enable /failure:enable

sudo auditpol /get /subcategory:"进程创建"
sudo auditpol /get /subcategory:"进程终止"

# python脚本监听，比较方便，见下方
pip install pywin32
python monitor_events.py

sudo auditpol /set /subcategory:"进程创建" /success:disable /failure:disable
sudo auditpol /set /subcategory:"进程终止" /success:disable /failure:disable
```

监听脚本：
```python
import win32evtlog
import win32security
import win32api
import time
import datetime

# ================= 配置区域 =================
# 监控目标关键词 (不区分大小写)
TARGET_PROCESS_KEYWORD = "cmd.exe" 

# 日志源设置
LOG_SERVER = "localhost"
LOG_TYPE = "Security"

# 内存字典：用于存储"活着"的进程信息
# Key: PID (十六进制字符串), Value: {start_time, cmd, parent_pid, name}
active_processes = {}
# ===========================================

def enable_security_privilege():
    """
    [关键] 提升当前进程权限，允许读取 Security 安全日志
    """
    try:
        flags = win32security.TOKEN_ADJUST_PRIVILEGES | win32security.TOKEN_QUERY
        htoken = win32security.OpenProcessToken(win32api.GetCurrentProcess(), flags)
        id = win32security.LookupPrivilegeValue(None, "SeSecurityPrivilege")
        new_privs = [(id, win32security.SE_PRIVILEGE_ENABLED)]
        win32security.AdjustTokenPrivileges(htoken, 0, new_privs)
        # print("[System] 权限提升成功")
    except Exception as e:
        print(f"[Warning] 提权失败 (可能无法读取日志): {e}")

def get_status_desc(hex_status):
    """
    将晦涩的十六进制退出代码转换为人话
    """
    status_map = {
        "0x0": "成功 (Success)",
        "0x1": "通用错误 (Incorrect Function)",
        "0x2": "文件未找到 (File Not Found)",
        "0x5": "拒绝访问 (Access Denied)",
        "0x80004005": "未指定错误 (Unspecified Error)",
        "0xc0000005": "内存访问冲突 (Access Violation)",
        "0xc000013a": "被强制中断 (Ctrl+C / Kill)",
        "0xc0000409": "堆栈缓冲区溢出 (Stack Buffer Overrun)",
        "0xffffffff": "通用失败 (-1)"
    }
    return status_map.get(hex_status, "未知状态")

def parse_event_smart(event):
    """
    智能解析器：提取关键信息
    """
    data = event.StringInserts
    if not data: return None
    
    info = {
        "pid": None,
        "name": None,
        "cmdline": "N/A",
        "parent_pid": None,
        "hex_candidates": [] # 存放所有 0x... 格式的数据
    }
    
    # 1. 扫描所有字段提取特征
    for idx, s in enumerate(data):
        s_str = str(s)
        
        # 提取十六进制数 (PID, ExitCode, LogonID 都是这种格式)
        if s_str.startswith("0x"):
            info["hex_candidates"].append(s_str)
            
        # 提取进程名 (包含 .exe 的)
        if ".exe" in s_str.lower() and "\\" in s_str:
            # 取最长的那个通常是全路径
            if info["name"] is None or len(s_str) > len(info["name"]):
                info["name"] = s_str

    # 2. 针对 4688 (创建) 的补充逻辑
    if event.EventID == 4688:
        if len(data) > 8:
            info["cmdline"] = str(data[8])
        
        # Win10/11 标准: [4]是CreatorPid, [5]是NewPid
        if len(data) > 5 and str(data[5]).startswith("0x"):
            info["pid"] = str(data[5])
            info["parent_pid"] = str(data[4])
        elif len(info["hex_candidates"]) >= 2:
            info["parent_pid"] = info["hex_candidates"][0]
            info["pid"] = info["hex_candidates"][1]

    return info

def monitor_main():
    enable_security_privilege()
    print(f"=== 全生命周期监控已启动 ===")
    print(f"目标进程: {TARGET_PROCESS_KEYWORD}")
    print(f"正在监听创建(4688) 和 退出(4689)... (按 Ctrl+C 停止)\n")
    
    print(f"{'时间':<10} | {'类型':<6} | {'PID':<8} | {'详情'}")
    print("-" * 100)

    last_timestamp = datetime.datetime.now()

    while True:
        try:
            hand = win32evtlog.OpenEventLog(LOG_SERVER, LOG_TYPE)
            flags = win32evtlog.EVENTLOG_BACKWARDS_READ | win32evtlog.EVENTLOG_SEQUENTIAL_READ
            
            events = win32evtlog.ReadEventLog(hand, flags, 0)
            
            new_events = []
            if events:
                for event in events:
                    if event.TimeGenerated <= last_timestamp:
                        break
                    new_events.append(event)
            
            if new_events:
                last_timestamp = new_events[0].TimeGenerated
                
                # 按时间正序处理 (生 -> 死)
                for event in reversed(new_events):
                    
                    # ----------------------------------------------------
                    # 场景 1: 进程创建 (4688)
                    # ----------------------------------------------------
                    if event.EventID == 4688:
                        info = parse_event_smart(event)
                        
                        if info["name"] and TARGET_PROCESS_KEYWORD in info["name"].lower():
                            pid = info["pid"]
                            if pid:
                                active_processes[pid] = {
                                    "start_time": event.TimeGenerated,
                                    "cmd": info["cmdline"],
                                    "parent": info["parent_pid"],
                                    "name": info["name"]
                                }
                                t_str = str(event.TimeGenerated)[11:19]
                                print(f"{t_str:<10} | \033[92m创建\033[0m   | {pid:<8} | 命令: {info['cmdline']}")

                    # ----------------------------------------------------
                    # 场景 2: 进程退出 (4689)
                    # ----------------------------------------------------
                    elif event.EventID == 4689:
                        info = parse_event_smart(event)
                        
                        # 只要名字匹配，我们就处理
                        if info["name"] and TARGET_PROCESS_KEYWORD in info["name"].lower():
                            
                            found_pid = None
                            exit_code = "0x0" # 默认为成功，防止误报
                            
                            # A. 确定 PID
                            # 先去内存里查
                            for h in info["hex_candidates"]:
                                if h in active_processes:
                                    found_pid = h
                                    break
                            
                            # 如果没查到，通常 4689 的 PID 是 hex_candidates 的倒数第一个
                            if not found_pid and info["hex_candidates"]:
                                found_pid = info["hex_candidates"][-1]

                            # B. 确定 Exit Code (核心修复逻辑)
                            candidates = [h for h in info["hex_candidates"] if h != found_pid]
                            
                            if "0x0" in candidates:
                                # 只要有 0x0，那就是成功
                                exit_code = "0x0"
                            elif any(len(c) > 8 for c in candidates):
                                # 如果有长得像错误码的 (0xc0000005)，取最长的
                                exit_code = max(candidates, key=len)
                            elif candidates:
                                # 如果没有 0x0，也没有长错误码，取【最后一个】
                                # 因为 LogonID (0x132d3a) 这种通常在前面，ExitCode 在后面
                                exit_code = candidates[-1]

                            # C. 计算数据
                            duration = "N/A"
                            original_cmd = "未知 (脚本启动前已运行)"
                            
                            if found_pid and found_pid in active_processes:
                                start_data = active_processes[found_pid]
                                try:
                                    delta = event.TimeGenerated - start_data["start_time"]
                                    duration = f"{delta.total_seconds():.2f}s"
                                except: pass
                                original_cmd = start_data["cmd"]
                                del active_processes[found_pid]

                            # D. 打印
                            t_str = str(event.TimeGenerated)[11:19]
                            status_desc = get_status_desc(exit_code)
                            
                            # 颜色判定: 非0且非Unknown为红
                            if exit_code != "0x0":
                                type_str = "\033[91m异常\033[0m" # 红
                            else:
                                type_str = "退出"       # 白
                            
                            print(f"{t_str:<10} | {type_str:<6} | {found_pid:<8} | 代码: {exit_code} ({duration})")
                            
                            if exit_code != "0x0":
                                print(f"{' ':<30} ^ 状态: {status_desc}")
                                if "未知" not in original_cmd:
                                    print(f"{' ':<30} ^ 原令: {original_cmd}")

            win32evtlog.CloseEventLog(hand)
            time.sleep(0.5)

        except KeyboardInterrupt:
            print("\n监控已停止。")
            break
        except Exception as e:
            # print(f"Error: {e}")
            time.sleep(1)

if __name__ == '__main__':
    monitor_main()
```

Antigravity启动时确实会扫一遍所有terminal，包括powershell、cmd、wsl、git bash等，查版本之类的。确实比较大的可能是wsl的问题，wsl安装本来就是一堆bug，报`已退出进程，代码为1`的可能性也比较大。

Google账号地区问题，看 https://policies.google.com/country-association-form ，填写表格申请更改账号国家地区。还是出现过美区账号也无法使用的情况，换个账号试就可以，全被锁概率上比较小，最好还是有个备用账号。

## Win11 蓝牙

突然蓝牙找不到了，设备管理器里在“串行总线设备”里出现未知设备，卸载后再扫描，可能会恢复为蓝牙，但我遇到了不持久的情况，没过一会儿又变成未知设备了。如果不是这里的问题，就去服务等地方看。

针对蓝牙变成未知设备，还是需要物理断电，重置硬件寄存器。做法：关机，长按电源键30-40秒放电，放完了再开机就好了。

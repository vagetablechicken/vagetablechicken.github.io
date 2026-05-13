---
title: Docker Notes
date: 2026-01-06 20:00:00+0800
tags: [docker, devops]
categories: [DevOps]
---

# Docker Notes

这里记录与 Docker 相关的常用命令、配置、排错经验等。

## Network

Docker网络问题是最多的，本质上等于将调试时间从调试程序本身，转移到了调试网络配置上。

### Docker Pull Image

无论是docker pull还是docker compose pull，都有可能vpn开了也不行。目前可行的方式是，不用给docker加proxy重启，vpn开TUN保证所有流量走代理，whitelist也跑通过，全局代理应该没啥影响，但如果非全局挂了，可以切全局试试。只动vpn的配置，不动docker的配置，避免vpn不稳定的时候又把docker卡住，导致每次要重启dockerd。

### Docker Compose Network

> docker compose 和 run 不一样！
docker run 创建的容器，如果不指定网络模式，默认是 bridge 模式。
而 docker compose 创建的容器，默认是创建一个自定义网络，并将容器加入该网络。
两种启动方式的`/etc/resolv.conf`文件内容可能不一样，不要用run测试的结果来推断compose的网络问题。
{: .prompt-warning }


网络类问题，常用curl测试，但很多镜像都不带curl，可以直接用tcp。

如果需要更多，比如dig、nslookup等，最好是拉支持这些命令的镜像，先调通问题，再换回需要的镜像。可以使用：

```
docker run --rm -it --network <需要检查的网络名> busybox ping 8.8.8.8
docker run --rm -it --net container:<出问题的容器ID> nicolaka/netshoot
```

#### Docker Compose 内通信

内通信主要是Docker和系统层面的，什么sysctl配置、iptable都没用。主要是内核老了，没办法。

**Centos7一般都是3.x的内核**，自定义network的解析就不行，但可以用IP，内网172的IP也安全。但得先up，然后再配置IP，然后重启。只要不删除容器，IP是固定的，凑合用。

1. 不兼容的内核

    CentOS7 默认内核：3.10.x 系列
    你的当前版本是 3.10.0-1160.119.1.el7.x86_64
    对 Docker 26 的自定义 bridge NAT 和 embedded DNS 支持不完整
    docker0 接口可能无法自动 UP，导致所有容器出网失败
    即便修改 br_netfilter、iptables、firewalld，也无法彻底解决

2. 兼容性良好的内核

    ELRepo 主线内核（kernel-ml）
    系列版本：5.x、6.x
    完全支持 Docker 26 的 bridge 网络、自定义网络、docker0 NAT
    docker0 自动 UP → embedded DNS 可用 → 自定义网络出网正常
    RHEL / CentOS 8 内核
    系统自带内核 4.18+，也完全支持 Docker 26
    Docker 官方测试的自定义 bridge / embedded DNS 功能在这些内核上稳定
    Ubuntu 内核
    Ubuntu 20.04 / 22.04 内核 5.x / 5.15+
    Docker 26 自定义网络功能稳定，docker0 NAT 正常
    可作为替代宿主机操作系统方案

不兼容内核的特点，**可能**是ip addr show docker0 是 DOWN的，无法UP。docker0 DOWN 导致 embedded DNS 127.0.0.11 无法提供服务。

#### Docker Compose 外通信

外通信指compose容器访问外部网络，比如访问google网站，baidu网站。docker compose 默认创建一个独立网卡（172里也是独立网段），一般连不上外部网络。

**一般来讲，很多compose系统都不用往外发，但是有些服务可能是smtp往外发邮件或发slack通知，这种就需要能连外网。**

测试方法：
```bash
curl https://www.baidu.com || echo "域名连接失败"
curl https://110.242.68.4 || echo "IP连接失败"
timeout 3 bash -c '</dev/tcp/www.baidu.com/80' && echo "域名连接成功" || echo "域名连接失败"
timeout 3 bash -c '</dev/tcp/110.242.68.4/80' && echo "IP连接成功" || echo "IP连接失败"
```

调试方法：

容器里`cat /sys/class/net/eth0/iflink`找到网卡编号，然后宿主机上`ip link`找到对应的网卡，`tcpdump -i <网卡名称> port 53`抓包，看tcp请求发到哪里去了，为什么连接不上。

curl某个网站，tcpdump就有对应的连接流程，比较常见的是发了SYN，但是没有ACK，日志上就是有`[S]`标志，没有`[S.]`标志。也可能是只显示了dns通路，但并没有连上dns，可以用过ping dns ip来检查。

```
16:11:25.584111 IP 172.19.0.2.33031 > dns.google.domain: 44700+ A? www.baidu.com. (31) 16:11:25.584144 IP 172.19.0.2.33031 > dns.google.domain: 57496+ AAAA? www.baidu.com. (31)
```

无论是dns不同还是ack回不来，开启NAT伪装和转发基本都能解：

```bash
# 下面172.24换成你的docker网络子网，ip addr show 网卡名称查属于docker哪个bridge，也可以用docker network inspect <network_name>查看
# 开启NAT伪装
iptables -t nat -A POSTROUTING -s 172.24.0.0/16 ! -o docker0 -j MASQUERADE
# 开启转发
iptables -A FORWARD -s 172.24.0.0/16 -j ACCEPT
iptables -A FORWARD -d 172.24.0.0/16 -j ACCEPT

# 或是对br做
iptables -A FORWARD -i br-68c2531a5152 -j ACCEPT
iptables -A FORWARD -o br-68c2531a5152 -j ACCEPT
```

NAT类的配置，docker0都是自带的，不需要手动创建，一般是`172.18`。

容器往外访问的网通了，一般也不能自动使用到宿主机代理，`apt update` 通常是`archive.ubuntu.com (28.0.0.9)`这种地址，这是被污染的ip，也没有转到宿主机代理。除非容器必须要走代理，否则解决方式最好不要是劫持走代理，代理不稳定还要增加运维成本，直接换国内源更简单。换源方法见[apt 源更换](./2022-05-07-linux-env.md#apt-源更换)。.

### 换image源（一般不要做）

配置换image源是个非常重的操作，要重启docker。不是必须这么做，一般先考虑给pull的地址加镜像前缀，比如daocloud的，具体去搜。

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

## Windows Docker Desktop

docker desktop默认自带了ubuntu distro，但碰到几次意外退出了，最好先把wsl ubuntu装好，做个准备。而且有些image对windows文件系统支持很差，最好在wsl中mount linux文件系统。

docker desktop安装好后立马在Settings里把 WSL2 集成打开（对所有distro都加上），把上传usage数据的选项关掉。

有一说法是最好使用wsl内原生docker ce，但是这样的话开机自启动又要麻烦一些。暂时不考虑，经常desktop crash，最好还是换ubuntu系统，社区里有些issue反馈但没有回应。

## 导入导出

**千万不要用docker export/import**，大的image导入导出都慢的可怕。用save/load。

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

### Docker pull

无论是docker pull还是compose pull，都有可能vpn开了也不行。目前可行的方式是，vpn开TUN+全局代理，不用给docker加proxy重启。

### Docker compose 

> docker compose 和 run 不一样！
{: .prompt-warning }

docker run 创建的容器，如果不指定网络模式，默认是 bridge 模式。而 docker compose 创建的容器，默认是创建一个自定义网络，并将容器加入该网络。所以，两种启动方式的`/etc/resolv.conf`文件内容可能不一样，不要用run测试的结果来推断compose的网络问题。

docker compose 默认创建一个独立网卡（172里也是独立网段），一般连不上外部网络，用curl测试，没有curl的话，直接用tcp。

```bash
curl https://www.baidu.com || echo "域名连接失败"
curl https://110.242.68.4 || echo "IP连接失败"
timeout 3 bash -c '</dev/tcp/www.baidu.com/80' && echo "域名连接成功" || echo "域名连接失败"
timeout 3 bash -c '</dev/tcp/110.242.68.4/80' && echo "IP连接成功" || echo "IP连接失败"
```

容器里`cat /sys/class/net/eth0/iflink`找到网卡编号，然后宿主机上`ip link`找到对应的网卡，`tcpdump -i <网卡名称> port 53`抓包，看tcp请求发到哪里去了，为什么连接不上。

curl某个网站，tcpdump就有对应的连接流程，比较常见的是发了SYN，但是没有ACK，日志上就是有`[S]`标志，没有`[S.]`标志。

可以开启NAT伪装和转发：

```bash
# 下面172.24换成你的docker网络子网，ip addr show 网卡名称查属于docker哪个bridge，也可以用docker network inspect <network_name>查看
# 开启NAT伪装
iptables -t nat -A POSTROUTING -s 172.24.0.0/16 ! -o docker0 -j MASQUERADE
# 开启转发
iptables -A FORWARD -s 172.24.0.0/16 -j ACCEPT
iptables -A FORWARD -d 172.24.0.0/16 -j ACCEPT
```

NAT类的配置，docker0都是自带的，不需要手动创建，一般是`172.18`。

**一般来讲，很多compose系统都不用往外发，但是有些服务可能是smtp往外发邮件或发slack通知，这种就需要能连外网。**

容器往外访问的网通了，一般也不能自动使用到宿主机代理，`apt update` 通常是`archive.ubuntu.com (28.0.0.9)`这种地址，这是被污染的ip，也没有转到宿主机代理。除非容器必须要走代理，否则解决方式最好不要是劫持走代理，代理不稳定还要增加运维成本，直接换国内源更简单。换源方法见[apt 源更换](./2022-05-07-linux-env.md#apt-源更换)。

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

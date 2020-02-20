# RKE dual-stack

## 虚拟机环境规划

> 本次实验内容采用AWS EC2虚拟机，AWS EC2需要开启VPC IPv6支持。AWS EC2实例需要关闭源地址/目标地址检查。

实例名称 | IPv4 | IPv6 | 角色
------- | ---- | ---- | ---
Jason-ZW-IPv6-1 | 172.30.10.85 | 2406:da14:242:f10:2e55:ec8a:852f:e0f6 | master
Jason-ZW-IPv6-2 | 172.30.10.55 | 2406:da14:242:f10:b02:812c:46e7:80aa | node
Jason-ZW-IPv6-3 | 172.30.10.29 | 2406:da14:242:f10:143f:5b90:1544:1aa1 | node

## 集群网络规划

参数 | 值 | 描述
------- | ---- | ---- 
cluster-cidr | 192.168.0.0/24,fd05::/120 | 集群地址段
service-cluster-ip-range | 192.168.30.0/24,fd03::/120 | 服务地址段
cluster-dns | 192.168.30.10 | DNS服务地址
node-cidr-mask-size-ipv4 | 25 | 节点IPv4掩码
node-cidr-mask-size-ipv6 | 121 | 节点IPv6掩码

## 主机准备

```
# 允许ipv6转发
$ vi /etc/sysctl.conf
   net.ipv6.conf.all.disable_ipv6 = 0
   net.ipv6.conf.default.disable_ipv6 = 0
   net.ipv6.conf.lo.disable_ipv6 = 0
   net.ipv6.conf.all.forwarding=1
$ sysctl -p
# 关闭系统交换区
$ swapoff -a
# 加载ipvs内核模块
$ modprobe ip_vs
$ modprobe ip_vs_rr
$ modprobe ip_vs_wrr
$ modprobe ip_vs_sh
$ modprobe nf_conntrack_ipv4
$ modprobe nf_conntrack_ipv6
$ modprobe ipt_set
$ modprobe ipt_REJECT
# 必须目录准备
$ mkdir -p /var/lib/calico
```

## 下载安装rke

[参考官网](https://github.com/rancher/rke)

## 初始化集群

```
$ rke up --config=cluster.yml
```

## 验证集群

> 验证步骤这里不做赘述

验证场景 | 状态
------- | ----
集群状态 | ok
节点到pod连通性( ipv4 & ipv6) | ok
pod到pod连通性( ipv4 & ipv6) | ok
节点访问service( ipv4 & ipv6) | ok
访问dns( ipv4 & ipv6) | ok
验证NetworkPolicy | ok

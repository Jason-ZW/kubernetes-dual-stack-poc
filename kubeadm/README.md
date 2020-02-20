# RKE dual-stack

## 虚拟机环境规划

> 本次实验内容采用AWS EC2虚拟机，AWS EC2需要开启VPC IPv6支持。AWS EC2实例需要关闭源地址/目标地址检查。

实例名称 | IPv4 | IPv6 | 角色
------- | ---- | ---- | ---
Jason-ZW-IPv6-4 | 172.30.10.253 | 2406:da14:242:f10:7575:2952:409:e4bc | master
Jason-ZW-IPv6-5 | 172.30.10.249 | 2406:da14:242:f10:9aad:24e:e984:bc6d | node
Jason-ZW-IPv6-6 | 172.30.10.237 | 2406:da14:242:f10:ee10:dd0e:a807:9f0a | node

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
$ mkdir -p /var/lib/etcd
$ mkdir -p /var/lib/kubelet
$ mkdir -p /var/lib/calico
# 配置/etc/hosts
$ vi /etc/hosts
  127.0.0.1 localhost

  # The following lines are desirable for IPv6 capable hosts
  ::1 ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ff02::3 ip6-allhosts

  # Additional by yourself
  2406:da14:242:f10:7575:2952:409:e4bc ip-172-30-10-253
  2406:da14:242:f10:9aad:24e:e984:bc6d ip-172-30-10-249
  2406:da14:242:f10:ee10:dd0e:a807:9f0a ip-172-30-10-237

  172.30.10.253 ip-172-30-10-253
  172.30.10.249 ip-172-30-10-249
  172.30.10.237 ip-172-30-10-237
```

## 安装docker

```
$ apt-get update
$ apt-get install docker.io
```

## 安装kubeadm,kubelet,kubectl

```
$ apt-get update && apt-get install -y apt-transport-https curl
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
$ deb https://apt.kubernetes.io/ kubernetes-xenial main
$ EOF
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl
$ apt-mark hold kubelet kubeadm kubectl
```

## 初始化master节点

```
$ kubeamd init --config=kubeconfig.conf
```

## 添加node节点

```
$ kubeadm join 172.30.10.253:6443 --token yxav4s.xxxxxx     --discovery-token-ca-cert-hash sha256:xxxxxxx
```

## 添加calico网络插件

```
$ kubectl apply -f calico.yaml
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

---

layout: post
title: docker 架构
category: 技术
tags: Container
keywords: docker

---

## 简介（未完成）

* TOC
{:toc}

## Docker Daemon 调用栈分析

docker在 1.11 之 后，被拆分成了多个组件以适应 OCI 标准。拆分之后，其包括 docker daemon， containerd，containerd-shim 以及 runC。组件 containerd 负责集群节点上容器 的生命周期管理，并向上为 docker daemon 提供 gRPC 接口。

![](/public/upload/docker/docker_call_stack.png)

在一个docker 节点上执行 `ps -ef | grep docker`

```
/usr/bin/dockerd
docker-containerd  
docker-containerd-shim $containerId /var/run/docker/libcontainerd/  $containerId docker-runc
docker-containerd-shim $containerId /var/run/docker/libcontainerd/  $containerId docker-runc
...
```
1. dockerd 是docker-containerd 的父进程， docker-containerd 是n个docker-containerd-shim 的父进程。
2. Containerd 是一个 gRPC 的服务器。它会在接到 docker daemon 的远程请 求之后，新建一个线程去处理这次请求。依靠 runC 去创建容器进程。而在容器启动之后， runC 进程会退出。
3.  runC 命令，是 libcontainer 的一个简单的封装。这个工具可以 用来管理单个容器，比如容器创建，或者容器删除。

## 排查工具

1. 使用` kill -USR1 <pid>`命令发送 USR1 信号给 docker daemon， docker daemon 收到信号之后，会把其所有线程调用栈输出 到文件 /var/run/docker 文件夹里。
2. 我们可以通过` kill -SIGUSR1 <pid>` 命令来输出 containerd 的调用栈。不同的是，这次调用栈会直接输出到 messages 日志。
3. 向 kubelet 进程发送 SIGABRT 信号，golang 运行时就会帮我们输出 kubelet 进程的所有调 用栈。需要注意的是，这个操作会杀死 kubelet 进程。

## 命令行方式创建一个传统虚拟机

qemu 创建传统虚拟机流程（virtualbox是一个图形化的壳儿）

```sh
# 创建一个虚拟机镜像，大小为 8G，其中 qcow2 格式为动态分配，raw 格式为固定大小
qemu-img create -f qcow2 ubuntutest.img 8G
# 创建虚拟机（可能与下面的启动虚拟机操作重复）
qemu-system-x86_64 -enable-kvm -name ubuntutest  -m 2048 -hda ubuntutest.img -cdrom ubuntu-14.04-server-amd64.iso -boot d -vnc :19
# 在 Host 机器上创建 bridge br0
brctl addbr br0
# 将 br0 设为 up
ip link set br0 up
# 创建 tap device
tunctl -b
# 将 tap0 设为 up
ip link set tap0 up
# 将 tap0 加入到 br0 上
brctl addif br0 tap0
# 启动虚拟机, 虚拟机连接 tap0、tap0 连接 br0
qemu-system-x86_64 -enable-kvm -name ubuntutest -m 2048 -hda ubuntutest.qcow2 -vnc :19 -net nic,model=virtio -nettap,ifname=tap0,script=no,downscript=no
# ifconfig br0 192.168.57.1/24
ifconfig br0 192.168.57.1/24
# VNC 连上虚拟机，给网卡设置地址，重启虚拟机，可 ping 通 br0
# 要想访问外网，在 Host 上设置 NAT，并且 enable ip forwarding，可以 ping 通外网网关。
# sysctl -p
net.ipv4.ip_forward = 1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# 如果 DNS 没配错，可以进行 apt-get update
```

可以看到至少在网络配置这个部分，传统虚拟机跟容器区别并不大

![](/public/upload/container/container_vs_vm.png)






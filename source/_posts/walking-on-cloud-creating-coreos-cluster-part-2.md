title: 行走在云端：构建CoreOS集群 – 第二步 架设CoreOS集群
date: 2014-11-05 09:58:33
author: Lin Fan
tags:
---

CoreOS 集群的架设比架设一个传统服务器集群更加容易。一方面因为 CoreOS 使用了 Cloud-init 自动化了集群信息的配置，另一方面则是受益于 etcd 分布式存储实现的消息分发和服务器自发现机制。这些便利性正是 CoreOS 系统设计充分为集群架构考虑带来的效率提升。

## 安装 CoreOS

CoreOS 的安装方法和传统 Linux 系统有很大的不同。官方提供的[<u>系统 ISO 镜像文件</u>](https://coreos.com/docs/running-coreos/platforms/iso/)实际上只是一个 Live CD，也就免安装的试用镜像，这个 ISO 所提供的系统是不具备服务自发现和分布式消息分发的能力的。

正如[前一篇博客](http://the-1.net/walking-on-cloud-creating-coreos-cluster-part-1)所提到的，Cloud-init 通常依赖于具体平台的实现定制，将其直接在物理机上使用并不是主流的使用方法。对于这种安装方法，官方的[<u>这篇文档</u>](https://coreos.com/docs/running-coreos/bare-metal/installing-to-disk/)提供了详细的步骤，这里不再进行详细讨论。

首先来看一下 CoreOS 原生支持的平台。截止到版本 CoreOS v472，已经支持的平台如下图。

![Download](/images/CoreOS_download.png)

可以看到除去安装到本地的 Bare Metal，其余基本是针对主流的云服务平台定制的版本。这里的定制主要是 Cloud-init 等启动服务的配置，那么如何知道 CoreOS 已经支持自动化的集群部署的平台有哪些呢？我们可以从 CoreOS 源代码的 coreos-base 目录里得到答案。

链接地址：[https://github.com/coreos/coreos-overlay/tree/master/coreos-base](https://github.com/coreos/coreos-overlay/tree/master/coreos-base)
![OEM]](/images/CoreOS_Supported_Platform.png)

这些 oem 开头的目录就是平台定制的实现。其中每个目录中的 files/cloud-config.yml 文件，就是 Cloud-init 的配置文件。在每一种平台安装 CoreOS 的方式各有不同，可以从官方网站相应的页面找到相应步骤。这里我们选择其中的 Vagrant 作为演示的目标平台。

## 在 Vagrant 上部署 CoreOS 集群

使用 Vagrant 建立 CoreOS 集群可以说是最简单且经济的方式了，使用本地虚拟机构建，特别适合快速验证 CoreOS 的功能 :cool: 。

- ### 预备

需要准备的东西，包括一台连接到互联网的 Mac 或者桌面 Linux 电脑，安装好 [Git](http://git-scm.com)、[VirtualBox](https://www.virtualbox.org) 和 [Vagrant](https://www.vagrantup.com)。 

通过 Git 下载官方的 Vagrant 仓库：
```
git clone https://github.com/coreos/coreos-vagrant.git
```

下载完成后，我们接下来配置 CoreOS 集群。

- ### 配置

为了使用集群服务器的自发现功能，我们需要一个能用来唯一标识一个集群并提供集群信息的地址。CoreOS 官方提供了这个服务，当然我们也可以使用自己搭建的私有集群标识服务器，参考[<u>这个 Docker Repo</u>](https://registry.hub.docker.com/u/digitalism/discovery.etcd.io/)以及[<u>它的源码</u>](https://github.com/digitalism/discovery.etcd.io)，也可以直接使用[<u>官方的源代码</u>](https://github.com/coreos/discovery.etcd.io)。通过浏览器或命令行 curl 访问地址 https://discovery.etcd.io/new 可以得到一个新的集群标识 URL，这个 URL 会在配置 user-data 时候使用到。

```
curl https://discovery.etcd.io/new
```

进入 coreos-vagrant 目录，将 user-data.sample 和 config.rb.sample 两个文件各拷贝一份，并去掉 .sample 后缀。得到 user-data 和 config.rb 文件。

首先修改 user-data 文件，它将作为启动的配置文件提供给 CoreOS 操作系统。值得一提的是，在这个配置中，可以使用两个变量 $private_ipv4 和 $public_ipv4，它们会在实际运行的时候被自动替换为主机的真实外网 IP 和内网 IP 地址。

这里我们需要做的仅仅是将其中 <cluster-url> 所在行的 URL 替换为之前获得的集群标识 URL 地址。简单来说，所有使用了同一个标识 URL 的主机实例都会在 CoreOS 启动时自动加入到同一个集群中，这就实现了无需人工干预的集群服务器自发现。

```
#cloud-config

coreos:
  etcd:
      discovery: &lt;cluster-url&gt;
      addr: $public_ipv4:4001
      peer-addr: $public_ipv4:7001
  fleet:
      public-ip: $public_ipv4
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
```

然后修改 config.rb 文件，这里包含了 Vagrant 虚拟机的配置。通过这个文件实际上可以覆写任何 Vagrantfile 里的参数，但是目前我们只需要关注 $num_instances 和 $update_channel 这两个参数的值。
- $num_instances 表示将启动的 CoreOS 集群中需要包含主机实例的数量。
- $update_channel 表示启动的 CoreOS 实例使用的升级通道，可以是 'stable'，'beta' 或 'alpha'。
```
$num_instances=3
$update_channel='stable'
```

CoreOS 没有跨越式的版本发布，而是使用与 Arch Linux 类似的平滑的滚动升级，确保用户任何时候下载到的版本都是最新发布的系统镜像，并且从根本上解决了服务器系统在运行几年后，由于无法平滑升级而被迫重新安装的情况。此外 CoreOS 提供了 Stable、Beta 和 Alpha 三种升级通道，用于满足不同用户对系统新特性和稳定性的平衡。关于升级通道的切换，可参考[<u>官方的文档</u>](https://coreos.com/docs/cluster-management/setup/switching-channels/)。


- ### 启动


启动集群，执行：
```
vagrant up
```

查看集群运行状态，所有的集群实例都已经启动。
```
vagrant status

Current machine states:
core-01                   running (virtualbox)
core-02                   running (virtualbox)
core-03                   running (virtualbox)
```

此时，在 CoreOS 集群的内部正发生着许多故事，集群的实例之间通过自发现服务，相互认识了对方并建立了联系。它们具备了在集群中任意一个实例节点控制整个集群的能力。是的，一个功能完备的 CoreOS 服务器集群已经完全运行起来了。

## 探索 CoreOS

在下一部分，我们将会进入启动完成的 CoreOS 实例中，继续探索其中的奥秘。

## 参考文章

- [Running CoreOS on Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant/)

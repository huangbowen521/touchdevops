layout: post
title: AWS reInvent 2014
date: 2014-11-25 00:12:11
author: 黄博文
categories: [Cloud前瞻]
tags: AWS
---

亚马逊在2014年11月11-14日的拉斯维加斯举行了一年一度的re:Invent大会。在今年的大会上，亚马逊一股脑发布和更新了很多服务。现在就由我来带领大家了解一下这些新服务。

<!-- more -->


## 安全及规范相关

### AWS Key Management Service

该服务可用于管理数据加密秘钥，以及使用硬件设备来保护秘钥。它与Amazon EBS，Amazon S3及Amazon Redshift高度集成。还与CloudTrail集成，可提供所有秘钥使用情况的日志。

### AWS Config(预览)

这是个配置管理服务，可以监控和记录你对AWS资源的使用情况及配置修改历史。通过它你能轻易得到你所使用的所有AWS资源情况，以及各自的配置细节，并且能够追溯任何资源的历史配置情况。

### AWS Service Catalog

AWS Service Catalog可以让管理员创建和管理一组资源，只有包含在该组中的资源才能被指定终端用户访问。这保证了服务的隔离性和安全性，快速响应市场变化。

## 应用程序生命周期管理工具

### AWS CodeDeploy

顾名思义，就是代码部署服务。其实现将代码部署到EC2实例的自动化功能。并且它与你的基础设施紧密结合，天生具有可伸缩性，不管是部署到1台EC2还是1千EC2都得心应手。

### AWS CodeCommit

一个云端的代码托管服务。如果你的程序使用的是Git版本控制工具，可以与它很轻松的集成。CodeCommit还提供一个在线代码工具，可以通过它来浏览、编辑项目，进行项目协作。（这Y听起来有点像GitHub啊）。

### AWS CodePipeline

难道是另一款Jenkins或Go Server工具？CodePipeline提供了持续部署和发布的自动化服务。从迁入代码、构建代码、测试、部署、上线一气呵成。并且可以将第三方工具集成到发布流程中。这款工具我还是比较期待的，不知道它会给持续交付带来什么样的变化。

## 数据库服务

### Amazon RDS for Aurora

Amazon Aurora是一款兼容MySQL的关系型数据库引擎。它既具备高档商业数据库的的速度及可用性优点，也具有开源数据库的简单和成本低廉等优点。其实我并不明白在当前No SQL大行其道的情况下，为什么亚马逊会推出一款关系型数据库引擎，让我们拭目以待吧。

## 容器服务

### Amazon EC2 Container Service

最近一段时间Docker火的不行，亚马逊对Docker容器的支持也不甘落后。它直接发布了一个EC2 Container Service，内置对Docker的支持，提供了一系列API来方便对Docker容器程序的管理。

## 计算服务

### AWS Lambda

AWS Lambda是一个计算服务，提供对事件或通知的快速响应机制。看起来是一个神奇的服务。收费方式是根据请求次数以及计算执行时间，完全是按需收费。具体使用场景目前还不得而知。

## 对一些已有服务的更新

### Amazon EC2 C4 instances

这次亚马逊提供了个全新的EC2类型- C4。 C4实例基于Intel Xeon E5-2666 V3,主频高达2.9GHz，专门设计用来进行计算密集型的操作。可适用于高性能计算要求的场景，比如运行应用程序，游戏服务，Web服务等。

### Amazon EBS

亚马逊将会增加普通用途的SSD及Provisioned IOPS(SSD)卷的性能和大小。普通用途的SSD最大可达16TB，IOPS可达10000。而Provisioned IOPS(SSD)卷最大可达16TB，IOPS可达20000。当将EBS附加到优化EBS实例上时，普通用途的SSD卷MBps最高可达160，而Provisioned IOPS(SSD)可达320 MBps。


-------------------

这次大会发布的服务大部分要等到2015年初才能开始体验。中国区region就更晚了。总体来说，体现出re:Invent的服务不是很多，但也有不少值得期待。AWS作为云服务厂商的领军人物，自身实力毋庸置疑，改革、创新也是大刀阔斧。只不过我感觉它横跨IASS,PASS,SASS三大领域，包罗万象，越来越臃肿，有些服务是否画蛇添足不得而知。







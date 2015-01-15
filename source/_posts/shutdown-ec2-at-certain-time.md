---
layout: post
title: 定时关闭AWS上的EC2机器实例
date: 2014-12-26 14:19:34 +0800
author: 黄博文
comments: true
categories: Cloud 
---

最近一段时间在做一个产品从阿里云向亚马逊云中国区迁移的前期试验。亚马逊中国区并没有开放免费体验账号，使用的每一份资源都要实打实的掏钱。而为了实验我们使用时一般要启动好几台EC2实例。为了不浪费辛辛苦苦赚的钱，特写了一个脚本，在每天晚上6点将所有的EC2实例自动关闭。由于在亚马逊云中关闭的EC2实例是不收费的，只收取少量的存储费用，所以这样节省不少钱。

<!-- more -->

为了让一台机器可以值守这个任务，所以我们在AWS留一台机器用来定期执行关闭其它机器的命令。关闭EC2的命令主要是使用AWS提供了awscli来实现。

首先在这台机器上安装awscli。这台机器使用的操作系统是ubuntu 12.04，所以使用其自带的包管理器可以一键安装。

```bash

$ apt-get install awscli

```

安装完毕后需要配置aws命令行工具的认证。方式有很多，最简单的是运行`aws configure`来实现。

```bash

$ aws configure 
AWS Access Key ID [None]: YOURACCESSKEY
AWS Secret Access Key [None]: YOURSECRTKEY
Default region name [None]: cn-north-1
Default output format [None]: json

```

或者你可以在当前用户根目录下的.aws目录中配置认证信息，详情请移步[http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

接下来在当前用户根目录下创建stopinstance.sh文件，并输入以下信息.

```text stopinstance.sh

#!/bin/bash
aws ec2 stop-instances --instance-ids i-68726951 i-965ca276 i-377a620e i-d35fa133 i-fe5ca21e

```

这就是关闭指定EC2实例的命令，`--instance-ids`后面跟的就是你的EC2实例的id。

把该文件权限改为可执行。

```bash

$ chmod +x stopinstance.sh

```

最后要让该命令定时执行，那么就要借助`crontab`这个命令了。

使用`crontab -e`来编辑batch文件，在文件最后加上下行.

```text

* 18 * * * ~/stopinstance.sh >> ~/shutdown.log

```

前五个字段制定命令执行的时间。第一个是分钟，第二个是小时，第三个是每月的哪天，第四个是月份，第五个是每周的哪天。配置相当灵活。

这句话是描述了一个batch任务，在每天的下午6点执行，会执行stopinstance.sh脚本，并把日志输出到shutdown.log文件中。

注意使用crontab执行任务时一定要注意当前机器的时区和你期望执行的时间所用时区是否不同。有关crontab和cron命令的进一步使用请移步[http://www.adminschoice.com/crontab-quick-reference](http://www.adminschoice.com/crontab-quick-reference)


至此，就大功告成了。





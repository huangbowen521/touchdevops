---
layout: post
title: AWS助理架构师样题解析
date: 2014-10-22 20:33:42 +0800
comments: true
author: 黄博文
categories: AWS
tags: [AWS, Cloud] 
---

AWS 认证是对其在 AWS 平台上设计、部署和管理应用程序所需的技能和技术知识的一种认可。获得证书有助于证明您使用 AWS 的丰富经验和可信度，同时还能提升您所在的组织熟练使用基于 AWS 云服务应用的整体水平。

<!-- more -->

目前亚马逊推出了Solutions Architect,Developer和SysOps Administrator三个方向的认证。每个方向又分为Associate Level(助理级)，Professional Level（专家级）和Master Level（大师级）。当然目前只有Solutions Architect开放了Professional Level,其他层级会逐步开放中。

{% img /images/cert_roadmap_Q214_EN.png %}

最近在打算备考AWS的Solutions Architect的Associate Level。关于这个考试AWS出了一个考试样题。下载链接：[http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_solutions_architect_associate_blueprint.pdf](http://awstrainingandcertification.s3.amazonaws.com/production/AWS_certified_solutions_architect_associate_blueprint.pdf)

我把样题都做了一遍，并且都尽力找到了答案在AWS文档中的出处。以下是样题和解答。

Amazon Glacier is designed for: (Choose 2 answers)

A.active database storage.

B.infrequently accessed data.

C.data archives.

D.frequently accessed data.

E.cached session data.

答案：B和C

出处文档：[http://aws.amazon.com/glacier/?nc2=h_ls](http://aws.amazon.com/glacier/?nc2=h_ls)

>> Amazon Glacier is an extremely low-cost cloud archive storage service that provides secure and durable storage for data archiving and online backup. In order to keep costs low, Amazon Glacier is optimized for data that is infrequently accessed and for which retrieval times of several hours are suitable.

Your web application front end consists of multiple EC2 instances behind an Elastic Load Balancer. You 
configured ELB to perform health checks on these EC2 instances. If an instance fails to pass health 
checks, which statement will be true?

A.The instance is replaced automatically by the ELB.

B.The instance gets terminated automatically by the ELB.

C.The ELB stops sending traffic to the instance that failed its health check. 

D.The instance gets quarantined by the ELB for root cause analysis.

答案：C

出处文档：[http://aws.amazon.com/elasticloadbalancing/?nc2=h_ls](http://aws.amazon.com/elasticloadbalancing/?nc2=h_ls)

>> Elastic Load Balancing ensures that only healthy Amazon EC2 instances receive traffic by detecting unhealthy instances and rerouting traffic across the remaining healthy instances.

You are building a system to distribute confidential training videos to employees. Using CloudFront, what 
method could be used to serve content that is stored in S3, but not publically accessible from S3 
directly?

A.Create an Origin Access Identity (OAI) for CloudFront and grant access to the objects in your S3 
bucket to that OAI.

B.Add the CloudFront account security group “amazon-cf/amazon-cf-sg” to the appropriate S3 bucket 
policy.

C.Create an Identity and Access Management (IAM) User for CloudFront and grant access to the 
objects in your S3 bucket to that IAM User.

D.Create a S3 bucket policy that lists the CloudFront distribution ID as the Principal and the target 
bucket as the Amazon Resource Name (ARN).

答案：A

OAI介绍：[http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)

OAI基本上就是专为这个场景引入的。


Which of the following will occur when an EC2 instance in a VPC (Virtual Private Cloud) with an 
associated Elastic IP is stopped and started? (Choose 2 answers)

A.The Elastic IP will be dissociated from the instance

B.All data on instance-store devices will be lost

C.All data on EBS (Elastic Block Store) devices will be lost

D.The ENI (Elastic Network Interface) is detached

E.The underlying host for the instance is changed

答案：B E

这个题难度比较高。可以用排除法，A，C，D肯定不能选，B是对的，那么剩下一个答案只有E了啊。


In the basic monitoring package for EC2, Amazon CloudWatch provides the following metrics:

A.web server visible metrics such as number failed transaction 
requests

B.operating system visible metrics such as memory utilization

C.database visible metrics such as number of connections

D.hypervisor visible metrics such as CPU utilization

答案：D

注意题干说的是basic monitoring,A,B,C肯定不对。具体支持的监控指标可见[http://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/DeveloperGuide/ec2-metricscollected.html#ec2-metrics](http://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/DeveloperGuide/ec2-metricscollected.html#ec2-metrics)。D是唯一接近正确答案的，但是我对hypervisor了解不多，有些迷惑人。

Which is an operational process performed by AWS for data security?

A.AES-256 encryption of data stored on any shared storage device

B.Decommissioning of storage devices using industry-standard practices

C.Background virus scans of EBS volumes and EBS snapshots

D.Replication of data across multiple AWS Regions

E.Secure wiping of EBS data when an EBS volume is unmounted

答案：B

具体可以查看 was security whitepaper: [https://media.amazonwebservices.com/pdf/AWS_Security_Whitepaper.pdf](https://media.amazonwebservices.com/pdf/AWS_Security_Whitepaper.pdf)

Storage Device Decommissioning 小节里面有这么一句话：

>> All decommissioned magnetic storage devices are
degaussed and physically destroyed in accordance with industry-standard practices.


To protect S3 data from both accidental deletion and accidental overwriting, you should:

A.enable S3 versioning on the bucket

B.access S3 data using only signed URLs 

C.disable S3 delete using an IAM bucket policy 

D.enable S3 Reduced Redundancy Storage

E.enable Multi-Factor Authentication (MFA) protected access

答案：A

出处文档:[http://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html](http://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html)

>> Versioning-enabled buckets enable you to recover objects from accidental deletion or overwrite. 


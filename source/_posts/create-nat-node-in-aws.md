layout: post
title: 在AWS中创建NAT节点
date: 2015-01-17 23:23:15 +0800
author: 黄博文
comments: true
categories: AWS
tags: [AWS, CloudFormation, NAT]
---


NAT, Network Address Translation,即网络地址转换。当内部网络的主机想要访问外网，但是又不想直接暴露给公网，可以通过NAT节点来访问外网。这样做有两个好处，第一是内网的主机无需拥有公网IP就可访问网络（NAT节点需要公网IP），节约了公网IP；第二是内网的主机由于没有公网IP，所以公网的电脑无法访问到它，这样就可以隐藏自己。一个很经典的示例是假如你有一台数据库服务器放置在内网中，为在同一个内网中的web服务器提供数据服务，为了安全性考虑你不会把它直接暴露在公网中。但是数据库服务器有时候自己是需要访问公网的，比如需要升级数据库服务器中的某些软件等。采用NAT方案可以很好的解决这个问题。

<!-- more -->

下图是NAT节点的功能示意图。


{% img /images/natnode.png %}


一些路由器或者装有特定软件的主机都可以作为NAT节点。在AWS中如果你想创建一个NAT节点的话那是非常的方便，因为AWS直接提供了预装了NAT软件的AMI，你只需直接使用该AMI在你的公共子网中实例化一台机器，并进行相应的配置即可。


下面的图展示了在AWS中的一个经典的VPC架构。该VPC里面建立了两个子网，一个是公共子网，通过Intenet Geteway和公网连接；一个是私有子网，无法直接访问公网。然后在公共子网中建立了一个EC2机器，使用的是AWS提供的具有NAT功能的AMI，并为它分配了一个弹性IP，这样该EC2就是一个NAT节点。在私有子网的所有机器都具有了通过该NAT节点访问外网的能力。

{% img /images/vpcandnat.png %}

为了创建这样一套网络及机器，最简便的方式当然是使用AWS提供的CloudFormation了。如果不了解CloudFormation，可以看我以前写过的一篇文章 [《亚马逊云服务之CloudFormation》](http://www.huangbowen.net/blog/2013/10/23/aws-cloudformation/)。下面展示的是创建该整个VPC的CloudFormation脚本。


```json

{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "Setup a vpc, which contains two subnets and one NAT machine",
  "Parameters": {
    "KeyName": {
      "Description": "Name of and existing EC2 KeyPair to enable SSH access to the instance",
      "Type": "String"
    },
    "VpcCidr": {
      "Description": "CIDR address for the VPC to be created.",
      "Type": "String",
      "Default": "10.2.0.0/16"
    },
    "AnyCidr": {
      "Description": "CIDR address for Any Where.",
      "Type": "String",
      "Default": "0.0.0.0/0"
    },
    "AvailabilityZone1": {
      "Description": "First AZ.",
      "Type": "String",
      "Default": "cn-north-1a"
    },
    "PublicSubnetCidr": {
      "Description": "Address range for a public subnet to be created in AZ1.",
      "Type": "String",
      "Default": "10.2.1.0/24"
    },
    "PrivateSubnetCidr": {
      "Description": "Address range for private subnet.",
      "Type": "String",
      "Default": "10.2.2.0/24"
    },
    "NATInstanceType": {
      "Description": "Instance type for NAT",
      "Type": "String",
      "Default": "t1.micro"
    }
  },
  "Mappings": {
    "AWSNATAMI": {
      "cn-north-1": {
        "AMI": "ami-eab220d3"
      }
    }
  },
  "Resources": {
    "VPC": {
      "Type": "AWS::EC2::VPC",
      "Properties": {
        "CidrBlock": {
          "Ref": "VpcCidr"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "VPC"
          }
        ]
      }
    },
    "InternetGateWay": {
      "Type": "AWS::EC2::InternetGateway",
      "Properties": {
        "Tags": [
          {
            "Key": "Name",
            "Value": "INTERNET_GATEWAY"
          }
        ]
      }
    },
    "GatewayToInternet": {
      "Type": "AWS::EC2::VPCGatewayAttachment",
      "Properties": {
        "InternetGatewayId": {
          "Ref": "InternetGateWay"
        },
        "VpcId": {
          "Ref": "VPC"
        }
      }
    },
    "PublicSubnet": {
      "Type": "AWS::EC2::Subnet",
      "Properties": {
        "CidrBlock": {
          "Ref": "PublicSubnetCidr"
        },
        "AvailabilityZone": {
          "Ref": "AvailabilityZone1"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "PUBLIC_SUBNET"
          }
        ],
        "VpcId": {
          "Ref": "VPC"
        }
      }
    },
    "PrivateSubnet": {
      "Type": "AWS::EC2::Subnet",
      "Properties": {
        "CidrBlock": {
          "Ref": "PrivateSubnetCidr"
        },
        "AvailabilityZone": {
          "Ref": "AvailabilityZone1"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "PRIVATE_SUBNET"
          }
        ],
        "VpcId": {
          "Ref": "VPC"
        }
      }
    },
    "DefaultSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription": "Default Instance SecurityGroup",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "-1",
            "CidrIp": {
              "Ref": "VpcCidr"
            }
          }
        ],
        "Tags": [
          {
            "Key": "Name",
            "Value": "DEFAULT_SECURITY_GROUP"
          }
        ],
        "VpcId": {
          "Ref": "VPC"
        }
      }
    },
    "PublicRouteTable": {
      "Type": "AWS::EC2::RouteTable",
      "Properties": {
        "VpcId": {
          "Ref": "VPC"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "PUBLIC_ROUTE_TABLE"
          }
        ]
      }
    },
    "PublicRoute": {
      "Type": "AWS::EC2::Route",
      "Properties": {
        "DestinationCidrBlock": {
          "Ref": "AnyCidr"
        },
        "GatewayId": {
          "Ref": "InternetGateWay"
        },
        "RouteTableId": {
          "Ref": "PublicRouteTable"
        }
      }
    },
    "PublicSubnetRouteTableAssociation": {
      "Type": "AWS::EC2::SubnetRouteTableAssociation",
      "Properties": {
        "RouteTableId": {
          "Ref": "PublicRouteTable"
        },
        "SubnetId": {
          "Ref": "PublicSubnet"
        }
      }
    },
    "NATEIP": {
      "Type": "AWS::EC2::EIP",
      "Properties": {
        "InstanceId": {
          "Ref": "NATInstance"
        }
      }
    },
    "NATInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "InstanceType": {
          "Ref": "NATInstanceType"
        },
        "KeyName": {
          "Ref": "KeyName"
        },
        "SubnetId": {
          "Ref": "PublicSubnet"
        },
        "SourceDestCheck": false,
        "ImageId": {
          "Fn::FindInMap": [
            "AWSNATAMI",
            {
              "Ref": "AWS::Region"
            },
            "AMI"
          ]
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "NAT"
          }
        ],
        "SecurityGroupIds": [
          {
            "Ref": "NATSecurityGroup"
          }
        ]
      }
    },
    "PrivateSubnetRouteTable": {
      "Type": "AWS::EC2::RouteTable",
      "Properties": {
        "VpcId": {
          "Ref": "VPC"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "PRIVATE_SUBNET_ROUTE_TABLE"
          }
        ]
      }
    },
    "PrivateSubnetRoute": {
      "Type": "AWS::EC2::Route",
      "Properties": {
        "DestinationCidrBlock": {
          "Ref": "AnyCidr"
        },
        "InstanceId": {
          "Ref": "NATInstance"
        },
        "RouteTableId": {
          "Ref": "PrivateSubnetRouteTable"
        }
      }
    },
    "PrivateSubnetRouteTableAssociation": {
      "Type": "AWS::EC2::SubnetRouteTableAssociation",
      "Properties": {
        "RouteTableId": {
          "Ref": "PrivateSubnetRouteTable"
        },
        "SubnetId": {
          "Ref": "PrivateSubnet"
        }
      }
    },
    "NATSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription": "NAT Instance SecurityGroup",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "-1",
            "CidrIp": {
              "Ref": "VpcCidr"
            }
          }
        ],
        "Tags": [
          {
            "Key": "Name",
            "Value": "NAT_SECURITY_GROUP"
          }
        ],
        "VpcId": {
          "Ref": "VPC"
        }
      }
    }
  },
  "Outputs": {
    "VPCId": {
      "Description": "VPC id",
      "Value": {
        "Ref": "VPC"
      }
    },
    "PublicSubnetId": {
      "Description": "public subnet id",
      "Value": {
        "Ref": "PublicSubnet"
      }
    },
    "PrivateSubnetId": {
      "Description": "private subnet id",
      "Value": {
        "Ref": "PrivateSubnet"
      }
    },
    "NATSecurityGroupId": {
      "Description": "NAT SG id",
      "Value": {
        "Ref": "NATSecurityGroup"
      }
    },
    "NATEIP": {
      "Description": "NAT Server EIP.",
      "Value": {
        "Ref": "NATEIP"
      }
    }
  }
}


```

你可以通过AWS提供的图形化界面AWS Management Console来使用该CloudFormation脚本，也可以通过AWS CLI来使用。使用以上的CloudFormation脚本创建的VPC可以一键创建你AWS中的基础网络架构。从此再也不用为配置网络发愁了。


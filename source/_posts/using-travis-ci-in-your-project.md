layout: post
title: 持续集成之道：在你的开源项目中使用Travis CI
date: 2014-10-09 09:20:24
author: 黄博文
categories: [持续集成与部署]
tags: [Travis, CI] 
---

自从接触并践行了敏捷的一些实践之后，便深深的喜欢上了敏捷。尤其是测试自动化和持续集成这两个实践，可以显著的提高软件的质量和集成效率，实时检测项目健康度，使团队成员对项目保持充足的信心。

但是对于个人项目而言，虽然测试自动化好实现，但是要实现持续集成还是稍有难度。因为持续集成需要搭建一个集成服务器，并建立某种反馈机制。而大多数人来说并没有自己的独立服务器，并且配置也极为繁琐。

<!-- more -->

不过不用怕，现在已经进入了云时代。 [Travis CI]为我们提供了免费的集成服务器，让我们省却了自己搭建集成服务器的烦恼。

[Travis CI]的官网介绍是: **A hosted continuous integration service for the open source community.** 表明它主要是给开源社区提供持续集成服务。其与github这个全球最火爆的代码托管网站高度集成，可以很方便的为github中的项目建立持续集成服务。

它不仅支持多种语言，而且支持同时在多个运行环境中运行build，能全方位的测试你的程序。

下面就介绍下如何将[Travis CI]与自己在github上的某个repository集成。（这里以我自己的repository <https://github.com/huangbowen521/SpringMessageSpike> 为例。 ）

1. 使Travis CI通过github OAuth认证。

	点击<https://travis-ci.org/>右上角的`Sign in with GitHub`按钮，输入自己的github账号和密码，并允许Travis CI的认证。

2. 激活GitHub Service Hook。 

	GitHub给用户提供了一个Service Hook接口,只要用户对host在github上的repository作用了一些action(比如push，pull)，就会触发相应的Service Hook。而[Travis CI]正是基于这个原理来trigger你的build。当你发起一个push操作时，就会trigger [Travis CI]的服务。

	设置方法是访问[Travis CI]的[profile](https://travis-ci.org/profile)，选择相应的repository打开Service Hook开关。

	{% img /images/TravisProfile.png 600 %}

	然后登陆你的github，访问具体的repository的Service Hook页面，确保设置了Travis CI Hook的github name和travis token。

	{% img /images/ServiceHook.png 600 %}

3. 给repository配置.travis.yml文件。该文件需要放置在repository的根目录下。

	.travis.yml文件是一个相当重要的文件，里面需要配置你所使用的语言、运行环境、构建工具、构建脚本、通知方式等。最重要的是设置语言，其它的都有相应的默认值。

	这是为我的[SpringMessageSpike]设置的.travis.yml文件。由于我的项目中使用了maven作为构建工具，而[Travis CI]对java语言设置的默认构建工具就是maven，所以无需在文件中显式指定。

```yaml 
language: java
jdk:
  - oraclejdk7
  - openjdk7
  - openjdk6

```

你可以使用一个travis-lint来检查你的yml文件是否是有效的。他是ruby写的一个gem，需要ruby的运行环境。安装方式是在terminal下`gem install travis-lint`。你只需要在你的repository根目录下运行`travis-lint`即可进行检查。

想要更进一步的关于.travis.yml的配置请参见：<http://about.travis-ci.org/docs/user/build-configuration/> 

只要这三步就完成了配置。现在发起一个push就可以trigger你在[Travis CI]的build。
这时候登陆[Travis CI]可以看到你的Build的状态和日志。

{% img /images/BuildInfo.png 600 %}

你可以在respository的README.md文件中加入build状态图标。方法是在在该文件中加入
`[![Build Status](https://travis-ci.org/[YOUR_GITHUB_USERNAME]/[YOUR_PROJECT_NAME].png)](https://travis-ci.org/[YOUR_GITHUB_USERNAME]/[YOUR_PROJECT_NAME])`即可。

{% img /images/BuildImage.png 600 %}

总体来说[Travis CI]是一个轻量级、可高度定制化的免费的持续集成服务。但我觉得还是有几个缺点:

1. 运行build需要大量的准备，耗时较长。

2. 作为免费的服务，不支持build时间超过20分钟的项目。

3. 主站访问速度略慢。

[Travis CI]: (https://travis-ci.org/)
[SpringMessageSpike]: (https://github.com/huangbowen521/SpringMessageSpike)

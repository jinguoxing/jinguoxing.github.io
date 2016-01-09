---
layout: post
title: "CentOS安装最新版的git"
description: ""
categories: git
tags: 
    - git
    - centos
---
####官方给出的安装参考
   
  `$ yum install git`

<p>新版的CentOS后续好像也支持了这个功能，但是这种方法虽然简单，但是一般仓库里的版本更新不及时，比如 CentOS 仓库中的 git 最新版是1.7.1，但是 git 官方已经到2.x 的版本了。对于想要获取最新git的系统，只能下rpm包或者用源码。</p>

步骤如下：

1. 下载编译工具

    `$ yum groupinstall "Development Tools"`

2. 下载依赖包

	`$ yum install zlib-devel perl-ExtUtils-MakeMaker asciidoc xmlto openssl-devel`

3. 下载 git 最新版本的源代码

    `$ wget http://www.codemonkey.org.uk/projects/git-snapshots/git/git-latest.tar.gz`



4. 解压源文件

    `$ tar -zxvf git-latest.tar.gz`

5. 编译安装

    `$ autoconf`

 	`$ ./configure`

 	`$ make -jn && make -jn install`

	其中make -j n中的n为指定线程数，对于多核处理器这样可以加快编译安装的速度

6. 添加link

	`$ ln -s /usr/local/bin/git /usr/bin/`

	这一步对于原本系统中有旧版git的系统很重要，会报告Link已存在，此时要删除原来的Link即/usr/bin/git，再执行第六步。

7. 检查版本号

	`$ git --version`

	对于系统中存在老版的git的系统，安装了新git后用git --version查看仍然显示为老版就是因为忽略了第六步，这是很重要的！ 

	------
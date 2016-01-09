---
layout: post
title: "使用rake自动生成文件"
description: "使用rake自动生成一些模板"
category: [jekyll,rake]
date:  2015-06-01
tags: 
    -jekyll
---

####rake的安装
`sudo gem install rake`

####page或post的创建及发布
创建post文章

    rake post title="文章标题" date="日期"

会自动创建一个具有合适文件名和YAML Front Matter的文件(使用时将"文章标题"替换成你要创建的文章的标题，将"日期"换成你要的时间，如果date缺少将是当前的日期)

创建Page页面
    
    rake page name="页面名称.md"   
    或者
	rake page name="pages/页面名称.md"  



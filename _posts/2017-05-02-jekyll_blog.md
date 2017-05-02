---
layout: post
title: Xcode8代码格式化插件
categories: [jekyll]
tags: [jekyll]
date: 2016-05-02 15:00:00
comments: true
---

Jekyll时一款简单的静态博客生成引擎，比较适合技术博客。当然其他的博客引擎比较著名的有Wordpress 、octopress、hexo等，这里不做讨论。Jekyll有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。
可以配合第三方服务例如： Disqus（评论）、多说(评论) 以及分享 等等扩展功能，可以绑定自己的域名。


官方上的教程快速指南如下
~ $ gem install jekyll
~ $ jekyll new myblog
~ $ cd myblog
~/myblog $ jekyll serve
# => Now browse to http://localhost:4000
 
-------------
使用jekyll时基于ruby开发的，所以要安装ruby包。git环境以及包管理器RubyGems。如果你是Mac用户，需要安装Xcode和Command Line Tool，因为本人时Mac，所以下面以Mac为例说明配置过程。
 
1.首先查看自己电脑ruby版本信息，确认是否安装ruby
$ gem ruby -v
 
2.安装Jekyll
安装Jekyll最好的方式是使用RubyGems，记住新版mac系统要加上sudo
$ sudo install jekyll
如果提示没有权限，这是因为OSX 10.11 Rootless，防止对系统路径的修改，需要按Command + R,进入恢复模式，设置 csrutil 为disable
$ csrutil disable
当前安装完成之后，可以再把csrutil设置为enable
 
3.紧接着可以 创建博客
$ jekyll new myBlog
着知性这一步骤的时候，我这边提示Dependency Error: Yikes! It looks like you don't have bundler...， 但是查看本地，发现myBlog文件夹已经生成，但是实际上执行$ jekyll serve发现本地服务无法启动，所以需要先安装 bundler   命令: gem install bundler
 
4.启动服务
$ jekyll serve  
#Server running... press ctrl-c to stop. （服务已经启动啦！）
服务已经启动，此时打开http://localhost:4000,出现如下界面，说明jekyll已经配置完成了

![ ]({{ site.url }}/img/blogImg/jekyll_blog.png)   


//觉得默认主题不好看，也可以搜下好看的主题或者自己实现.

//完整项目代码:https://github.com/maxzhang123/codeFormat
//转载请注明出处，谢谢



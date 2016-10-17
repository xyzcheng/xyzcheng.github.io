---
layout: post
title:  "jekyll的使用"
date:   2016-10-17 18:50:20
categories: Tools
tags: jekyll
---

* content
{:toc}

jekyll是一款免费的blog生成工具，相比于workpress，jekyll不需要数据库的支持，它能够将文本文件转换成静态网页供我们在浏览器阅读。同时，github也支持jekyll的部署，这样我们就可以很方便的搭建自己的博客网站了。

下面我们就来介绍下如何使用 jekyll + github + markdown编辑工具 来创建自己的blog网站：





## 1 jekyll介绍
jekyll是一款免费的blog生成工具，相比于workpress，jekyll不需要数据库的支持，它能够将文本文件转换成静态网页供我们在浏览器阅读。同时，github也支持jekyll的部署，这样我们就可以很方便的搭建自己的博客网站了。

下面我们就来介绍下如何使用 jekyll + github + markdown编辑工具 来创建自己的blog网站：

## 2 jekyll安装
注：这里的安装环境是windows，如果是Linux或者mac可以自行百度如何安装

### 2.1 安装ruby和devkit

windows下使用rubyinstaller安装，下载地址为 [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/ "http://rubyinstaller.org/downloads/")

请同时下载最新版的ruby   
![ruby](http://115.29.144.199/proj/imgs/blog/blog-jekyll-1.png)  
和devkit  
![devkit](http://115.29.144.199/proj/imgs/blog/blog-jekyll-2.png)

在安装ruby的第三步时注意勾上 “Add Ruby excutables to your PATH” 
 
![add path](http://115.29.144.199/proj/imgs/blog/blog-jekyll-3.png)

devkit直接解压到指定目录即可

1. 检验ruby是否安装成功  
	`$ ruby -v`   
	ruby 2.3.1p112 (2016-04-26 revision 54768) [x64-mingw32]
2. 创建 config.yml 文件
注意：cd到 devkit 解压目录    
	`$ cd devkit`    
	`$ ruby dk.rb init`
3. 查看config.yml文件，如果未添加ruby安装路径，则需要添加  
	`$ vi config.yml`  

![config](http://115.29.144.199/proj/imgs/blog/blog-jekyll-4.png)  

4. 安装和审查  
	`$ ruby dk.rb install`  
	`$ ruby dk.rb review`

### 2.2 安装jekyll
$ gem install jekyll  

上述命令可能会出现以下错误：  
	ERROR:  Could not find a valid gem 'jekyll' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)  
原因是国内屏蔽了rubygems的站点服务器，你懂得。  

解决办法：  
	`$ gem sources --remove https://rubygems.org/`   
	`$ gem sources -a http://gems.ruby-china.org/`  
（网上有说指定https://ruby.taobao.org的，亲测，不好使，据说是因为这个站点在维修中...）
	`$ gem sources -l`   
查看是否已经更换：  
*** CURRENT SOURCES ***  
http://gems.ruby-china.org/

再次使用 gem install jekyll 稍等片刻即可顺利完成安装

## 3 jekyll的使用
	$ jekyll -v
jekyll 3.3.0  

### 3.1 创建一个新的blog    
	$ jekyll new blog`  

可能会出现以下错误：  
	Dependency Error: Yikes! It looks like you don't have bundler or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- bundler' If you run into trouble, you can find helpful resources at http://jekyllrb.com/help/!
	jekyll 3.3.0 | Error:  bundler  

原因在于没有安装bundler，解决办法：  
	`$ gem install bundler`  

删除原blog后重新执行：  
	`$ jekyll new blog`  

不出问题看似应该可以正常执行了，但注意是否发生以下错误：  
	Gem::RemoteFetcher::FetchError: SSL_connect returned=1 errno=0 state=SSLv3 read
	server certificate B: certificate verify failed
	(https://rubygems.org/gems/minima-2.0.0.gem)  
	An error occurred while installing minima (2.0.0), and Bundler cannot continue.  
	Make sure that `gem install minima -v '2.0.0'` succeeds before bundling.  
  
原因是没有正确安装minima，解决方法：  
	`$ gem install minima`

重复执行：  
	`$ jekyll new blog`  

可能还会出现上面类似的错误，只要把对应的依赖安装就好了，最后出现：  
	Bundle complete! 3 Gemfile dependencies, 20 gems now installed.  
	Use `bundle show [gemname]` to see where a bundled gem is installed.

### 3.2 本地查看新创建的简单blog网站
1. cd到刚创建的blog目录  
	`$ cd blog`
2. 发布到localhost  
	`$ jekyll s`
3. 使用浏览器查看  
用浏览器打开localhost:4000/即可查看  

![ruby](http://115.29.144.199/proj/imgs/blog/blog-jekyll-5.png)

## 4 jekyll主题模板
jekyll有很多开源的主题模板，你可以到 [http://jekyllthemes.org/](http://jekyllthemes.org/ "http://jekyllthemes.org/") 进行查看，选择适合自己的主题，进入到对应的github项目中，clone或者直接下载到本地，然后到对应目录下使用：  
	`$ jekyll s`  
类似于自己创建的blog使用方式 

## 5 配合github page使用
github支持jekyll的部署，这样，只要我们将生成的blog目录放到github上并加以配置，就能够像有个人网站那么使用了。

### 5.1 创建github.io仓库
1. 登录到你的github，如 https://github.com/username
2. 创建仓库: username.github.io
3. 到username.github.io的settings中找到Github Pages
4. 选择Launch automatic page generator
5. 到模板界面随便选择一个模板点击确定即可
6. 
6. 稍等几分钟，浏览器打开https://username.github.io就可以查看你选择的github page了

### 5.2 本地测试刚选择的github page
1. 将刚刚创建完的仓库clone到本地
2. cd到此项目，使用 `$ jekyll s` 发布，打开localhost:4000即可查看你选择的主题

### 5.3 替换到开源的jekyll theme
1. 将项目中的除ReadMe.md外全部删除
2. 将4（“4 jekyll主题模板”）中你选择的开源模板项目复制到刚自己创建的项目中
3. 再次使用`$ jekyll s` 发布，这样就可以看到你选择的别人的模板blog

### 5.4 push到github远程分支
1. `$ git add .`
2. `$ git commit -m 'commit message'`
3. `$ git push origin master`
4. 正确执行完上述操作后，打开https://username.github.io就可以查看你刚发布的blog网站了

### 5.5 jekyll如何发布自己的文章
打开你创建的blog项目，可以看到有_posts目录，这里面存放的文件最终会被jekyll转换成web静态网页，你可以在这里尝试创建一个文件测试

## 6 配合markdown工具

markdown非常适合于blog写作，windows下推荐使用 markdownpad，下载地址为： [http://markdownpad.com/](http://markdownpad.com/ "http://markdownpad.com/")

关于markdownpad的使用这就不详细说明了，你将它当做是一款文本编辑工具即可。  

利用markdownpad编辑的.md文件，将其放在_posts中，重新`jekyll s`即可查看新增的博客条目

具体的jekyll主题可能有不同的操作方式，你可以查看对应的说明使用。

## 7 推荐文章

[如何快速给自己构建一个温馨的"家"——用Jekyll搭建静态博客](http://www.jianshu.com/p/9a6bc31d329d)  

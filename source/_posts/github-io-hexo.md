---
title: hexo+github搭建个人博客
date: 2017-02-13 18:20:48
tags: 
- github
- github.io
- hexo
categories: 
- tools
---

近来发现一种很方便的搭建个人博客的方法，不需要云服务器，使用**hexo**+**github**实现，完全免费，效果还不错。此站是博主搭建出来的效果。

## 1.准备GitHub
### 1.1 新建gitbub库
如果没有，去[github](https://github.com)申请一个；然后新建一个Repository,名字为yourname.github.io; 
### 1.2 clone gitbub库
clone 到本地机器
```
git clone https://github.com/yourId/yourname.github.io.git
```

## 2.hexo使用
### 2.1 hexo 安装
使用前提，本机已经安装git 和 nodejs
[教程参见](https://hexo.io/docs/)
### 2.2 生成博客
-  初始化目录
```
$ hexo init <folder>
$ cd <folder>
$ npm install
```
- 新建文档
```
$ hexo new [layout] <title>
```

参数| 描述
---|---
layout| 默认使用post，存放位置为：~/你的文档根目录/source/_posts
title | 文档名

- 编辑文档   
文档编辑，编辑使用[markdown](http://wowubuntu.com/markdown/)语法,不赘述；
需要注意，front-master一般采用这种格式：
```
---
title: mvn tutorial
date: 2017-02-13 13:22:33
tags: 
- maven 
- learn
categories:
- tools
---
```
参数| 描述
---|---
tags| 标签
categories | 文档分类
- 生成网站
```
$ hexo generate
```
- 部署网站
```
$ hexo deploy
```
==注意：==
部署前设置部署方式和目标url;  
部署之前可以使用命令：**hexo server** 本地运行看一下。
```
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]
```
参数| 描述
---|---
repo| 库（Repository）地址
branch | 分支名称。如果您使用的是 GitHub 或 GitCafe 的话，程序会尝试自动检测。
message |	自定义提交信息 (默认为 update+提交时间)

至此完成，你可以访问自己的github.io地址：yourGithubId.github.io

### 2.3 主题使用
好的博客需要一个好看的主题（楼主使用的是Next主题，**关于标签和分类的index.html文件生成问题**需要[参考](http://theme-next.iissnan.com/theme-settings.html)   ），[主题地址](https://hexo.io/themes/),如何装饰呢？

点击相关主题，找到此主题相关的github地址，举例：某主题地址是
https://github.com/levblanc/hexo-theme-aero-dual
```
$ git clone https://github.com/levblanc/hexo-theme-aero-dual.git themes/aero-dual
```
clone 主题项目到你的博客的themes目录下，便安装完成。
更改你博客的目录下的  _config.yml 文件中theme的值为aero-dual；然后hexo server 查看是否已经装饰上主题。



  
	
	

 
 







  
	
	

 
 




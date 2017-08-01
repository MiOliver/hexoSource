---
title: Hexo 图片引用解决方案
date: 2017-02-17 13:08:56
tags: 
- 图片引用
- tags
- plugin
- hexo
categories: 
- tools
---


记录几种hexo使用图片的方式：
## 传统方式
采用html标签方式引用：
```
<img src="/2016/3/9/本地图片测试/logo.jpg" alt="logo">
```
或者使用markdown语法：
```
![image](/img/imageName.png)
```
## hexo 标签方式
1.首先确认_config.yml 中有 post_asset_folder:true。  
Hexo 提供了一种更方便管理 Asset 的设定：post_asset_folder
当您设置post_asset_folder为true参数后，在建立文件时，Hexo
会自动建立一个与文章同名的文件夹，您可以把与该文章相关的所有资源都放到那个文件夹，可以更方便的使用资源。

```
{% asset_img test.png image_title %}
```
asset_img 表示要引用图片, test.png是资源名称, 后面的是图片显示的标题

然后执行 **hexo generate** (或者hexo g)  
就会将资源拷贝到和生成的文章相同的目录下,这样就可以了.在本地查看会显示不正常,但是部署到github上就显示正常了.

## 插件推荐
回到hexo的主目录下执行
```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```
完成安装后用hexo新建文章的时候会发现_posts目录下面会多出一个和文章名字一样的文件夹。图片就可以放在文件夹下面。
```
本地图片测试
├── apppicker.jpg
├── logo.jpg
└── rules.jpg
本地图片测试.md
```
这样的目录结构（目录名和文章名一致），只要使用
```
![logo](本地图片测试/logo.jpg)
```
就可以插入图片。其中[]里面写相关图片标题。
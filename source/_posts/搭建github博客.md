---
title: 搭建github博客
date: 2019-02-06 19:26:40
categories:
    - 环境
tags: 
    - github博客
    - hexo
---

## 背景

1. 很多博客网站的广告很多，或者不够自由。
2. 博客相关的代码存放不方便
3. 自己购买域名需要花钱
4. github提供pages功能，用户可以免费创建博客空间。



## hexo

hexo是一种静态网页开发平台，基于node.js等。开发人员提供markdown网页，hexo基于themes等资源生成静态网页并发布成网站。

hexo还支持git 方式发布，用户可直接将静态网页等资源直接发布到github上，基于github pages功能，我们可以直接在互联网上浏览相应网页。



## 搭建过程

网上有很多相关文章，这里简单说一下大概步骤。

1. 安装node.js，基于node.js安装hexo。
2. 安装git，代码的提交和网站的发布都需要git。
3. 在github上创建一个仓库，并克隆到本地。
4. 在一个空文件夹blog中执行hexo init，初始化网站资源。
5. 将blog中的文件拷贝到git路径下，执行hexo d -g发布到github上的master分支上。
6. 创建dev分支，删除.gitignore文件，将所有文件都提交到远程dev分支中。后续在dev分支上修改源码和文件，使用hexo d -g将网站发布到github的master分支上。
7. 在hexo 的themes上找到喜欢的主题，克隆到themes中，修改_config.xml中的theme。
8. 使用hexo new 创建新的文章，注意添加categories和tags，并严格遵守文件格式， - 后面只有一个英文空格。
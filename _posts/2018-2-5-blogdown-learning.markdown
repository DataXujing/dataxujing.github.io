---
layout: post
title: "blogdown用R做自己的网站"
img: wu1.jpg 
date: 2018-2-5 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [R,blogdown]
---

**徐静**

最近一个月比较忙，一直没有时间更新自己的博客，各方面的原因都有，今天我给大家介绍一个比较好玩的R包：blogdown，这个包是谢益辉写的，主要用来生成静态的网页博客，通过Gitpage,Netlify等托管到网上。我的这个博客使用jekyll搭建的，其实在搭建这个博客时我就纠结是用jekyll还是blogdown去完成，最终选择的jekyll。今天在地铁上突然有个想法，想搭建一个自己家庭生活的博客，记录家庭生活的的点点滴滴，毕竟程序员，数据分析师，算法工程师还是要有自己的生活的，花了1个小时用R搭建了自己的生活博客。下面就把blogdown搭建博客的流程介绍一下。

blogdown非常简单易用，对于数据分析师而言，通过这种方式搭建自己的技术博客或共享分析报告都是非常OK的选择，整体来说他要比shiny简单很多。它主要依托于hugo和Rstudio及git,所以在使用blogdown前除了安装这个吧之外还要安装hugo和Rstudio及git安装好。


### rstudio配置

大家尽量安装最新版本的RStudio, v1.1.383以后的版本才支持我的如下操作。安装好上述软件后，需要对rstudio进行一下简单配置：

+ Tools -> Global Options -> Sweave -> Weave Rnw files using:knitr
+ Tools -> Global Options -> Sweave -> Typeset LaTex into PDF using:XeLaTeX
+ Tools -> Global Options -> Git/SVN -> Git executable:


### 安装blogdown和hugo

安装blogdown：

```r
install.packages('blogdown')
```

安装hugo

```r
blogdown::install_hugo()
```

如果hugo安装不成功，可以自行下载安装

```r
# 注意这里是三个冒号
blogdown:::install_hugo_bin("D:/hugo.exe")
```

同时可以通过一些命令查看hugo的版本。

### 用github创建repository

该部分比较简单略过

### blogdown建站

谢大神在blogdown中指定了默认的blog的版本，你也可以通过`blogdown::install_theme(theme = 'XX/XX')去下载GitHub上的其他Hugo模板，相关模版可以在[3]中找到，但是遗憾的是本人发现好多模板我在我托管到netlify的过程中并不能正常渲染，在blogdown的官方文档中也提到，并没有尝试所有的Hugo主题，如有问题可以在留言区共同讨论完成。

那我们这一个环节就尝试一下我们我们自己下载的hugo主题去部署。读者向后站，我要看是blogdown建站了！

步骤如下：

+ 现在回到rstudio，File -> New Project -> Version Control -> Git，然后填写Repository URL:https://github.com/yourGithubName/domainname.com，Project directory name应该自动就生成了，可以选择一个合适的文件夹存放，点击Create Project创建项目。

+ 设置gitignore

打开rstudio右下角的Files标签，点击.gitignore文件，改成下面这样吧（copy Yihui的）：

```r

.Rproj.user 
.Rhistory 
.RData
.Ruserdata
public
static/figures
blogdown

```

+ 初始化blogdown

打开：File -> New Project -> New Directory -> Website using blogdown

+ 我选择的是Air这个主题，`blogdown::install_theme(theme = 'syui/hugo-theme-air') 稍等片刻

+ `blogdown::serve_sit()`或`blogdown::build_site(local=TRUE)` 运行blog

+ 如果正常运行，OK托管到GitHub就OK了

下面展示一下这个主题

![]({{site.url}}/assets/bowen27/deos.gif){:height="100%" width="90%"}

+ 可以通过点击菜单Help下面的Addins，这次我们点击New Post，就会弹出画面，Filename处会自动帮你填写为Title处的内容，Filename和Slug还是建议使用字母，尤其是Filename，如果博文里面不需要用到R语言的代码计算结果生成图表的话，Format处就选择Markdown格式，这可以省去一些系统生成的步骤，ok，点击Done，就会在/content/post文件夹下面生成一个文件名为2000-01-01-my-first-blog.Rmd这样的文件了，content文件夹下面的文件就是博客的文章了。

这个时候就可以用markdown格式专注于写作了。



## netlify部署

这一部分就比较简单的，但是一定要注意，hugo版本的设置，在部署的过程中，本人也踩过不少坑，现在这种复杂的hugo主题，在netlify的渲染仍然不成功，不知道是blogdown的原因还是netlify的原因。

在这里我想说的是我们同样可以依托github做线上部署，但是在官方文档中，谢大神还是推荐大家使用netlify做部署，除了hugo之外netlify同样也可以部署jekyll。 一句话我喜欢。

部署好之后我们可以修改域名，同时我们也可以设置自己购买的私有域名，这样看起来就高大上很多了。关于部署的相关内容可以参考[1][2]


## 博客展示

遗憾的是Air主题我并没有部署成功，只是用了这里的默认主题，可以看看部署的效果

[点这里](https://nuannuan.netlify.com/)


**参考内容**

[1].https://mp.weixin.qq.com/s/bULo3iv__f-64B_IMsPsnw

[2].https://bookdown.org/yihui/blogdown/

[3].https://themes.gohugo.io/

[4].https://nuannuan.netlify.com/


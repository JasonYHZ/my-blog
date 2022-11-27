---
title: 完美解决！使用Github创造一个带有CDN加速的完美图床，并配合工具实现图片水印和压缩
date: 2022-11-27 11:00:00
comment: true
toc: true
cover: false
tags:
    - 图床
    - CDN
    - Github
    - PicGo
    - 博客写作
categories:
    - 教程
    - 实用技巧
description: 本文将介绍一种简单实用的方式实现一个完美且强大的图床功能，可稳定用于博客写作等需求，并且针对博客场景还会介绍工具实现自动水印和图片压缩功能。
---

## 前言

对于纯静态化部署的博客站点来说，一般写作时使用 markdown 来编写文章，当面面临需要使用图片的时候就会比较难办，一般有两种方式来解决此问题。

方案一：最简单的方法就是将图片等媒体资源与站点文件耦合到一起，此方案虽然简单，但是使用起来比较麻烦引用图片的时候需要解决复杂的路径问题，而当博客部署的时候图片资源也会挤占我们有限的静态部署空间和网络带宽，体验不是太好。

方案二：另一种方案则是实用图床将需要使用到的图片上传到一个统一的地方进行管理，在博客中直接插入图片在图床中的 URL 链接即可，此方案优点就是方便快捷并且节省静态服务的空间和网络资源，但是此方案的问题就是我们需要找到一个地方来托管我们的图片，对于纯静态无服务端的博客来说再弄个服务器来部署图片资源显然不太可能，那么图床的使用场景就应运而生，但是目前好用的图床并不多想找到一个靠谱的图床不容易，试想下突然有一天你所用的图床突然停止服务了，那么你所有的图片就都没了，这种代价应该是谁都无法接受。

因此我此前在寻找图床过程中只有两个标准，1稳定可控、2访问速度，所以收费还是付费的我都测试过不少，但最终好多图床在第一条上都无法通过，因为没有控制权，如果哪天不能访问的话我们连把图片备份下来的机会都没有，然后基于这个因素我之后测试了实用 git 托管的方式来解决这个问题，后来考虑到访问速度因素，曾经有段时间我曾使用过 gitee 来托管图片，但是最近突如其来的审查机制导致这个方案也没法用了。

## Github + JsDelivr 方案介绍

这个方案的的思路其实跟 gitee 几乎是一样的，都是使用 git 托管图片文件，因此我很快就将 gitee 上的文件迁移到了 github 因为它的的稳定性和可控性应该是最好的，但是 github 唯一的问题就是他的访问速度因为一些未知因素导致有些地方访问 raw.githubusercontent.com 域名下的文件会很慢甚至无法打开，因此在使用这个方案来托管了文件之后还要使用一个 CDN 来解决访问速度问题，目前做的比较好的就是 [JsDeliver](https://www.jsdelivr.com/) 的 CDN， 做前端的同学应该不陌生，它不但提供了很多 npm 以及 js 文件的加速，而且它还提供了 github 和 wordpress 的加速服务，并且使用简单，只需要替换 CDN 域名即可到达加速效果。
例如加速 github 文件只需要将 github 的文件地址转换成如下地址即可：

`https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@发布的版本号/文件路径` 

其中 `@发布的版本号` 是非必需的,如果不带默认取的是仓库主分支的最新文件。

## 👷方案施工

1. 创建一个可公开访问的 [Github 仓库](https://github.com/new)
   
   例如下图所示创建一个名为 `img-bed` 的仓库。（名称可以随便取，自己开心就好😄）
   ![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271241119.webp)

2. 上传文件

   然后用你自己知道的方式上传文件即可。例如我的仓库中传入的文件如下
   ![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271244496.webp)

3. 更换 URL 链接使用
   
   上传完成后，获取到文件的链接，然后按照方案介绍中的方式将链接更换为 CDN 加速链接即可，例如
   原文件地址：`https://raw.githubusercontent.com/JasonYHZ/CDN/master/blog/202211271244496.webp`
   更换地址后：`https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271244496.webp`

以上步骤就是这个方案的全部使用过程，虽然过程比较简单，但也比较繁琐，如果写一篇文章有十几张图片，光是处理这个路径就要麻烦的要死，因此后面会介绍工具来自动帮助我们完成这些操作，甚至还可以自动帮我们压缩图片，添加水印。

## 🔧使用工具解放双手 PicGo

PicGo 官网 [https://picgo.github.io/PicGo-Doc/zh/guide/](https://picgo.github.io/PicGo-Doc/zh/guide/)，它是一个多平台的可用于快速上传图片并获取 URL 链接的工具。更详细的使用说明可以参考官方说明文档。这里直接切入主题介绍下上传设置，水印处理，和压缩等操作设置。

### 配置上传

PicGo 支持多种上传，这里我们使用 github 的图床设置，如图。

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271302123.webp)

没什么特别复杂的东西只是需要申请一个 token 来访问 github 仓库即可，创建 token 地址： [https://github.com/settings/tokens](https://github.com/settings/tokens)

点击 `Generate new token` 选择 `Generate new token (classic)`

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271306480.webp)

然后把 repo 的权限勾选即可，过期时间可以设置成永不过期。

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271307555.webp)

然后下滑到最底部，点击 `Generate token` 按钮。然后回到了前一个页面并显示刚刚创建的 Token

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271310667.webp)

{% note warning %}

这个 token 生成后只会显示一次，要及时复制备份。

{% endnote %}

此时就可以是用这个工具来上传图片了，至于其他使用设置，请看[官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/)的说明。


### 安装插件

PicGo 可以安装插件来自动对你的图片设置水印。我使用的是在线安装的方式来安装的，需要有 `Node.js` 的环境支持，如果你无法使用在线方式，可以参阅[官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/)的其他方式进行安装

另外使用在线安装的方式是使用 `Npm` 来安装插件的，如果遇到下载问题可以设置插件下载代理或者 `NPM` 下载镜像。

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271316350.webp)

### 自动水印插件

[picgo-plugin-watermark](https://github.com/fhyoga/picgo-plugin-watermark)


![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271318861.webp)

安装后设置一下就可以了

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271318077.webp)

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271319592.webp)

更多说明请参阅[插件仓库](https://github.com/fhyoga/picgo-plugin-watermark)的说明文档

### 自动压缩插件

[picgo-plugin-compress](https://github.com/JuZiSang/picgo-plugin-compress)

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271321483.webp)

![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271321223.webp)

设置压缩方式，其他方式请参阅[插件仓库](https://github.com/JuZiSang/picgo-plugin-compress)的说明文档
![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271322559.webp)

设置完成后需要开启。
![](https://cdn.jsdelivr.net/gh/JasonYHZ/CDN/blog/202211271321559.webp)


---
title: Hexo 博客搭建记录 Windows
date: 2018-09-23 16:36:08
tags: hexo
---

##前言
挺久之前就想弄个博客吧，由于拖延症硬是活生生拖到了毕业，趁着手头上没啥东西，抽空把博客搭出来。
原本想自己写一个，后面觉得太麻烦。还是用 Hexo 好了。
搭建的过程并没有一帆风顺，小小记录一番，也便后来人参考。

## 搭建过程

### 前提准备
>使用之前需要一些前提环境 (Windows)
* NPM (一般下载 Node.js 安装，就会自动安装)

### 如何安装
> 以下命令全在命令行完成 (Win + R -> cmd)
* `npm i hexo-cli -g`
* `hexo init <项目名称>`
* `cd <项目名称>`
* `npm i`
> 安装主题

在[主题商店](https://hexo.io/themes/)挑选一个喜欢的主题，然后进入到其仓库地址下载下来，放到项目目录的`/themes`下。

> 以我使用的 Material 为例
* 从项目地址,下载项目
* 放入`/themes`下，更名为 Material。
* 修改`/_config.yml`文件， 找到`theme`，修改为 `Material(/themes下的文件夹名称)`
* 进入`/themes/material`，将`_config.template.yml`复制一份，重命名为`_config.yml`。

### 挂载 Github
> 这里需要一个 Github 账号，请自行注册

* 安装`git`插件 `npm install hexo-deployer-git --save`
* 创建一个新项目，命名为`<username>.github.io`
* 复制项目的 `ssh` 地址
* 修改`/_config.yml`文件

        deploy:
          type: git
          repo: <git https 地址>
          branch: master
进入 [GitHub](https://github.com) 项目地址， 选择 `Setting` 选项卡

向下滚动找到GitHub Pages，配置 GitHub Pages
### 命令
* 新建文章

  运行 `hexo new post <博客名>`, 在 `/source/_posts` 目录下会看到新建的我呢推荐
* 测试
  
  运行 `hexo s --debug`， 然后直接到 `localhost:4000` 预览项目

* 发布 

  运行 `hexo d -g`， 将自动发布到 GitHub, 然后就可以开始访问你的博客了。地址为`<username>.github.io`
   

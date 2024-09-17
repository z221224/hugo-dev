+++
title = 'Hugo新建博客'
date = 2024-08-25T16:39:22+08:00
draft = true
+++

# 1 环境准备

## 1.1 Git下载

- 前往【[Git官网](https://git-scm.com/)】，下载安装程序
- 一直点下一步，默认安装即可。

## 1.2 Hugo下载

- 前往【[Hugo Github Tags](https://github.com/gohugoio/hugo/tags)】，选择对应版本下载，下载后解压即可
- Windows下载版本：**hugo_extended_xxxxx_windows_amd64.zip**

# 2 搭建博客

## 2.1 创建博客

（1）解压缩hugo_extended_xxxxx_windows_amd64.zip

在**hugo.exe**所在文件夹的地址栏敲打cmd，然后Enter唤起命令行

（2）敲打命令`hugo new site xxxx`创建hugo文件

（3）敲打命名`cd xxxx`切换目录，并把**hugo.exe**复制到刚生成的文件夹中

![Snipaste_2024-08-25_16-47-04](E:\stu\blog\mantao_test01\content\post\hugo新建博客\assets\Snipaste_2024-08-25_16-47-04.png)

（4）启动服务 hugo server -D ，访问[http://localhost:1313](http://localhost:1313/)，

​		 停止服务 Ctrl+C **（hugo默认是没有主题的，需要进行主题配置）**

## 2.2 配置主题

（1）前往【[Hugo Themes](https://themes.gohugo.io/)】，查找自己喜欢的主题，进行下载

（2）这边以【[Stack主题](https://github.com/CaiJimmy/hugo-theme-stack/tags)】为例，将下载好的主题解压，放到`/themes`文件夹中

**（3）将`exampleSite`样例数据中的 Content 和 hugo.yaml 复制到主文件夹中，并删掉`hugo.toml`和`content/post/rich-content`**

（4）修改 **hugo.yaml** 中的 **theme**，将他修改为跟主题文件夹同名

![](E:\stu\blog\mantao_test01\content\post\hugo新建博客\assets\Snipaste_2024-08-25_16-54-35.png)

（5）再次启动hugo服务，查看主题，具体主题配置修改 **hugo.yaml**。

# 3 新建文章

> hugo new content post/hugo新建文章/index.md


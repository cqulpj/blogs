---
layout: post
title:  "Apt、Pip等常用软件工具配置国内源"
author: lpj
categories: [ 技术, Linux ]
image: assets/images/3.jpg
tags: featured
---

## 一、debian系linux系统的APT源

* 打开`/etc/apt/source.list`文件
  ```bash
  sudo gedit /etc/apt/sources.list
  ```

* 网上搜索系统版本对应的国内源地址，然后清空sources.list文件，将新的地址复制粘贴到文件并保存，注意源地址每一行都以deb开头，复制粘贴过程中不能换行，以Ubuntu 18.04为例，阿里云的源地址如下：
  ```bash
  deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  ```
* 保存文件，执行下面的命令更新软件包信息
  ```bash
  sudo apt update
  ```


## 二、Python自动安装工具pip源

1. 国内常用pip源及其地址见下表，在linux和windows平台下配置源的方式稍有不同，下文会分别讲述：

    | pip源 | 地址 |
    | ----- | ---- |
    | 清华 | [https://pypi.tuna.tsinghua.edu.cn/simple](https://pypi.tuna.tsinghua.edu.cn/simple) |
    | 阿里云 | [http://mirrors.aliyun.com/pypi/simple/](http://mirrors.aliyun.com/pypi/simple/) |
    | 豆瓣 | [http://pypi.douban.com/simple/](http://pypi.douban.com/simple/) |

2. Linux平台

    * 创建~/.pip/pip.conf文件
      ```bash
      mkdir -p ~/.pip
      vi ~/.pip/pip.conf
      ```

    * 修改pip.conf文件，写入以下内容设置源（此处以清华源为例），然后保存即可
      ```bash
      [global] 
      index-url = https://pypi.tuna.tsinghua.edu.cn/simple
      [install]
      trusted-host = https://pypi.tuna.tsinghua.edu.cn
      ```
3. Windows平台

    * 在`%APPDATA%`目录创建名为pip的文件夹，然后在pip文件夹里创建pip.conf文件

    * 打开pip.conf文件，添加源设置，内容与Linux下文件内容一样，然后保存退出


## 三、Docker镜像服务器源

* 打开docker源配置文件
  ```bash
  sudo gedit /etc/docker/daemon.json
  ```

* 打开文件后，复制下面内容粘贴到文件里面并保存
  ```bash
  {
      "registry-mirrors": ["http://hub-mirror.c.163.com"]
  }
  ```

  上面使用的是163的源，目前国内常用的源主要有：  

  | 源名称 | 地址 |
  | ------------ | ---- |
  | Docker中国区 | [https://registry.docker-cn.com](https://registry.docker-cn.com) |
  | 网易 | [http://hub-mirror.c.163.com](http://hub-mirror.c.163.com) |
  | 中科大 | [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn) |
  | 阿里云 | [http://(你的ID).mirror.aliyuncs.com](http://(你的ID).mirror.aliyuncs.com) |
  
  其中阿里云需要先登录->创建我的容器镜像，然后得到专属的镜像服务地址

* 执行下面命令重启docker服务
  ```bash
  sudo systemctl restart docker.service
  ```

## 四、Nodejs包管理工具npm源

* 可以先用下面的命令查看当前npm源（这一步可以不做）
  ```bash
  npm config get registry
  ```

* 用下面的命令修改npm源为淘宝镜像源
  ```bash
  npm config set registry https://registry.npm.taobao.org
  ```

  修改完毕后可以用第一步的`npm config get registry`命令查看是否修改成功
  


## 五、Ruby包管理工具gem源

* 可以用下面的命令查看当前gem源
  ```bash
  gem sources -l
  ```

* 用下面的命令修改gem源为国内源
  ```bash
  gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
  ```

* 上一步命令是增加国内源同时删除默认官方源，也可以分别来操作，gem源设置命令有：
  ```bash
  # 查看gem源
  gem source -l
  # 删除源
  gem source --remove <源地址>
  gem source -r <源地址>
  # 添加源
  gem sources --add <源地址>
  ```


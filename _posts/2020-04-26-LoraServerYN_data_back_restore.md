---
layout: post
title:  "LoraServerYN_Docker中间件数据迁移说明"
author: lpj
categories: [ Lora, Docker ]
image: assets/images/11.jpg
tags: featured
---

## 一、数据迁移说明

### 1.1 数据迁移原理
docker容器创建后会在本地硬盘中创建虚拟卷volumes，具体路径为：/var/lib/docker/volumes（需要管理员权限才能访问），数据库数据存储在volumes中，对于LoraServerYN中间件来说，它的数据库主要有两个，分别是postgresql和redis，对应的数据库虚拟卷分别是volumes路径中的loraserveryn_docker_postgresqldata和loraserveryn_docker_redisdata两个文件夹，只要把这两个文件夹迁移到新服务器上对应的路径中即可实现数据迁移。

因为虚拟卷中文件很多，且权限较高，若直接复制粘贴，一是速度太慢容易卡住，二是存档不方便，所以本文档中采用tar包压缩和解压来实现数据的备份和恢复。

### 1.2 数据迁移适用于以下情况：
1. 服务器变更，需要将旧的服务器上的数据搬移过来
2. 服务器上Docker或者中间件启动不了，只能重装系统或者换新的服务器，需要对数据进行备份恢复
3. 定期对中间件数据进行备份存档

### **1.3 不适用于以下情况：**
1. 有两个服务器同时在适用，需要将一个服务器上的数据同步到另一个服务器中，即数据合并

## 二、数据备份

1. 打开终端，切换(cd)到要存储备份文件的目录，比如/home/gg，命令为:
    ```bash
    cd /home/gg
    ```

2. 执行以下命令将两个数据库目录压缩为databack.gz，若当前用户已是root，则不用加`sudo`
    ```bash
    sudo tar -zcvf databack.gz /var/lib/docker/volumes/loraserveryn_docker_*
    ```

3. 查看当前目录是否有已生成的databack.gz文件，该文件即为数据库备份文件，可以存档，也可以复制到新服务器进行数据恢复

## 三、数据恢复
首先确保新服务器上的中间件处于关闭状态，如果不确定，那么在中间件目录执行`docker-compose down`

1. 若在新服务器上已经运行过中间件，那么在volumes目录已经有生成的数据库文件夹，此时需要删除或者将它改名，建议改名，命令为：
    ```bash
    sudo su
    cd /var/lib/docker/volumes
    mv loraserveryn_docker_postgresqldata loraserveryn_docker_postgresqldata_bak
    mv loraserveryn_docker_redisdata loraserveryn_docker_redisdata_bak
    ```

2. 将在旧服务器上复制来的数据备份文件放到新服务器任何路径如/home/gg（如果是U盘可以不用进行该步骤，就在U盘路径中进行），然后启动终端，切换到该路径，如：
    ```bash
    cd /home/gg
    ```

3. 执行下面的命令将数据库恢复到虚拟卷中
    ```bash
    sudo su
    tar -zxvf databack.gz -C /var/lib/docker/volumes/
    ```

4. 启动中间件，查看数据恢复情况
  

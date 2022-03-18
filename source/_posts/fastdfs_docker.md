---
title: 相当靠谱的FastDFS Docker镜像
date: 2018-09-01 15:46:09
categories: 程序人生
tags:
    - Docker
    - FastDFS
---
该项目是我在网上搜的，迄今为止个人感觉最靠谱的`FastDFS`镜像，项目地址https://github.com/luhuiguo/fastdfs-docker

开启一个`tracker`
```
docker run -dti --network=host --name tracker -v /var/fdfs/tracker:/var/fdfs luhuiguo/fastdfs tracker
```
开启一个`storage`
```
docker run -dti --network=host --name storage0 -e TRACKER_SERVER=192.168.0.88:22122 -v /var/fdfs/storage0:/var/fdfs luhuiguo/fastdfs storage
```
开启一个`storage`，并指定`Group`
```
docker run -dti --network=host --name storage2 -e TRACKER_SERVER=10.1.5.85:22122 -e GROUP_NAME=group2 -e PORT=22222 -v /var/fdfs/storage2:/var/fdfs luhuiguo/fastdfs storage
```
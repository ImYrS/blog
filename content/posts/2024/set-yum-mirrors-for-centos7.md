---
title: "为过时的 CentOS 7 换源"
description: "在 CentOS 7 结束维护后，如何更换 yum 源"
date: 2024-10-10T15:20:00+08:00
lastmod: 2024-10-10T15:20:00+08:00
categories: ["教程"]
tags: ["CentOS", "服务器"]

draft: false
---

CentOS 7 已经于 2024 年 6 月 30 日结束维护，不再提供更新。官方的 yum 源也已经关闭，但是可以通过更换其他镜像源来继续使用。

### 备份

建议先备份一下之前的镜像源列表，以便出现问题时可以恢复。

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

### 使用国内镜像源

这里以阿里云的镜像源为例，可以根据自己的需求选择其他镜像源。

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

再提供一个网易的镜像源。

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
```

### 重建缓存

更换完镜像源后，需要重建一下 yum 缓存。

```bash
yum clean all
yum makecache
```

现在应该可以正常使用 yum 命令了。

如果还是有问题，可能是添加了一些其他的源，比如 docker 源。此时可以进入 `/etc/yum.repos.d/` 目录，删除一些不必要的源，尤其是跟 CentOS 7 源有冲突的。然后再重建缓存。

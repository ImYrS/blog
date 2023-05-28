---
title: "在 CentOS 7 上安装 Python 3.11"
description: "如何在 CentOS 7 上编译安装 Python 3.11, 3.10 或更高版本并避免 SSL 模块报错"
date: 2023-05-12T23:00:48+08:00
lastmod: 2023-05-12T23:00:48+08:00
categories:
tags:

weight:
draft: false
---

## 前情提要

之前一直使用 Python 3.7 都是直接安装, 没有什么复杂步骤. 前段时间因为项目使用了一些新特性, 切换到 3.10 和 3.11 后发现, 只要涉及 `HTTPS` 请求就会报 `SSL` 模块错误.

本文介绍了如何解决这个问题, 并加以总结, 以便日后查阅.

## 安装依赖

```bash
yum -y groupinstall "Development tools"
yum install -y yum install wget gcc-c++ pcre pcre-devel zlib zlib-devel libffi-devel zlib1g-dev openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel bzip2-devel
```

## 安装 OpenSSL

1. 查看 `openssl` 版本

   ```bash
   openssl version
   ```

   一般 CentOS 7 自带的 `openssl` 版本是 `1.0.2k` 之类的, 不符合 `python 3.10` 版本的要求, 所以直接安装 `python` 后一旦请求 `HTTPS` 链接就会报 `SSL` 模块错误.

2. 删除原 `openssl`

   ```bash
   yum remove openssl
   ```

3. 更新 CentOS 信任根证书, 防止遇到 `HTTPS` 不信任证书的问题

   ```bash
   yum install ca-certificates
   update-ca-trust force-enable
   update-ca-trust extract
   ```

4. 下载最新 `openssl` 并解压

   ```bash
   wget https://www.openssl.org/source/openssl-1.1.1t.tar.gz
   tar -zxf openssl-1.1.1t.tar.gz
   cd openssl-1.1.1t
   ```

   需要注意, 本文发布时已经 `openssl` 已经发布了 `3.0.0` 版本, 但没有测试该大版本与 `python` 的兼容性, 建议谨慎操作.

5. 配置并编译安装

   ```bash
   ./config --prefix=/usr/local/openssl
   make -j && make install
   ```

6. 创建软链接并配置环境变量

   ```bash
   ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
   ```

   用你喜欢的编辑器修改 `/etc/ld.so.conf` 如 `vim /etc/ld.so.conf`

   在文件末尾添加 `/usr/local/openssl/lib` 

   ![image-20230512235819057](https://cdn.imyrs.cn/u/i/img/202305122358082.png)

   然后使配置生效

   ```bash
   ldconfig -v
   ```

7. 验证

   在任意目录下输入

   ```bash
   openssl version
   ```

   可以正确输出新安装的版本即可

   ![image-20230512235726735](https://cdn.imyrs.cn/u/i/img/202305122357765.png)

## 安装 Python 3.11

1. 下载安装包并解压

   ```bash
   wget https://www.python.org/ftp/python/3.11.3/Python-3.11.3.tgz
   # 国内服务器可使用淘宝镜像
   # wget https://registry.npmmirror.com/-/binary/python/3.11.3/Python-3.11.3.tgz
   tar xvzf Python-3.11.3.tgz
   cd Python-3.11.3
   ```

2. 配置并编译

   ```bash
   ./configure --prefix=/usr/local/python3.11 --with-openssl=/usr/local/openssl --with-openssl-rpath=auto --enable-optimizations
   make -j && make altinstall
   ```

   其中最重要的是 `--with-openssl=/usr/local/openssl` 和 `--with-openssl-rpath=auto` 这两个关键参数决定了能否使用最新的 `openssl`, 也是本文中最大的坑.

   如果 `--enable-optimizations` 打开优化后编译失败或遇到 `Could not import runpy module` 错误, 可以考虑移除这个参数.

3. 创建软链接

   ```bash
   ln -sf /usr/local/python3.11/bin/pip3.11 /usr/bin/pip3.11
   ln -sf /usr/local/python3.11/bin/python3.11 /usr/bin/python3.11
   ```

4. 验证

   ```bash
   python3.11 -V
   pip3.11 -V
   ```

   ![image-20230513000422594](https://cdn.imyrs.cn/u/i/img/202305130004616.png)

## 建议

在使用 `python` 运行项目是, 推荐使用虚拟环境, 防止影响全局环境.

比如在 `/www/program/some-project` 这个项目中, 可以使用如下命令创建并使用虚拟环境

```bash
cd /www/program/some-project
python3.11 -m venv venv
source venv/bin/activate
```

此时, 终端前会显示 `(venv)`, 即代表使用了当前的虚拟环境, 此时可以直接用 `python` 和 `pip` 命令而不是 `python3.11` 和 `pip3.11`. 使用 `pip` 安装的依赖也仅在当前虚拟环境中可用.

![image-20230513001137367](https://cdn.imyrs.cn/u/i/img/202305130011389.png)
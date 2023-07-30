---
title: "一种基于 commit 自动更新和部署博客的方案"
description: "对比 git commit hash 来检查是否存在内容更新并重新构建静态页面的方案"
date: 2023-07-28T22:19:06+08:00
lastmod: 2023-07-30T14:42:00+08:00
categories: ["教程", "技术"]
tags: ["博客", "Git", "Hugo", "静态博客"]

weight:
draft: false
---

## 前文

本博客目前使用 Hugo 构建. 众所周知, 如果 Hugo 等静态博客部署在 CloudFlare 可以通过 CloudFlare Pages 自带的功能获取 push 推送, 自动化构建最新的内容, 十分方便. 但如果想要把博客部署在自己的服务器上呢?

通常有两种方案, 一是自己每次写新文章, 或者更新内容以后, 手动 `git pull && hugo`. 第二种方案则是通过 GitHub Actions 或者 webhook 推送更新到服务器. 个人尝试过采用 webhook 的方案, 但不知为何经常失败, 脚本也有一定问题. 现在突然想到一些略有繁琐, 但也能实现自动更新的办法, 分享出来.

### 本方案适用于

Hugo, Hexo 等静态内容构建程序. 构建页面时间较短, 页面数量相对不多的情况.

### 可能不适用于

动态内容博客, 构建时间较长, 随时更新可能导致问题的情况.

## 原理

每个 commit 都有其对应的 hash 值, 可以定时 `git pull` 从仓库拉取内容, 并对比 `pull` 前后 commit 值的变化, 以判断是否存在更新.

如果存在更新, 则执行重新构建的流程.

## 实现

### 通用思路

创建一个脚本文件, 可以在本地写好上传到服务器, 也可以直接在服务器写.

```bash
nano update-blog.sh
```

然后写入以下内容, 按需替换.

```bash
cd {{your site folder}}

commit=$(cat .git/refs/heads/main)

git pull

new_commit=$(cat .git/refs/heads/main)

if [[ "$new_commit" != "$commit" ]]; then
        {{execute your site rebuild program}}
fi

echo "----------------------------------------------------------------------------"
endDate=`date +"%Y-%m-%d %H:%M:%S"`
echo ">> [$endDate] Successful"
echo "----------------------------------------------------------------------------"
```

`{{your site folder}}` 部分改为你站点路径

`{{execute your site rebuild program}}` 部分改为你的网站构建流程

### Hugo

我自己的 Hugo 更新完整代码如下:

```bash
cd /www/imyrs.cn

commit=$(cat .git/refs/heads/main)

git pull

new_commit=$(cat .git/refs/heads/main)

if [[ "$new_commit" != "$commit" ]]; then
        echo "Updates detected, rebuilding pages."
        rm -rf public/*
        # 我将 Hugo 程序本体放在网站根目录
        # 所以生成静态页面的命令就是 ./hugo
        # 其他情况需要按需配置
        ./hugo
fi

echo "----------------------------------------------------------------------------"
endDate=`date +"%Y-%m-%d %H:%M:%S"`
echo ">> [$endDate] Successful"
echo "----------------------------------------------------------------------------"
```

## 自动化

通过配置服务器 crontab 来实现自动更新

```bash
crontab -e
```

添加一条

```bash
*/10 * * * *  /bin/bash /root/update-blog.sh >> /www/logs/update-blog.log 2>&1
```

表示每十分钟执行一次脚本, 并将结果输出到 `/www/logs/update-blog.log` 文件中.

可以按需添加更多输出, 包括运行时间等等.

## 其他

如果你博客的 GitHub 仓库不是公开仓库, 那在执行 `git pull` 的时候可能需要输入用户名和密码, 无法实现自动化. 你需要在服务器项目路径内执行 `git config credential.helper store`, 然后手动 `git pull` 一次, 凭证将自动保存在服务器, 下次无需输入密码.
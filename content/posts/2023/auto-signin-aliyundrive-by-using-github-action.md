---
title: "使用 GitHub Actions 实现阿里云盘自动签到"
description: "使用 GitHub Actions 无需服务器实现自动化阿里云盘的每日自动签到"
date: 2023-06-07T22:08:26+08:00
lastmod: 2023-06-07T22:08:26+08:00
categories: ["教程", "技术"]
tags: ["阿里云盘", "自动化", "GitHub", "Actions"]

weight:
draft: false
---

## 概述

本文介绍了如何使用 [ImYrS/aliyun-auto-signin](https://github.com/ImYrS/aliyun-auto-signin) 项目, 利用 GitHub 提供的 Actions 功能, 实现每日自动签到阿里云盘, 无需服务器, 无需额外的费用.

此文默认你知道:

1. GitHub 是什么, 如何访问及其基础知识
2. 阿里云盘签到的相关内容

## 部署步骤

### 创建仓库

在 GitHub **创建** 一个新仓库, **不要 Fork**. 仓库名称可以自己设置.

![](https://cdn.imyrs.cn/u/i/img/202306072221078.png)

此处推荐使用 **公开** 仓库 (Public Repo). 因为按照 [GitHub 计费说明](https://github.com/settings/billing/plans) 所述, 公开仓库的 Actions 不计费, 私人仓库会有运行时间限制. 即使是公开仓库, 配置中的机密参数也无法被其他人看到.

### 创建并配置 Actions 文件

在仓库中创建 `.github/workflows/signin.yml` 文件. 注意此处是纯英文路径, 不要使用浏览器翻译, 可能导致其他问题.

![](https://cdn.imyrs.cn/u/i/img/202306072226627.png)

将如下内容原封不动的粘贴至该文件中, 保存.

```yml
name: Aliyun Signin

on:
  schedule:
   # 每天国际时间 14:40 运行一次, 中国时间 22:40
    - cron: '40 14 * * *'
  workflow_dispatch:
jobs:
  signin:
    name: Aliyun Signin
    runs-on: ubuntu-latest
    steps:
      - uses: ImYrS/aliyun-auto-signin@main
        with:
          REFRESH_TOKENS: ${{ secrets.REFRESH_TOKENS }}
          GP_TOKEN: ${{ secrets.GP_TOKEN}}
          PUSH_TYPES: ''
          DO_NOT_REWARD: 'false'
          SERVERCHAN_SEND_KEY: ${{ secrets.SERVERCHAN_SEND_KEY }}
          TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
          PUSHPLUS_TOKEN: ${{ secrets.PUSHPLUS_TOKEN }}
          PUSHPLUS_TOPIC: ${{ secrets.PUSHPLUS_TOPIC }}
          SMTP_HOST: ${{ secrets.SMTP_HOST }}
          SMTP_PORT: ${{ secrets.SMTP_PORT }}
          SMTP_TLS: ${{ secrets.SMTP_TLS }}
          SMTP_USER: ${{ secrets.SMTP_USER }}
          SMTP_PASSWORD: ${{ secrets.SMTP_PASSWORD }}
          SMTP_SENDER: ${{ secrets.SMTP_SENDER }}
          SMTP_RECEIVER: ${{ secrets.SMTP_RECEIVER }}
          FEISHU_WEBHOOK: ${{ secrets.FEISHU_WEBHOOK }}
          WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}
          CQHTTP_ENDPOINT: ${{ secrets.CQHTTP_ENDPOINT }}
          CQHTTP_USER_ID: ${{ secrets.CQHTTP_USER_ID }}
          CQHTTP_ACCESS_TOKEN: ${{ secrets.CQHTTP_ACCESS_TOKEN }}
```

此时应如图所示

![](https://cdn.imyrs.cn/u/i/img/202306072228928.png)

### 获取阿里云盘账号 `refresh token`

在电脑浏览器**使用无痕模式**打开 [阿里云盘官网](https://aliyundrive.com) 并登录账号

按 F12 打开开发者工具, 在控制台内输入 `JSON.parse(localStorage.token).refresh_token` 或 `console.log(JSON.parse(localStorage.token).refresh_token)` 即可看到当前帐号的 `refresh token`.

![](https://cdn.imyrs.cn/u/i/img/202306072243468.png)

**不要在浏览器点击退出登录按钮, 这可能会导致令牌提前失效.** 直接关闭无痕浏览器即可.

如果需要同时签到多个账号, 现在请保存好此令牌. 重复上述流程, 获取多个令牌并保存好.

### 配置 GitHub Secrets

在仓库上方的导航按钮中点击 `Settings`, 依次进入 `Secrets and Variables` -> `Actions` 中并点击 `New repository secret`.

![](https://cdn.imyrs.cn/u/i/img/202306072250097.png)

参考[项目教程](https://cdn.imyrs.cn/u/i/img/202306072254456.png)配置 Secrets

#### 阿里云盘 `refresh token`

名称为 `REFRESH_TOKENS`, Secret 值就是刚才获取到的 `refresh token`.

![](https://cdn.imyrs.cn/u/i/img/202306072253345.png)

多账户同时签到时, 使用英文逗号将多个 `token` 隔开.

![](https://cdn.imyrs.cn/u/i/img/202306072254456.png)

#### GitHub Personal Token

前往 [Personal Access Tokens 配置页](https://github.com/settings/tokens). 点击 Generate new token **创建 classic 类型**的 token.

![](https://cdn.imyrs.cn/u/i/img/202306072301157.png)

名称 (Note) 自己填, 过期时长建议三个月以上. 时长越久, 越方便, 但也更不安全. **权限必须勾选 repo**.

![](https://cdn.imyrs.cn/u/i/img/202306072303513.png)

创建后 token 只能查看一次, 务必复制保存好.

回到项目的 Secrets 配置页面并添加刚创建的 token, 名称为 `GP_TOKEN`

![](https://cdn.imyrs.cn/u/i/img/202306072306583.png)

### 配置推送渠道

至此, 必须添加的两个 secrets 已经添加完成了. 如果需要配置签到结果推送, 则按需配置选定的推送渠道需要的 secrets. 此处以 `telegram` 为例. 所有支持的推送渠道可以在[项目说明](https://github.com/ImYrS/aliyun-auto-signin#%E6%8E%A8%E9%80%81%E6%B8%A0%E9%81%93)中查看.

#### 编辑 Actions 配置文件

将其中的 `PUSH_TYPES` 值改为配置的推送渠道, 多个渠道使用英文逗号隔开.

![](https://cdn.imyrs.cn/u/i/img/202306072314940.png)

按照项目说明中的提示和 Actions 配置文件, 照葫芦画瓢, 添加最相似的 secrets.

![](https://cdn.imyrs.cn/u/i/img/202306072319840.png)

将 `TELEGRAM_BOT_TOKEN` 和 `TELEGRAM_CHAT_ID` 配置到 secrets 以后即可.

### 运行 Actions

默认情况下, 中国时间每天 22:40 左右会自动运行. 可以在项目上方导航栏中的 Actions 页面中查看. 也可以手动点击运行.

![](https://cdn.imyrs.cn/u/i/img/202306072324277.png)

可以进入单次运行中查看运行结果.

![](https://cdn.imyrs.cn/u/i/img/202306072326427.png)

如果配置了推送渠道, 也可以看到签到结果.

![](https://cdn.imyrs.cn/u/i/img/202306072327627.png)

## 原理及接口

1. 通过抓包阿里云盘 App 签到过程, 获取签到接口.
2. 解析鉴权过程可知, 阿里云盘登录后获得 `refresh_token` 和 `access_token` 两个令牌, 后者为短效 JWT Token 用于请求鉴权, 前者用于更新后者.
3. 使用爬虫完成上述请求

刷新令牌 **POST** `https://auth.aliyundrive.com/v2/account/token `  
签到接口 **POST** `https://member.aliyundrive.com/v1/activity/sign_in_list `  
兑换奖励接口 **POST** `https://member.aliyundrive.com/v1/activity/sign_in_reward `

接口详情可自行[研读代码](https://github.com/ImYrS/aliyun-auto-signin/blob/main/app.py)了解.

## 结尾

项目 Telegram 交流群 [@aliyun_auto_signin](https://t.me/aliyun_auto_signin)

本文仅用于交流学习和分享.

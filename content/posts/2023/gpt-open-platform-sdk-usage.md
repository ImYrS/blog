---
title: "GPT+ Open Platform SDK 示例"
description: "GPT+ 开放平台的 SDK 简单用法介绍和示例."
date: 2023-06-20T14:13:30+08:00
lastmod: 2023-07-29T13:32:00+08:00
categories: ["教程"]
tags: ["GPT+", "GPT", "OpenAI", "SDK", "AI"]

weight:
draft: false
---

## 介绍

GPT+ ([官网链接](https://cn.ss.chat)) 是一个基于 OpenAI 的语言模型调用平台, 有 Web UI 页面, 也支持 API 接入. 在中国大陆地区有良好的连通性.

[GPT+ Open Platform](https://cn.ss.chat/user/apps), 指 GPT+ 开放平台, 又称 gpto, GPT+ Open 等. 是 GPT+ API 的授权组件和接入模块. 提供了高可读性的 API 文档和代码规范的 SDK 可供参考.

## SDK 安装

SDK 模块的安装方式可以在 GPT+ 网站中的开放平台页面查看

![](https://cdn.imyrs.cn/u/i/img/202307291313133.png)

## Python

1. 使用以下命令安装 SDK.
   ```bash
   pip install --upgrade gptopen
   ```

2. 如提示以下错误可能是因为使用了国内镜像, 尚未完成同步.
   ![](https://cdn.imyrs.cn/u/i/img/202307291318758.png)使用以下命令指定 pypi 源安装即可.

   ```bash
   pip install --upgrade gptopen -i https://pypi.org/simple
   ```

3. 引入 SDK 并初始化. **请不要将 `app_id` `app_key` 等鉴权密钥硬编码在代码中.**
   ```python
   from gptopen import App
   
   app = App(
       model='gpt-3.5-turbo-0613',
       app_id='app_id',
       app_key='app_key',
   )
   ```

4. SDK 初始化时可以选择使用 `app_id` 和 `app_key` 自动获取 `access_token`, 也可以在特定情况下直接使用已有的 `access_token`.
   ```python
   from gptopen import App
   
   app = App(
       model='gpt-3.5-turbo-0613',
       access_token='access_token',
       access_token_expired_at=0,
   )
   ```

5. 创建对话, 再发送消息接收回复.
   ```python
   app.create_conversation()  # 创建成功后对话 ID 会自动保存在实例属性中
   
   response = app.create_message(content='你好')
   print(response['content_raw'])
   ```

6. 也可以跳过创建对话, 直接发送消息. 这种情况下整体速度更快, 但接口返回数据结构不同需要注意. 这种方法一般用于单次生成回复, 不需要上下文对话的情况.
   ```python
   # 没有创建对话, 直接创建信息时, 返回的数据包含两个字段 "conversation" 和 "message"
   response = app.create_message(content='你好')
   print(response['message']['content_raw'])
   
   # 如果需要继续这个对话, 即保留上下文. 则需要手动设置实例属性
   # 不设置 conversation 属性继续调用 create_message() 会认为是全新对话
   app.conversation = response['conversation']['uuid']
   
   # 设置了对话 ID 后接口返回结构中只有 "message" 结构了
   response = app.create_message(content='你可以做些什么?')
   print(response['content_raw'])
   ```

## 注意

SDK 只是辅助调用 API, 所有返回结构等内容, 不会发生改变. 如有疑问请查阅 GPT+ 开放平台的开发文档.
---
layout: post
title: Mac网络访问Win的文件
categories: Softwares
description: Mac网络访问Win的文件
keywords: Mac, Win
---

> Mac如何实现网络访问Win文件
>

# 1.需求

通过U盘传输文件不太方便，而且QQ、微信等即时通信软件只能在一台PC上登陆，无法实现快速的文件共享。Mac和Win可以通过smb来实现文件共享。

# 2.解决过程

## 2.1 准备工作

**（1）保证Mac和Win在一个局域网下**

可以通过ifconfig（Mac）或者ipconfig（Win）来查询自身IP地址，通过ping 对方的IP确保在一个局域网且可以联通

**（2）Win配置——关闭防火墙**

```
控制面板 -> Windows防火墙 -> 打开或关闭Windows防火墙 -> 将家庭或工作（专用）网络和公用网络的防火墙都关闭
```

是否可以选择只关闭一个？待考证。

**（3）Win配置——设置高级共享**

```
控制面板 -> 选择家庭组和共享选项 -> 更改高级共享设置 -> 网络发现、文件和打印机共享、公用文件夹共享都设置为启用...
```

**（4）Win配置——添加用户**

```
管理 -> 本地用户与组 -> 给“用户”添加一个用户
```

记住用户名和密码

**（4）Win配置——安全选项**

1、运行里面输入"secpol.msc"来启动本地安全设置

2、网络安全LAN 管理器身份验证级别修改成“发送 LM 和 NTMLM响应(&)”或者“发送 LM 和 NTMLM - 如果已协商，则使用NTLMv2 会话安全”

```
本地策略 -> 安全选项 -> 网络安全LAN 管理器身份验证级别
```

3、打开来宾账户状态

```
本地策略 -> 安全选项 -> 来宾账户状态
```

4、网络访问：本地账户的共享和安全模型设置为“经典”

```
本地策略 -> 安全选项 -> 网络访问：本地账户的共享和安全模型
```

**（5）Win配置——共享文件**

```
文件夹 -> 右键选择共享 -> 特定用户 -> 添加之前创建的用户
```

## 2.2 Mac访问

```
桌面 -> Go -> Connect to Server -> 输入smb://[Win的计算机名（可以通过属性得到）] -> 输入用户名和密码 -> 连接
```


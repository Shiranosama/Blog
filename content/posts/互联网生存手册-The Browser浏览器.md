---
title: The Browser浏览器
subtitle: 互联网生存手册
date: 2026-09-18T20:44:29+08:00
slug: 13bf15e
draft: true
categories:
  - 互联网生存手册
collections:
  - 互联网生存手册
description: 互联网生存手册 隐私浏览篇，使用隐私浏览对抗互联网追踪器，让有心者难以追踪
summary: 互联网生存手册 隐私浏览篇，使用隐私浏览对抗互联网追踪器，让有心者难以追踪
---

## 岚的声明
<font size=4>

本文为**岚**的**折腾日记**，它只是因为我**想**而写下的，**不具备**权威性，我也**无法**保证正确，**仅供参考**

***本文仅作经验分享，无任何不良引导，所有内容的最终解释权归我（笔者）所有***

***读者对所有内容的实践/理解均属于读者个人行为，本文笔者不会也不应负任何责任***

</font>

```
经验之谈，无有权威；思辨一二，方得自知
```

## 浏览器

### 追踪器
网站使用**追踪器**来收集有关你的浏览行为的数据，追踪器收集有关你如何与网站互动的数据，一些追踪器会**跨多个站点**收集有关你的数据，这些数据会被服务商拿去个性化你的用户体验，一些服务商还会把收集到的数据分享给合作商；合法的服务商的数据收集条款写在你大概率不会去看就同意的《隐私政策》里（建议把完整条款丢给LLM去提炼核心内容），但也可能遇上恶意的有心者利用**追踪器**、**漏洞**去收集并追踪你的信息，常见的手段包括：**Cookie（储存用户行为的数据文件）**、**浏览器指纹（由一系列浏览器与设备信息组成）**、**IP地址**、**会话信息（记录用户输入）**、**恶意代码**等

### Cookie
是一个保存在**客户机**中的简单的文件，它的作用是**辨别用户身份**，以追踪网页会话进度，Cookie的初衷是方便网页会话实现**持久性**，但由于其特性本就具有**追踪性**，是设置跟踪器的天然土壤

对于Cookie，我们的应对手段显而易见，是**禁用/清理Cookie**，但棘手的问题在于Cookie也是网站**验证登录身份**的重要数据，一般来说不宜使用直接禁用的**激进手段**，因此通常选择退而求其次，仅禁用**跨站Cookie**——一种通过其他网站的Cookie追踪用户的手段，并配以**自动清理某些网站Cookie等数据**（通常，浏览器可以在例外中配置需要保持登录状态的网站eg. https://github.com、http://github.com），以达到**正常使用网站功能**，但**对抗部分追踪**的目的，总结步骤：
  + 禁用跨站Cookie
  + 配置退出/定时自动清理数据
  + 为需要保持登录的网站添加例外

### 浏览器指纹
这是浏览器隐私的重灾区，其主要包括：
  - User Agent
  - HTTP_ACCEPT Headers HTTP接受请求头
  - Browser Plugin Details 浏览器插件（非附加组件）
  - Time Zone Offset 时区偏移
  - Time Zone 时区
  - Screen Size and Color Depth 屏幕分辨率与色彩位深
  - System Fonts 系统字体
  - Are Cookies Enabled 是否启用Cookies
  - Limited supercookie test 检测Super cookies
  - Hash of canvas fingerprint 画布指纹的哈希值
  - Hash of WebGL fingerprint WebGL的哈希值
  - WebGL Vendor & Renderer WebGL（显卡）的供应商和渲染器
  - DNT Header Enabled 是否启用DNT（请求不追踪）请求头
  - Language 语言（一般是浏览器语言）
  - Platform 系统架构
  - Touch Support 触摸支持
  - Ad Blocker Used 是否使用广告拦截器
  - AudioContext fingerprint 音频处理器（声卡）指纹
  - CPU Class CPU制造商
  - Hardware Concurrency CPU内核数量
  - Device Memory 设备内存

上面这些数据，其初衷均是方便网页使用，很多数据相互之间的关系可以说是八竿子打不着，但有心者将零散数据集中，由此形成了被称作**浏览器指纹**的东西

# 草稿
因此浏览器层面能做到的隐私保护是**尽可能**提升**匿名性**（降低独特性），让有心者难以追踪你，说人话就是：让你在有心者看来与多数人没有区别，混进人海以提高追踪成本

我们的首要目标是让你在**第三方追踪者**眼中是匿名的

我们将以LibreWolf的默认配置为基础进行配置，以下配置项均可在任何基于Firefox的浏览器的**about:config**页面中找到：
    + 启用privacy.resistFingerprinting：这是最主要的配置，是目前同类中最好的防指纹工具，它会伪装上面指纹数据中的大多数，以让你看起来和多数人没有区别
    + 启用privacy.resistFingerprinting.letterboxing：RFP的子项，此配置会为页面周围添加边距，返回有限的一组圆角分辨率，而非真实分辨率以抵抗**屏幕分辨率**指纹，另外，不最大化浏览器窗口有助于增强此项的效果
    + 启用webgl.disabled：禁用WebGL，以抵抗**WebGL**相关指纹

- IP地址：
请使用Tor或代理，这不是浏览器本身能做到的事情

- 恶意代码：
主要靠你的安全软件以及**[uBlock Origin](https://github.com/gorhill/uBlock)**：
    + 启用安全软件的**Web扫描**
    + uBlock Origin规则：
        - Online Malicious URL Blocklist
        - Phishing URL Blocklist

- 拦截广告和追踪器：
uBlock Origin规则：
    - EasyPrivacy
    - Actually Legitimate URL Shortener Tool
    - AdGuard URL Tracking Protection
    - Peter Lowe’s Ad and tracking server list
    - uBlock filters – Ads
    - uBlock filters – Badware risks
    - uBlock filters – Privacy
    - uBlock filters – Quick fixes
    - uBlock filters – Unbreak
    - EasyList
    - CN特供：
        - CJX's Annoyance List
        - CJX's uBlock list
        - EasyList China

## 注意事项
- 上述内容在增强隐私的同时，可能会**破坏**部分网站的正常功能（如流媒体、在线办公应用），需根据实际使用情况权衡
- 推荐养成不最大化浏览器的习惯（LibreWolf窗口默认大小是最好的平衡）
- 附加组件的数量越少越好，任何附加组件都会**增加**你的**暴露面**
- LibreWolf 默认**禁用**了所有数据同步功能，以确保您的浏览数据完全保留在本地，如果您有跨设备同步的需求（如书签、密码），可以手动启用Firefox同步服务
- 不建议使用任何浏览器自带的密码管理，***请使用额外的密码管理器！！！***
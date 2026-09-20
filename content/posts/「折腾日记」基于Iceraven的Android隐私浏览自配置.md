---
title: 「折腾日记」基于Iceraven的Android隐私浏览自配置
date: 2026-09-20T01:04:57+08:00
slug: 0f97a27
draft: false
description: 利用Iceraven的about:config支持，配合Arkenfox作为配置清单，尽可能在不破坏日常浏览体验的情况下实现隐私保护
weight: 0
categories:
  - 折腾日记
  - "That's My Privacy!"
tags:
  - Privacy
summary: 利用Iceraven的about:config支持，配合Arkenfox作为配置清单，尽可能在不破坏日常浏览体验的情况下实现隐私保护
---

## 前言
移动端的隐私浏览器项目一直都比较少，目前还在活跃的例如Brave、IronFox等，又达不到我想要的可控性，于是我物色到了 **[Iceraven](https://github.com/fork-maintainers/iceraven-browser)** ——为Android Firefox based浏览器提供了about:config支持，手动在Android上实践Arkenfox成为了可能

> ⚠️ 它的项目声明是：“No warranties or guarantees of security or updates or even stability!”

## 配置项

### GUI设置页
- 密码：
  + 保存密码：永不保存
  + 在Iceraven中自动填充：禁用
  + 在其他应用程序中自动填充：禁用
  + 同步密码：禁用
- 付款方式：
  + 保存和填写付款方式：禁用
  + 同步Card信息：禁用
- AI控制：
  + 屏蔽AI增强功能：启用
- HTTPS-Only模式
  + 启用，且对所有标签页启用
- 基于HTTPS的DNS：
  + 按需启用
- 增强型跟踪防护：
  + 自定义模式：
    - 修复严重网站问题：不勾选
    - 修复轻微网站问题：不勾选
    - Cookie：勾选且屏蔽所有第三方Cookie
    - 跟踪性内容：勾选且选择所有标签页
    - 加密货币挖矿程序：勾选
    - 已知的数字指纹追踪程序：勾选
    - 重定向追踪器：勾选
    - 存疑的数字指纹追踪程序：勾选且选择所有标签页
    - 要求网站不共享和不出售我的数据：建议不勾选
- 退出时删除浏览数据（使用菜单中的退出按钮触发）：开启，勾选Cookie和网站数据、缓存的图像和文件
- 拓展：默认收藏集提供了uBlock Origin，请自行安装
- 远程改进：禁用

### uBlock Origin
- uBlock Origin规则：
    - EasyPrivacy
    - AdGuard URL Tracking Protection
    - Peter Lowe’s Ad and tracking server list
    - uBlock filters – Ads
    - uBlock filters – Badware risks
    - uBlock filters – Privacy
    - uBlock filters – Quick fixes
    - uBlock filters – Unbreak
    - Online Malicious URL Blocklist
    - Phishing URL Blocklist
    - EasyList
    - AdGuard - Mobile Ads

### about:config
在地址栏访问about:config即可进入

没啥需要废话的，直接上列表：

|配置项|说明|小补充|
|-|-|-|
|privacy.resistFingerprinting = true|控制rFP是否启用|rFP的功能主要包括：<br /> - 修改时区为Atlantic/­Reykjavik（UTC+0）<br /> - canvas随机化<br /> - 地区报告为en-US<br /> - 仅预置字体<br /> - WebGL渲染器信息伪装<br /> - 其他指纹数据伪装|
|privacy.resistFingerprinting.block_mozAddonManager = true|禁用mozAddonManager API访问|防止读取拓展列表|
|media.peerconnection.ice.proxy_only_if_behind_proxy = true|强制WebRTC使用代理| - |
|media.peerconnection.ice.default_address_only = true|ICE只暴露一个接口|防止通过STUN获取实际IP|
|media.peerconnection.ice.no_host = true|排除私网IP|会破坏视频会议站点，不过一般不会在Android浏览器里使用网页视频会议|
|browser.cache.disk.enable = false|禁用硬盘缓存| - |
|browser.privatebrowsing.forceMediaMemoryCache = true|在隐私浏览模式中使媒体缓存于内存| - |
|media.memory_cache_max_size = 65536|内存缓存上限| - |
|browser.sessionstore.privacy_level = 2|禁止保存会话数据|0 = 任何情况都保存<br />1 = 仅未加密<br />2 = 从不|
|privacy.sanitize.sanitizeOnShutdown = true|关闭浏览器时清理|请使用Iceraven菜单中的“退出 Iceraven”触发|
|privacy.clearOnShutdown_v2.cache = true|关闭浏览器时清理缓存|同上|
|network.prefetch-next = false|禁用下一条目预获取| - |
|network.dns.disablePrefetch = true|禁用DNS预获取| - |
|network.dns.disablePrefetchFromHTTPS = true|禁用HTTPS DNS预获取| - |
|network.http.speculative-parallel-limit = 0|http预连接并行为0| - |
|browser.urlbar.speculativeConnect.enabled = false|禁用地址栏输入补全预连接| - |
|browser.places.speculativeConnect.enabled = false|禁用预连接| - |
|captivedetect.canonicalURL = ""|设置captive portal检查链接为空|默认值为firefox.com的captive portal检查链接，每次启动均会访问该链接|
|network.captive-portal-service.enabled = false|彻底禁用captive portal检查| - |
|network.connectivity-service.enabled = false|禁用连通性检查|连通性检查URL不要动，万一连通性检查被启用，那么它将使用设置的链接，可能导致出现连通性检查失败|
|dom.security.https_only_mode = true|HTTPS Anywhere| - |
|security.ssl.require_safe_negotiation = true|拒绝不支持 RFC 5746 的服务器|99.85% 的服务器均支持|
|security.tls.enable_0rtt_data = false|禁用0-RTT|TLS 1.3 0-RTT 有重放风险|
|security.cert_pinning.enforcement_level = 2|严格证书钉扎|强制公钥钉扎校验|
|network.http.referer.XOriginTrimmingPolicy = 2|跨源 referer 只发 scheme+host+port|挡掉路径与查询串泄漏|
|network.IDN_show_punycode = true|IDN 域名显示 Punycode|防 homograph 域名仿冒|
|network.proxy.socks_remote_dns = true|SOCKS 代理时 DNS 一并走代理|防代理下 DNS 直连泄漏|
|privacy.antitracking.isolateContentScriptResources = true|隔离 content script 的 referer/storage|FF139+|
|security.csp.reporting.enabled = false|关 CSP Level 2 上报|FF140+，违规报告不出站|
|browser.safebrowsing.malware.enabled = false|关恶意软件拦截|Safe Browsing 查询含数据上传|
|browser.safebrowsing.phishing.enabled = false|关钓鱼拦截|同上|
|browser.safebrowsing.blockedURIs.enabled = false|关被封 URI 拦截|同上|
|browser.safebrowsing.downloads.enabled = false|关下载扫描|同上|
|browser.safebrowsing.downloads.remote.enabled = false|关远程下载查证|同上|
|browser.safebrowsing.provider.google4.gethashURL = ""|清空 Safe Browsing v4 查询端点|开关关了 URL 也要清，杜绝残余请求|
|browser.safebrowsing.provider.google4.updateURL = ""|清空 Safe Browsing v4 更新端点|同上|
|browser.safebrowsing.provider.google.gethashURL = ""|清空 Safe Browsing v2 查询端点|同上|
|browser.safebrowsing.provider.google.updateURL = ""|清空 Safe Browsing v2 更新端点|同上|
|media.autoplay.blocking_policy = 2|自动播放需手动激活|2 = 点击播放，LibreWolf 同款默认|
|intl.accept_languages = "en-US,en"|伪装英语环境请求头|与 RFP 的 en-US 指纹对齐，避免语言与指纹矛盾|
|browser.formfill.enable = false|禁用表单填充历史|填充历史明文存于本地，无主密码加密|
|signon.autofillForms = false|禁用密码自动填充|填充过程可被页面脚本探测读取|
|signon.formlessCapture.enabled = false|禁用无表单登录捕获|formless 捕获会把非表单场景输入的凭据也存进去|
|network.auth.subresource-http-auth-allow = 1|subresource HTTP 认证仅同源|1 = 同源；跨源 subresource 不携带凭据|
|browser.uitour.enabled = false|关 UITour 后端|防远程页面触发浏览器 UI 控制|
|devtools.debugger.remote-enabled = false|关远程调试| - |
|browser.download.useDownloadDir = false|每次询问下载位置|防意外落盘|
|permissions.manager.defaultsUrl = ""|清空默认权限清单|移除 Mozilla 域名的特殊权限|
|pdfjs.enableScripting = false|PDF.js 禁脚本|PDF 内嵌 JS 是真实攻击面|

> browser.contentblocking.category = "strict" 不设：Iceraven 为 ETP 提供了额外的自定义模式，可以做到精细调控

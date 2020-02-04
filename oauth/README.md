# 3.3 OAuth 接入
本章介绍使用 OAuth2.0 协议接入 SEAC.适用对 Shibboleth 不太熟悉,需要快速完成接入,对 IdP 选择页面没有特殊需求的应用.
#### 简介
上海教育认证中心(SEAC) 也提供 OAuth2.0 接口服务向接入应用提供基于OAuth2.0的开放服务.通过对 Shibboleth 加以封装,以简化应用的接入.如下图所示:

![](https://eac.cloud.sh.edu.cn/images/oauth-shibboleth.png)

#### 接入模式
OAuth 可以分为认证中心接入和授权网关接入两种.
1. 认证中心接入 OAuth 时,使用统一的 SP 封装 OAuth 协议.所调用的 OAuth 接口为固定接口,IdP 所显示的 SP 名称和 SP Logo 均为"上海教育认证中心".选择 IdP 的页面(既 DS)不能定制修改.
2. 授权网关接入 OAuth 时,使用单独的 SP 封装 OAuth 协议,认证中心提供安装包和技术支持,由 SP 自行部署.可以自定义 SP 的名称和 Logo.如果对嵌入式 DS 有所了解,亦可自行修改定制 IdP 的选择页面.
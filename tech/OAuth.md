# 1.2.2 OAuth

#### OAuth 简介
OAuth标准协议是目前互联网业界普遍接受的认证授权体系。数十万开发商基于这些开放标准为新浪、腾讯、Facebook、Google等开放平台开发了大量应用，培育了一大批相关的开发者。 实现对OAuth的兼容可以大幅度增强信息化环境的开放性，成熟的产品和服务可以得到快速、便捷的整合，使得信息化建设的应用开发环境与互联网成熟的应用开发环境相一致。

#### 授权模型
OAuth 为客户端提供了一种代表资源拥有者访问受保护资源的方法。即 Authorization Code 模式的授权.他的工作流程如下图所示。

![](https://eac.cloud.sh.edu.cn/images/oauth-proccess.png)

流程如下：
（A）用户访问客户端，后者将前者导向认证服务器。

（B）用户选择是否给予客户端授权。

（C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。

（D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。

（E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

#### OAuth + Shibboleth
如果将换取授权码时,用于认证的服务替换为 Shibboleth,就能够将 OAuth 和 Shibboleth 结合在一起.结构如下图所示:

![](https://eac.cloud.sh.edu.cn/images/oauth-shibboleth.png)
# 注销配置

### 普通注销
如果用户以 `GET` 方式请求 `/profile/Logout` ，并且存在合法的会话的话，那么这个会话将被注销，用户会跳到注销页面（模板视图是 `logout.vm`），该页面会告诉用户以下事项：
  - IdP 的会话已经注销成功
  - 列出 IdP 会话期间所访问的所有 SP（如果开启了 trackSPSessions 的话），并向每个 SP 广播注销请求来结束这些会话

如果用户选择传播注销信息（即单点登出 `SLO`)，那么浏览器（通过前端）会向所有的 SP 发起注销请求，并返回注销的结果——红色的X表示失败，绿色的复选框则表示成功。这部分的模板视图是 `logout-propagate.vm`

如果用户不选择传播注销信息，那么会直接告诉用户已经注销，并提示用户没有注销掉所有的服务。这部分呢的模板视图是 `logout-complete.vm`

相关配置选项均在 `idp.properties` 中修改

#### track SP Sessions
IdP4 默认应该自动开启了 `trackSPSessions`，如果未开启请手动开启。
```
idp.session.trackSPSessions = true
```
#### HTML LocalStorage
由于默认情况下，idp 的 `session` 存储方式是基于 `cookie` 的，并且不支持 `track SP Sessions`。所以必须要开启其他的存储模式。可以使用 `HTML LocalStorage` ，或者使用服务端存储。

IdP4 默认应该自动开启了 `HTML LocalStorage` 如果未开启请手动开启。
```
idp.storage.htmlLocalStorage = true
```
#### UI 定制
如上文所述，涉及的模板试图主要是 `logout.vm` ,`logout-propagate.vm` , `logout-complete.vm` 可以根据需要进一步定制。

#### 浏览器支持
`SLO` 需要现代浏览器的支持，任何支持 `HTML5 LocalStorage` 的现代浏览器都可以支持。简单来说，不要用 `IE` 就好了。

#### CAS/OAuth2 注销
当 IdP 对接 CAS/OAuth2 时，我们需要在注销 IdP 的同时，也注销掉 CAS/OAuth2 上的会话。这部分可以通过在 IdP 的 `logout.vm` 中以不可见的 `iframe` 调用 CAS/OAuth2 的注销接口实现，比如:
```
<iframe style="display:none" src="https://cas.example.org/cas/logout"></iframe>
```

#### SAML 注销
前面所描述是通过 IdP 的注销接口注销。在实际使用中，用户可能更多会从 SP 侧发起注销。此时，如果要实现 `SLO` ，那么需要 IdP 开启相应的 `SAML SLO` 支持。默认情况下，这些 `endpoint` 是注释的，需要更新相应的 `Metadata`.

IdP4 已经默认开启了 SLO 的相关端点，对于从 IdP3 升级而来的 IdP4 服务器，你需要打开这部分端点，并向重新向联盟提交 `metadata`
取消 `metadata` 中 `SingleLogoutService` 部分的注释，并重新向联盟提交 `metadata`
```
<!--

        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://idp.xxx.edu.cn/idp/profile/SAML2/Redirect/SLO"/>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://idp.xxx.edu.cn/idp/profile/SAML2/POST/SLO"/>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST-SimpleSign" Location="https://idp.xxx.edu.cn/idp/profile/SAML2/POST-SimpleSign/SLO"/>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://idp.xxx.edu.cn:8443/idp/profile/SAML2/SOAP/SLO"/>
        
-->
```
然后确保开启下面的配置
```
idp.session.secondaryServiceIndex = true
```
同样的，基于 `cookie` 的 `session` 存储不支持 `SAML SLO`，请开启 `HTML LocalStorage` 模式

#### 非 SAML 模式的单点注销

由于 shibboleth 的默认的单点注销方案无法支持回调参数，我们建议 IdP 增加一段纯前端的本地代码来协助实现这个功能。

代码是开源的可供大家监督优化 - [idp-slo](https://github.com/shanghai-edu/idp-slo)

首先拉取项目
```
cd /opt
git clone https://github.com/shanghai-edu/idp-slo.git
```
在 Tomcat 的 `server.xml` 中增加映射配置
```xml
<Context path="logout" docBase="/opt/idp-slo/src/" debug="0" reloadable="true" crossContext="true"/>
```

在反向代理上也增加对应的配置，以 `nginx` 为例
```
      location /logout/ {
          proxy_pass      http://127.0.0.1:8080/logout/;
        }
```
# oauth 对接

#### 准备工作
- 下载相关的 [cas-client-core-3.6.0.jar
](https://eac.cloud.sh.edu.cn/download/cas-client-core-3.6.0.jar) 和 [shib-cas-authenticator-3.3.0-oauth.jar
](https://eac.cloud.sh.edu.cn/download/shib-cas-authenticator-3.3.0-oauth.jar)文件备用。
- 下载 [no-conversation-state.jsp](https://eac.cloud.sh.edu.cn/download/no-conversation-state.jsp) 文件备用
- IdP 版本至少为 3.4.6 以上

#### 直接安装
首先确保 IdP 是 3.4.6 以上版本​
- 把下载的 `no-conversation-state.jsp` 放入 `/opt/shibboleth-idp/edit-webapp` 中
- 把下载的 `cas-client-core-3.6.0.jar` 和 `shib-cas-authenticator-3.3.0-oauth.jar` 放入 `/opt/shibboleth-idp/edit-webapp/WEB-INF/lib` 中
- 将 `/opt/shibboleth-idp/dist/webapp/WEB-INF/web.xml` 拷贝到 `/opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml`

```
cp /opt/shibboleth-idp/dist/webapp/WEB-INF/web.xml /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml
```

- 修改 `/opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml` 增加以下部分

```
...
    <!-- Servlet for receiving a callback from an external CAS Server and continues the IdP login flow -->
    <servlet>
        <servlet-name>ShibCas Auth Servlet</servlet-name>
        <servlet-class>net.unicon.idp.externalauth.ShibcasAuthServlet</servlet-class>
        <load-on-startup>2</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>ShibCas Auth Servlet</servlet-name>
        <url-pattern>/Authn/External/*</url-pattern>
    </servlet-mapping>
...
```
- 修改 `idp.properties` 配置文件

```
idp.authn.flows=External
# 新增

# oauth 的地址
shibcas.oauth2UrlPrefix = https://oauth.xxx.edu.cn
# oauth 的 authorize 发起请求的 url
shibcas.oauth2LoginUrl = ${shibcas.oauth2UrlPrefix}/oauth/v1/authorize?response_type=code&client_id=${shibcas.oauth2clientid}&state=xyz
# idp 的地址
shibcas.serverName = https://idp.xxx.edu.cn
# oauth 换取 token 的地址
shibcas.oauth2TokenUrl = https://oauth.xxx.edu.cn/oauth/v1/token
# oauth 获取用户信息的地址
shibcas.oauth2ResourceUrl = https://oauth.xxx.edu.cn/oauth/v1/userinfo
# oauth 的 client_id
shibcas.oauth2clientid = testcient
# oauth 的 client_secret
shibcas.oauth2clientsecret = testpass
# redirect uri
shibcas.oauth2redirecturi = https://idp.xxx.edu.cn/idp/Authn/External?conversation=e1s1

# 新增
# redirect uri 的前缀
shibcas.oauth2redirecturiBase = https://idp.xxx.edu.cn/idp/Authn/External
# 返回属性中，标识用户名的字段
shibcas.oauth2principalname =  uid
```
- 运行 `/opt/shibboleth-idp/bin/build.sh` 重新编译 idp 即可
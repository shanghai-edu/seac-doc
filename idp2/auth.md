# 用户认证配置

idP 有多种认证方式，这里主要介绍两种：用户名+密码的认证方式和调用 CAS 认证的方式。

#### 用户名+密码认证

##### 开启用户名+密码认证
首先修改 ```IDP_HOME/conf/handle.xml``` 文件，开启 ```UsernamePassword``` 元素。
```
<!--  Username/password login handler -->
<ph:LoginHandler xsi:type="ph:UsernamePassword" jaasConfigurationLocation="file:///opt/idp/conf/login.config">
 	<ph:AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</ph:AuthenticationMethod>
</ph:LoginHandler>
```

注释掉除了 ```PreviousSession ```以外的所有 ```Loginhander```。（用于保持会话）

##### 登录页面修改

登录页面可通过修改 ```login.jsp ```和 ```login.css``` 文件来自定义。
文件位置在具体的 ```webapp``` 内，例如 ```/opt/apache-tomcat-6.0.36/webapps/idp/login.jsp```

##### 配置 ldap 认证

修改 ```IDP_HOME/conf/login.config``` ，例如
```
ShibUserPassAuth {
   edu.vt.middleware.ldap.jaas.LdapLoginModule required
      ldapUrl="ldap://ldap.example.org:389"
      ssl="false"
      tls="false"
      baseDn="ou=people,dc=example,dc=org"
      subtreeSearch="true"
      userFilter="uid={0}"
      bindDn="<ldapservicedn>"
      bindCredential="<password>";
};
```

#### 调用 CAS 认证

Shibboleth 支持 CAS 认证。详情可见 CAS 的官方文档 http://jasig.github.io/cas/4.0.x/integration/Shibboleth.html

下载最新版的 cas-client，并将 ```cas-client-$VERSION/modules/cas-client-core-$VERSION.jar``` 复制到 idp 安装目录的 ```lib/``` 当中。然后重新安装 idp

修改 ```IDP_HOME/conf/handler.xml``` , 开启 ```RemoteUser```

```
<!-- Remote User handler for CAS support -->
<LoginHandler xsi:type="RemoteUser">
  <AuthenticationMethod>
    urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified
  </AuthenticationMethod>
  <AuthenticationMethod>
    urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
  </AuthenticationMethod>
</LoginHandler>
```
修改 IDP war 所部署目录内的 ```web.xml```
例如 ```/opt/apache-tomcat-6.0.36/webapps/idp/WEB-INF/web.xml```

```
<!-- For CAS client support -->
<context-param>
  <param-name>serverName</param-name>
  <param-value>${idp.hostname}</param-value>
</context-param>
CAS Filters
<!-- CAS client filters -->
<filter>
  <filter-name>CAS Authentication Filter</filter-name>
  <filter-class>
      org.jasig.cas.client.authentication.AuthenticationFilter
  </filter-class>
  <init-param>
    <param-name>casServerLoginUrl</param-name>
    <param-value>${cas.server.url}login</param-value>
  </init-param>
</filter>
 
<filter-mapping>
  <filter-name>CAS Authentication Filter</filter-name>
  <url-pattern>/Authn/RemoteUser</url-pattern>
</filter-mapping>
  
<filter>
  <filter-name>CAS Validation Filter</filter-name>
  <filter-class>
    org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter
  </filter-class>
  <init-param>
    <param-name>casServerUrlPrefix</param-name>
    <param-value>${cas.server.url}</param-value>
  </init-param>
  <init-param>
    <param-name>redirectAfterValidation</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
  
<filter-mapping>
  <filter-name>CAS Validation Filter</filter-name>
  <url-pattern>/Authn/RemoteUser</url-pattern>
</filter-mapping>
  
<filter>
  <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
  <filter-class>
    org.jasig.cas.client.util.HttpServletRequestWrapperFilter
  </filter-class>
</filter>
  
<filter-mapping>
  <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
  <url-pattern>/Authn/RemoteUser</url-pattern>
</filter-mapping>
```

确认 RemoteUserHander 是开启的。

```
<!-- Servlet protected by container user for RemoteUser authentication -->
<servlet>
  <servlet-name>RemoteUserAuthHandler</servlet-name>
  <servlet-class>edu.internet2.middleware.shibboleth.idp.authn.provider.RemoteUserAuthServlet</servlet-class>
</servlet>
  
<servlet-mapping>
  <servlet-name>RemoteUserAuthHandler</servlet-name>
  <url-pattern>/Authn/RemoteUser</url-pattern>
</servlet-mapping>
```

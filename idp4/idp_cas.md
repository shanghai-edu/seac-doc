# 2.3.4 cas 对接

### 准备工作
- 使用 [Unicon/shib-cas-authn3](https://github.com/Unicon/shib-cas-authn3) 插件
- 下载相关的 [cas-client-core-3.6.0.jar
](https://eac.cloud.sh.edu.cn/download/cas-client-core-3.6.0.jar) 和 [shib-cas-authenticator-3.3.0.jar
](https://eac.cloud.sh.edu.cn/download/shib-cas-authenticator-3.3.0.jar)文件备用。
- 下载 [no-conversation-state.jsp](https://eac.cloud.sh.edu.cn/download/no-conversation-state.jsp) 文件备用
- IdP 版本至少 4

#### 直接安装
首先确保 IdP 版本为 4 以上
- 把下载的 `no-conversation-state.jsp` 放入 `/opt/shibboleth-idp/edit-webapp` 中
- 把下载的 `cas-client-core-3.6.0.jar` 和 `shib-cas-authenticator-3.3.0.jar` 放入 `/opt/shibboleth-idp/edit-webapp/WEB-INF/lib` 中
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
- 修改 idp.properties 配置文件

```
idp.authn.flows = External

shibcas.casServerUrlPrefix = https://cassserver.example.edu/cas
shibcas.casServerLoginUrl = ${shibcas.casServerUrlPrefix}/login

# idp 的地址
shibcas.serverName = https://idp.xxx.edu.cn

# 如果不支持 cas3.0 协议，这里修改为 cas20 并取消注释
# shibcas.ticketValidatorName = cas30
```
- 修改 conf/authn/external-authn-config.xml 配置文件

```
#将原先的external.jsp修改为Authn/External
<bean id="shibboleth.authn.External.externalAuthnPath" class="java.lang.String" c:_0="contextRelative:Authn/External" />
```
运行 /opt/shibboleth-idp/bin/build.sh 重新编译 idp 即可

#### 从 3.2.3 升级
首先确认 idp 已经升级到 3.4.6
- 修改 `web.xml` ，调整为如下

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
- 修改 `idp.properties` 配置文件，将 flow 修改为 External  `idp.authn.flows=External` 
- 删除 `/opt/shibboleth-idp/conf/authn/general-authn.xml` 中新增的 bean 内容，例如：
  
```
        <bean id="authn/Shibcas" parent="shibboleth.AuthenticationFlow"
                p:passiveAuthenticationSupported="true"
                p:forcedAuthenticationSupported="true"
                p:nonBrowserSupported="false" />
```
- 删除 `/opt/shibboleth-idp/flows/authn/Shibcas/` 文件夹
- 删除 `/opt/shibboleth-idp/edit-webapp/WEB-INF/lib` 中老版本的 jar 包，更新为`cas-client-core-3.6.0.jar` 和 `shib-cas-authenticator-3.3.0.jar` 
- 运行 `/opt/shibboleth-idp/bin/build.sh` 重新编译 idp 即可

##### 配置属性释放
`xsi:type="SubjectDerivedAttribute"` 为从插件中获取属性的配置，例如下面的示例的映射关系如下所示：
- ID_NUMBER -> uid
- USER_NAME -> cn
- TYPE_NAME -> typeOf

```xml
     <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="uid" principalAttributeName="ID_NUMBER">
         <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:uid" encodeType="false" />
         <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.2" friendlyName="uid" encodeType="false" />
     </AttributeDefinition>

   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="cn" principalAttributeName="USER_NAME">
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:cn" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.3" friendlyName="cn" encodeType="false" />
   </AttributeDefinition>

   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="typeOf" principalAttributeName="TYPE_NAME">
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:typeOf" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.100.2" friendlyName="typeOf" encodeType="false" />
   </AttributeDefinition>
```

##### 重新编译war文件，并且重启tomcat：

```
[root@www ~]# cd /opt/shibboleth-idp/bin
[root@www ~]# ./build.sh

Installation Directory: [/opt/shibboleth-idp] #enter

Rebuilding /opt/shibboleth-idp/war/idp.war ...

...done

BUILD SUCCESSFUL

Total time: 3 seconds

[root@www ~]# systemctl restart tomcat
```



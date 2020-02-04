# 2.3.3 cas 对接

### 准备工作
- [下载](https://github.com/shanghai-edu/shib-cas-authn3/releases/tag/3.2.3) cas 插件所需的各项资料。
- IdP 的版本不能超过 3.4.3

#### 具体配置
##### 新建文件夹，获取并拷贝相关文件：

```
[root@www ~]# wget https://github.com/shanghai-edu/shib-cas-authn3/releases/download/3.2.3/shib-cas-authenticator-3.2.3.tar.gz
[root@www ~]# tar -zxvf shib-cas-authenticator-3.2.3.tar.gz
[root@www ~]# mkdir /opt/shibboleth-idp/flows/authn/Shibcas/
```
​        把附件中shibcas-authn-beans.xml、shibcas-authn-flow.xml放入文件夹中。

​        把附件中的no-conversation-state.jsp放入/opt/shibboleth-idp/edit-webapp中。

​        把附件中的shib-cas-authenticator-3.2.3.jar、cas-client-core-3.4.1.jar放入/opt/shibboleth-idp/edit-webapp/WEB-INF/lib中。

​        如shibboleth路径为默认（/opt/shibboleth-idp），可以在附件根目录使用如下命令

```
cp shibcas-authn-beans.xml /opt/shibboleth-idp/flows/authn/Shiboauth2/shibcas-authn-beans.xml

cp shibcas-authn-flow.xml /opt/shibboleth-idp/flows/authn/Shiboauth2/shibcas-authn-flow.xml

cp no-conversation-state.jsp /opt/shibboleth-idp/edit-webapp/no-conversation-state.jsp

cp shib-cas-authenticator-3.2.3.jar /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/shib-cas-authenticator-3.2.3.jar

cp cas-client-core-3.4.1.jar /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/cas-client-core-3.4.1.jar
```

##### 配置web.xml：

```
[root@www ~]# cp /opt/shibboleth-idp/dist/webapp/WEB-INF/web.xml /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml
[root@www ~]# vi /opt/shibboleth-idp/edit-webapp/WEB-INF/web.xml

# 在<!-- Servlets and servlet mappings -->后加上

<!-- Servlet for receiving a callback from an external CAS Server and continues the IdP login flow -->
     <servlet>
         <servlet-name>ShibCas Auth Servlet</servlet-name>
         <servlet-class>net.unicon.idp.externalauth.ShibcasAuthServlet</servlet-class>
         <load-on-startup>2</load-on-startup>
     </servlet>
     <servlet-mapping>
         <servlet-name>ShibCas Auth Servlet</servlet-name>
         <url-pattern>/Authn/ExtCas/*</url-pattern>
</servlet-mapping>
```

##### 配置idp.properties：
```
[root@www ~]# vi /opt/shibboleth-idp/conf/idp.properties
```

```
# 修改

idp.authn.flows = Shiboauth2

#新增
shibcas.casServerUrlPrefix = https://xxx.xxx.xxx.xxx/cas  #CAS服务器域名
shibcas.casServerLoginUrl = ${shibcas.casServerUrlPrefix}/login
shibcas.serverName = https://xxx.xxx.xxx.xxx #IdP的域名 
```
###### CAS2.0 兼容
CAS 插件只支持 CAS3.0 协议，即 ticket 校验接口为 https://xxx.xxx.xxx.xxx/cas/p3/serviceValidate

因此如果 CAS 协议只支持 2.0, 即 ticket 校验接口为 https://xxx.xxx.xxx.xxx/cas/serviceValidate

可以选择加一层代理作为封装，来伪装成 CAS3.0 的校验接口。并将 shibcas.serverName 和 shibcas.casServerUrlPrefix 配置为代理的地址，并将shibcas.casServerLoginUrl 直接配置为 cas 的 login 地址

类似 nginx 的代理方式如下所示：
```nginx
      location /cas/p3 {
              proxy_pass      http://xxxxxx/cas;
              proxy_set_header            X-real-ip $remote_addr;
              proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
      }
```

##### 配置general-authn.xml：

```
[root@www ~]# vi /opt/shibboleth-idp/conf/authn/general-authn.xml

# 在<util:list id="shibboleth.AvailableAuthenticationFlows">后新增

<bean id="authn/Shiboauth2" parent="shibboleth.AuthenticationFlow"
                 p:passiveAuthenticationSupported="true"
                 p:forcedAuthenticationSupported="true"
                 p:nonBrowserSupported="false" />
```

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



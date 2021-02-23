# 配置

## 认证对接
### ldap
详见 [LDAP 对接文档](https://eac.cloud.sh.edu.cn/document/idp4/idp_auth.html), 重新配置各方面的配置文件即可。

注意，LDAP 的链接账号的密码现在 `/opt/shibboleth-idp/credentials/secrets.properties` 里配置。

`secrets.properties`
```ini
# Default access to LDAP authn and attribute stores. 
idp.authn.LDAP.bindDNCredential              = myServicePassword
idp.attribute.resolver.LDAP.bindDNCredential = %{idp.authn.LDAP.bindDNCredential:undefined}
```
### CAS
详见 [CAS 对接文档](https://eac.cloud.sh.edu.cn/document/idp4/idp_cas.html), 重新配置各方面的配置文件并重新编译即可。

### OAuth2
详见 [Oauth2 对接文档](https://eac.cloud.sh.edu.cn/document/idp4/idp_oauth.html), 重新配置各方面的配置文件并重新编译即可。

## 属性获取
详见 [数据连接器](https://eac.cloud.sh.edu.cn/document/idp4/dataconnector.html) 及 [属性定义](https://eac.cloud.sh.edu.cn/document/idp4/attributes.html)

完整的配置示例详见 [CARSI-IdP 接入](https://eac.cloud.sh.edu.cn/document/idp4/idp_carsi.html)

其中 `eptid` 部分，建议升级时切换至数据库存储模式，以便于需要时提供追查支持。

文档详见 [eptID](https://eac.cloud.sh.edu.cn/document/idp4/eptID.html)

## 属性释放
详见 [属性释放配置](https://eac.cloud.sh.edu.cn/document/idp4/idp_filter.html)，重新配置各方面配置文件即可。

建议选择同步上海教育认证中心所推荐的属性释放策略，从而在不产生运维负担的前提下，保障属性释放的安全可控。

## 单点注销
IdP4 已经默认开启单点注销，但是如果您是从 IdP3 升级而来，则之前所提交给联盟的 `metadata` 中应该没有打开单点注销部分的 `endpoint`。您需要取消掉这部分的注销，并重新提交 `metadata` 文件到联盟。
```xml
<!--

        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://idp.xxx.edu.cn/idp/profile/SAML2/Redirect/SLO"/>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://idp.xxx.edu.cn/idp/profile/SAML2/POST/SLO"/>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST-SimpleSign" Location="https://idp.xxx.edu.cn/idp/profile/SAML2/POST-SimpleSign/SLO"/>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://idp.xxx.edu.cn:8443/idp/profile/SAML2/SOAP/SLO"/>

-->
```
同时建议开启非 SAML 模式的单点注销模块，以更好的支持单点注销环境下的注销请求回调需求。
更多详细信息可以参考 [单点注销](https://eac.cloud.sh.edu.cn/document/idp4/idp_logout.html)

## Metadata
### 配置 metadata-providers.xml
参照原来的配置文件，重新配置 `/opt/shibboleth-idp/conf/metadata-providers.xml` 配置文件。如果有

### 迁移证书
备份原 IdP 服务器上 `/opt/shibboleth-idp/credentials` 目录内的所有文件
```
cd /opt/shibboleth-idp/credentials/
tar -czvf credentials.tar.gz *.*
```
将 `credentials.tar.gz` 下载并上传到新服务器的`/opt/shibboleth-idp/credentials` 目录内，解压替换目录内的同名文件。
```
tar -zxvf credentials.tar.gz
rm -rf credentials.tar.gz 
```

如果在 IdP4 的安装过程中，证书密码和 cookie 密码与原服务器不一致。则需要修改 `/opt/shibboleth-idp/credentials/secrets.properties` 配置文件，确保与原服务器 `/opt/shibboleth-idp/conf/idp.properties` 中的配置保持一致。
```ini
# Access to internal AES encryption key
idp.sealer.storePassword=password
idp.sealer.keyPassword=password
```

替换完成后，重新赋予 tomcat 用户权限。
```
chown -R tomcat.tomcat /opt/shibboleth-idp/credentials
```

删掉最初安装时生成的 `idp-metadata.xml`，并替换为原 IdP 服务器上的 `idp-metadata.xml` 文件
- 实际上现在 IdP4 并不会读取自己的 Metadata，所以不换也能用。但是会导致 `https://idp.ecnu.edu.cn/idp/shibboleth` 接口 404，影响一些外部的管理维护，还是放上去的比较好。

## 其他配置
### auditlog
详见 [访问日志](https://eac.cloud.sh.edu.cn/document/idp4/auditlog.html)
#### carsi-auditlog
如果需要将 auditlog 提交给 CARSI，则修改 `/opt/shibboleth-idp/conf/audit.xml` 配置文件
```xml
    <!--
    This bean defines a mapping between audit log categories and formatting strings.
    -->
    <util:map id="shibboleth.AuditFormattingMap">
        <entry key="Shibboleth-Audit" value="%T|%b|%I|%SP|%P|%IdP|%bb|%III|%u|%ac|%attr|%n|%i|%a|%s|" />
    </util:map>

    <!-- Override the format of date/time fields in the log and/or convert to default time zone. -->

    <bean id="shibboleth.AuditDateTimeFormat" class="java.lang.String" c:_0="YYYY-MM-dd'T'HH:mm:ss.SSSZZ" />
    <util:constant id="shibboleth.AuditDefaultTimeZone" static-field="java.lang.Boolean.TRUE" />
```
创建供给 CARSI 读取的日志目录
```
mkdir /opt/auditlog
```
创建日志同步脚本 `vi /opt/auditlog.sh`, 内容如下
```
rm -rf /opt/auditlog/auditlog-`date -d -24hours +%Y-%m-%d-%H`.log
grep `date -d -1hours +%Y-%m-%dT%H` /opt/shibboleth-idp/logs/idp-audit.log > /opt/auditlog/auditlog-`date -d -1hours +%Y-%m-%d-%H`.log
```
`crontab -e`创建定时同步任务
```
0 */1 * * * sh /opt/auditlog.sh >/dev/null 2>&1
```
修改 Tomcat 的 `server.xml` 配置文件，挂载静态文件
```xml
        <Context path="/auditlog/" debug="0" docBase="/opt/auditlog/" reloadable="true">
                <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127.0.0.1|115.27.243.6" deny="" denyStatus="403"/>
        </Context>
```
修改 nginx 或其他反向代理的配置，代理此静态目录。此处的 ACL 和 Tomcat 中的 ACL，配一个地方即可。
```
      location /auditlog/ {
        proxy_pass      http://127.0.0.1:8080/auditlog/;
        allow 115.27.243.6;
        deny all;
      }
```
## UI 自定义
详见 [自定义 UI](https://eac.cloud.sh.edu.cn/document/idp4/ui.html)，重新配置各方面配置文件即可。

### 防暴力破解
详见 [防暴力破解](https://eac.cloud.sh.edu.cn/document/idp4/account_lock.html), 重新配置各方面的配置文件即可。

### IdP 监控
不再支持 `idp/profile/status` 和 `idp/status` 接口，现在统一为 `json` 格式化的 `metric` 指标接口，访问 `http://127.0.0.1:8080/idp/profile/admin/metrics` 即可获取到相关监控指标。

更多详细信息可以参考 [IdP的监控](https://eac.cloud.sh.edu.cn/document/idp4/monitor.html)


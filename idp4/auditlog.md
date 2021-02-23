# 访问日志

### 日志文件

IdP 会在 `/opt/shibboleth-idp/logs` 目录下生成下列日志文件

- idp-process.log：最全，用来 debug
- idp-warn.log：错误和警告
- idp-consent-audit.log：隐私保护的属性释放情况
- idp-audit.log：IdP 访问日志

其中 `idp-audit.log` 是每次用户完成认证后产生的记录，有比较大的数据分析价值。可以通过修改 `/opt/shibboleth-idp/conf/audit.xml` 来配置日志的格式。

### auditlog
默认的日志格式为 
```xml
    <util:map id="shibboleth.AuditFormattingMap">
        <entry key="Shibboleth-Audit" value="%a|%ST|%T|%u|%SP|%i|%ac|%t|%attr|%n|%f|%SSO|%XX|%XA|%b|%bb|%e|%S|%SS|%s|%UA" />
    </util:map>
```
这其中各项参数的详细含义，可以参考 [AuditLoggingConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/AuditLoggingConfiguration)

通常我们关注 `%a` - IP地址，`%T` - 认证完成的时间，`%u` - 用户名，`%SP` - SP，`%UA` - 浏览器的 User-Agent 就可以做比较完善的数据分析了。

以下是一条实际的对应日志
```
172.20.156.241|2021-02-23T11:57:26.925677Z|2021-02-23T11:57:38.078869Z|username|https://whoami.cloud.sh.edu.cn/shibboleth|_6dc96dc77f516174d881bb90c12976f3|password|2021-02-23T11:57:31.542923Z|eduPersonEntitlement,uid,eduPersonScopedAffiliation,eduPersonTargetedID,domainName,cn,typeOf|AAdzZWNyZXQxk7bLrVhziSVV6SWyVnPIa4NvuqV+zpAcTGyPHyCMiHkWjihvtDOcdcZ3+N1r2n+zINOFpCKiqMNo8+EfG017VPPvnH38RVX2XAZobgGYDNts3FJRHDGTUEcjX4vl91ojHhAmGf23BNTpEriBhOPdzw==|transient|false|false|AES128-GCM|Redirect|POST||Success||35de7df990f2aeb9f78c69f1cf8d3f386233817f452e69b892ef3170f39ed820|Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36 Edg/88.0.705.74
```
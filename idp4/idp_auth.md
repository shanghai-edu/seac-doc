# 认证配置和属性对接-LDAP

本文中的 `idp.home` 均指代实际的 idp 安装路径。例如 `/opt/shibboleth-idp`

### LDAP 认证配置
修改 `/idp.home/conf/ldap.properties`
```bash
# ldap 的认证模式，通常使用 bindSearchAuthenticator。即通过管理员账号 bindDN 和 bindDNCredential，根据 userFilter 先搜所出用户的 dn，再和用户的密码进行验证
idp.authn.LDAP.authenticator = bindSearchAuthenticator
idp.authn.LDAP.ldapURL = ldap://ldap服务器IP地址
idp.authn.LDAP.useStartTLS  = false
idp.authn.LDAP.sslConfig = jvmTrust
# 开始搜索的 basedn
idp.authn.LDAP.baseDN = ou=People,dc=Test,dc=Test
# 有搜索权限的管理员账号
idp.authn.LDAP.bindDN = cn=Manager,dc=Test,dc=Test
# 管理员账号的密码
idp.authn.LDAP.bindDNCredential = 密码
# 允许递归搜索子树
idp.authn.LDAP.subtreeSearch = true
# 即用户名所对应的 ldap 属性。例如在 ldap 中通常是 uid，在 ad 中则需要修改为 (sAMAccountName={user})
idp.authn.LDAP.userFilter = (uid={user})
# 类似上一条，在 ad 中需要修改为 (sAMAccountName=$resolutionContext.principal)
idp.attribute.resolver.LDAP.searchFilter = (uid=$resolutionContext.principal)
```

### LDAP 密码配置
由于idp4在安装时会默认创建credentials/secrets.properties文件，此文件会导致ldap.properties中对于ldap密码的设置被覆盖,所以需要修改secrets.properties中对应的密码项
修改 `/idp.home/credentials/secrets.properties`

```
idp.authn.LDAP.bindDNCredential=密码
```

#### LDAP 测试

由于 ldap 配置非常关键，建议先通过工具，在 IdP 环境上测试以下 LDAP 的认证，属性等是否均正常。以简化后续的排错判断。可以使用 [ldap-test-tool](https://github.com/shanghai-edu/ldap-test-tool) 进行测试

##### 安装 ldap-test-tool

[下载](https://github.com/shanghai-edu/ldap-test-tool/releases) ldap-test-tool 工具，选择对应的版本

解压后，修改 `cfg.json` 文件，填写 ldap 配置.`attributes` 指返回的属性，如果留空的话，则表示返回所有能读到的属性
```
{
    "ldap": {
        "addr": "ldap.example.org:389",
        "baseDn": "dc=example,dc=org",
        "bindDn": "cn=manager,dc=example,dc=org",
        "bindPass": "password",
        "authFilter": "(&(uid=%s))",
        "attributes": ["uid", "cn", "mail"],
        "tls":        false,
        "startTLS":   false
    },
    "http": {
        "listen": "0.0.0.0:8888"
    }
}
```

##### 查询测试
```
# ./ldap-test-tool search user qfeng
LDAP Search Start 
==================================


DN: uid=qfeng,ou=people,dc=example,dc=org
Attributes:
 -- uid  : qfeng 
 -- cn   : 冯骐测试 
 -- mail : qfeng@example.org


==================================
LDAP Search Finished, Time Usage 44.711268ms 
```

##### 认证测试
```
./ldap-test-tool auth single qfeng 123456
LDAP Auth Start 
==================================

qfeng auth test successed 

==================================
LDAP Auth Finished, Time Usage 47.821884ms 
```

更多信息请访问 [ldap-test-tool](https://github.com/shanghai-edu/ldap-test-tool) 查看
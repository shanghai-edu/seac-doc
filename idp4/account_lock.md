# 防暴力破解

shibboleth idp3.3以上版本支持账号锁定的功能，当账号错误登录达到一定次数时，自动锁定账号一段时间。

修改 `conf/authn/password-authn-config.xml` 配置文件，取消 `<bean id="shibboleth.authn.Password.AccountLockoutManager"` 这个 `bean` 的注释

```xml
<bean id="shibboleth.authn.Password.AccountLockoutManager"
parent="shibboleth.StorageBackedAccountLockoutManager"
p:maxAttempts="5"
p:counterInterval="PT5M"
p:lockoutDuration="PT5M"
p:extendLockoutDuration="false" />
```

这部分配置表示，5 分钟(`p:counterInterval`)内连续密码错误 5 次(`p:maxAttempts`)，则锁定 5 分钟(`p:lockoutDuration`)。
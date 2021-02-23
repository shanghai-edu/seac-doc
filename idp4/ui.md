# 自定义 UI

#### 汉化
首先下载国际化文件，放到 `/opt/shibboleth-idp/messages` 目录内。
```
cd /opt/shibboleth-idp/messages
wget https://wiki.shibboleth.net/confluence/download/attachments/55804430/messages_zh_CN.properties
```
重点修改这部分配置
```ini
idp.title=Web登陆服务
idp.title.suffix=错误
idp.logo=/images/你的单位logo.png
idp.logo.alt-text=你的单位名称
idp.message=发生了未知错误
idp.footer=Copyright信息
root.title=Shibboleth IdP
root.message=无可用服务
root.footer=Copyright信息

idp.url.password.reset=忘记密码的链接
idp.url.helpdesk=需要帮助的链接

# 默认的文本会降低暴力破解的难度，建议修改为用户名或密码不正确
bad-username.message=用户名或密码不正确。
bad-password.message=用户名或密码不正确。
account-locked.message=由于密码多次错误，此帐户已被暂时锁定。

# 这两部分是 IdP 出错时页面的文本。默认没有汉化，建议按此模板汉化之以提高用户体验。
stale.title=过期的请求
stale.message=<p>显示该页面是由于您在认证过程中可能使用了后退按钮。或者，您可能错误的将登录页面添加了书签，而不是您想要添加书签的实际网站，或者访问了由犯同样错误的其他人创建的链接。</p> <br/> <p>此时由于会话无法匹配将会产生错误，请关闭浏览器重新打开后，从实际资源的网站发起登录。</p>

runtime-error.title=未被捕获的异常
runtime-error.message=<p>系统出现错误\:</p><br/> <p><strong>\#if($exception)$encoder.encodeForHTML($exception.toString())\#elseif($flowExecutionException && $flowExecutionException.getCause())$encoder.encodeForHTML($flowExecutionException.getCause().toString())\#else$encoder.encodeForHTML($flowExecutionException.getMessage())\#end</strong></p><br/> <p>请联系xxxxx反馈错误：xxx@xxx.edu.cn</p>
```
其中 `logo` 部分，请上传至 `/opt/shibboleth-idp/edit-webapp/images`，并重新编译 `idp` 后，重启 `Tomcat` 生效

```
/opt/shibboleth-idp/bin/build.sh 
systemctl restart tomcat
```
#### 深度定制
如果需要针对 UI 做深度定制，则可以修改 `/opt/shibboleth-idp/views` 下的各种 `*.vm` 文件即可。




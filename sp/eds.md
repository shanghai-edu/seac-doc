# 3.1.4 嵌入式 DS

#### 环境准备
同 shibboleth-sp 安装
#### yum 安装 eds
```yum install shibboleth-embedded-ds```

#### 安装目录

默认会安装到 ```/etc/shibboleth-ds```
同时会自动配置 apache ，对此目录下的 5 个文件路径别名
```
# Basic Apache configuration

<IfModule mod_alias.c>
  <Location /shibboleth-ds>
    Allow from all
    <IfModule mod_shib.c>
      AuthType shibboleth
      ShibRequestSetting requireSession false
      require shibboleth
    </IfModule>
  </Location>
  Alias /shibboleth-ds/idpselect_config.js /etc/shibboleth-ds/idpselect_config.js
  Alias /shibboleth-ds/idpselect.js /etc/shibboleth-ds/idpselect.js
  Alias /shibboleth-ds/idpselect.css /etc/shibboleth-ds/idpselect.css
  Alias /shibboleth-ds/index.html /etc/shibboleth-ds/index.html
  Alias /shibboleth-ds/blank.gif /etc/shibboleth-ds/blank.gif
</IfModule>
```

#### 使用

在网站中嵌入 js 和 css 文件。
```
<link rel="stylesheet" type="text/css" href="idpselect.css">
```
```
<div id="idpSelect"></div>
```
```
<!-- Load languages scripts -->
<script src="idpselect_config.js" type="text/javascript" language="javascript"></script>
<script src="idpselect.js" type="text/javascript" language="javascript"></script>
```

默认提供了一个 index.html ，供测试

#### SP 配置

只需讲 SP 的 SSO 重定向指向嵌入式 DS 所在的 web 页即可，以默认的 index.html 为例
```
<SSO discoveryProtocol="SAMLDS" discoveryURL="https://sp.example.org/shibboleth-ds/index.html">
   SAML2 SAML1
</SSO>
```

#### 定制化
由于默认提供的 js 过于简陋且不易修改.通常建议自行定制开放对应的页面. DS 本身只是提供一个跳转链接,所供选择的列表由 SP 的以下接口产生 - https://sp.example.og/Shibboleth.sso/DiscoFeed
```
[
{
 "entityID": "https://idp.tongji.edu.cn/idp/shibboleth",
 "DisplayNames": [
  {
  "value": "03:同济大学",
  "lang": "zh-CN"
  }
 ]
},
{
 "entityID": "https://idp.ecust.edu.cn/idp/shibboleth",
 "DisplayNames": [
  {
  "value": "05:华东理工大学",
  "lang": "zh-CN"
  }
 ]
},
{
 "entityID": "https://idp.dhu.edu.cn/idp/shibboleth",
 "DisplayNames": [
  {
  "value": "07:东华大学",
  "lang": "zh-CN"
  }
 ]
}
.....
```
建议应用自行获取后,进行二次开发定制自己的嵌入式 DS
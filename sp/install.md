# 3.1.1 SP 安装

#### 环境准备
1. 由于安装方式采用 yum ,因此尽量选择 `centos` 或 `suse`. 本文以 Centos7 为例
2. 修改 /etc/hostname
将 localhost.localdomain 改成最终域名，例如 sp.example.org`sp.examle.org`
3. 修改 `/etc/selinux/config`,设置`SELINUX=disabled` 关闭 `selinux`,否则可能导致权限问题无法认证,然后重启服务器
4. 开放服务器的 80 和 443 端口
5. 确保时间同步正常

### 安装
#### yum 安装
```
# wget http://download.opensuse.org/repositories/security://shibboleth/CentOS_7/security:shibboleth.repo -P /etc/yum.repos.d
yum install shibboleth httpd mod_ssl -y
```
#### apache 配置
##### SSL 配置
安装好证书,可参考 lets encrypt
##### 设置 shib 保护路径 
修改 ```/etc/httpd/conf.d/shib.conf``` 文件
`/secure` 表示被 Shibboleth 保护的路径，访问此路径将会进行跨校认证。
```
<Location /secure>
  AuthType shibboleth
  ShibCompatWith24 On
  ShibRequestSetting requireSession 1
  require shib-session
</Location>
```

#### 配置 shibboleth
修改 `/etc/shibboleth/shibboleth2.xml` 文件,主要修改以下部分

修改 ```entityID```
```
<ApplicationDefaults entityID="https://sp.example.org/shibboleth"
         REMOTE_USER="eppn persistent-id targeted-id">
```
修改 ```SSO```
```
<SSO discoveryProtocol="SAMLDS" discoveryURL="https://ds.shec.edu.cn/ds/WAYF">
  SAML2 SAML1
</SSO>shec.edu.cn
```
如果使用嵌入式 DS 此处请参考嵌入式 DS 部分

修改 ```MetadataProvider```
```
<MetadataProvider type="XML" uri="https://ds.shec.edu.cn/metadata.xml"    
 backingFilePath="metadata.xml" legacyOrgNames="true" reloadInterval="600"/>
```

### 服务
启动服务
`systemctl start shibd `
`systemctl start httpd `

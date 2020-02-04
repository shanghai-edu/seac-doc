# 2.4 SSL 证书

由于几大证书商已经普遍不再签发 edu.cn 的免费证书,因此如若想使用免费证书,则只能选择 Lets encrypt 方案.

本章介绍一个基于 `shell` 脚本的 lets encrypt 自动续签方案,没有依赖,在不同服务器环境下比较通用

### lets encrypt 方案
#### 下载签发脚本
[下载地址](https://github.com/lukas2511/dehydrated/releases)
统一安装在 `/home` 路径上，重命名为  `/home/letsencrypt`

#### 续签证书
##### 建校验目录
`mkdir /var/www/dehydrated`
##### 映射校验 url 到校验目录
###### apache
修改 `/etc/httpd/conf/http.conf` 增加下面内容
```
Alias /.well-known/acme-challenge /var/www/dehydrated
<Directory /var/www/dehydrated>
        Options None
        AllowOverride None

        # Apache 2.x
        <IfModule !mod_authz_core.c>
                Order allow,deny
                Allow from all
        </IfModule>

        # Apache 2.4
        <IfModule mod_authz_core.c>
                Require all granted
        </IfModule>
</Directory>
```
###### nginx
```
  location ^~ /.well-known/acme-challenge {
    alias /var/www/dehydrated;
  }
```
##### 重启服务
```
service httpd restart
```
or
```
service nginx restart
```
##### 签发证书
如果网络失败之类的，重复多试几次。
```
/home/letsencrypt/dehydrated --register --accept-terms
/home/letsencrypt/dehydrated -c -d idp.xxx.edu.cn
```
##### 部署证书
###### 修改证书路径-apache
修改 `/etc/httpd/conf.d/ssl.conf` 中下面内容，即证书路径更改 letsencryt 中的证书
```
SSLCertificateFile /home/letsencrypt/certs/idp.xxx.edu.cn/fullchain.pem
SSLCertificateKeyFile /home/letsencrypt/certs/idp.xxx.edu.cn/privkey.pem
```
###### 修改证书路径-nginx
修改对应配置文件中下面内容，即证书路径更改 letsencryt 中的证书
```
ssl_certificate /home/letsencrypt/certs/idp.xxx.edu.cn/fullchain.pem;
ssl_certificate_key /home/letsencrypt/certs/idp.xxx.edu.cn/privkey.pem;
```
###### 重启 apahce 服务
```
service httpd restart
```
or
```
service nginx restart
```

##### 自动续签脚本
crontab -e  中增加下面内容，每天尝试更新续签。
```
15 12 * * * (/home/letsencrypt/dehydrated -c -d idp.xxx.edu.cn) > /tmp/lets_encrypt.log 2>&1
15 4 * * * (/sbin/service httpd restart) > /tmp/http.log 2>&1
```
##### 注意事项
部署中注意替换 idp.xxx.edu.cn 为学校服务器的实际域名
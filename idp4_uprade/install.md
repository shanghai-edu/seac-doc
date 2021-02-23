# 安装

## 环境
本文档以 Centos7 + openjdk 为例。

IdP 安装要求详见官网（https://wiki.shibboleth.net/confluence/display/IDP4/SystemRequirements）

### 防火墙
打开 80 和 443 端口
```
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```
### 时间同步
上海教育城域网环境建议使用 `ntp.shec.edu.cn`，校校通专网环境建议使用 `ntp.edu.sh.cn`
```
# chronyc
chrony version 3.4
Copyright (C) 1997-2003, 2007, 2009-2018 Richard P. Curnow and others
chrony comes with ABSOLUTELY NO WARRANTY.  This is free software, and
you are welcome to redistribute it under certain conditions.  See the
GNU General Public License version 2 for details.

chronyc> sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 202.120.161.137               1   6    77     8   +113us[ +202us] +/- 2456us
chronyc> 
```
### java 环境
安装 java 11
```
yum -y install java-11-openjdk java-11-openjdk-devel
java -version
```
### 基础工具
安装 `wget`,`lrzsz`,`unzip`,`git`
```
yum -y install wget lrzsz unzip git
```

## web中间件
### tomcat
安装 [Tomcat(https://tomcat.apache.org/download-90.cgi)
```
sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
cd /tmp
wget https://mirror.bit.edu.cn/apache/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz
tar -zxvf apache-tomcat-9.0.41.tar.gz
mv apache-tomcat-9.0.41 /opt/tomcat
sudo ln -s /opt/tomcat/apache-tomcat-9.0.41 /opt/tomcat/latest
chown -R tomcat: /opt/tomcat
sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'
```

修改 `/opt/tomcat/latest/conf/server.xml` 文件，在 `<Engine>` 标签内增加反向代理的相关配置
```xml
      <Valve className="org.apache.catalina.valves.RemoteIpValve"
             internalProxies="127.0.0.1|::1"
             remoteIpHeader="x-forwarded-for"
             protocolHeader="x-forwarded-proto"
      />
    </Engine>
  </Service>
</Server>
```

注册 systemctl , `tomcat.service` 的内容如下所示。其中 `CATALINA_OPTS` 部分的 `JVM` 参数建议根据服务器实际内存大小和负载情况在灵活调整。
```ini
[Unit]
Description=Tomcat 9 servlet container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/jre"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"

Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms1024M -Xmx2048M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

注册为 systemctl
```
cp tomcat.service /etc/systemd/system/tomcat.service
systemctl daemon-reload
sudo systemctl enable tomcat
sudo systemctl start tomcat
```

### 反向代理
我们建议将 IdP 通过反向代理发布，以获得更好的性能和安全性

此处以 `nginx` 为例，实际环境如果有商业产品，建议使用商业解决方案以获得更好的技术支持。

### nginx
安装 [nginx](https://nginx.org/en/linux_packages.html)，创建 `yum repo`

`/etc/yum.repos.d/nginx.repo`
```ini
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
安装 
```
yum install -y nginx
systemctl enable nginx
systemctl start nginx
```

`nginx.conf` 配置优化 - 供参考
```nginx
user nginx;
worker_processes  auto;
worker_rlimit_nofile 65535;


error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  2048;
    multi_accept on;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    # json 化日志，如搭配 elk 等日志服务器，则建议开启，更容易处理日志
    # log_format json '{"@timestamp":"$time_local",'
    #                       '"server_addr":"$server_addr",'
    #                      '"server_protocol":"$server_protocol",'
    #                       '"remote_addr":"$remote_addr",'
    #                       '"request_method":"$request_method",'
    #                       '"body_bytes_sent":"$body_bytes_sent",'
    #                       '"request_time":"$request_time",'
    #                       '"upstream_response_time":"$upstream_response_time",'
    #                       '"domain":"$http_host",'
    #                       '"request":"$request",'
    #                       '"uri":"$uri",'
    #                       '"query_string":"$query_string",'
    #                       '"http_x_forwarded_for":"$http_x_forwarded_for",'
    #                       '"http_referer":"$http_referer",'
    #                       '"http_user_agent":"$http_user_agent",'
    #                       '"status":"$status"}';

    server_tokens off;
    sendfile        on;
    tcp_nopush     on;
    charset UTF-8;
    gzip on;
    gzip_disable "msie6";
    gzip_proxied any;
    gzip_min_length 1000;
    gzip_comp_level 4;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    keepalive_timeout  15;

    proxy_http_version 1.1;
    client_header_buffer_size 256k;
    client_body_buffer_size 2m;
    proxy_buffer_size 64k;
    proxy_buffers 128 64k;
    proxy_busy_buffers_size 2m;
    proxy_temp_file_write_size 1024k;


    include /etc/nginx/conf.d/*.conf;
}
```
创建 IdP 的代理配置文件，`conf.d/idp.conf`

```nginx
server {
      listen       80;
      server_name  idp.xxx.edu.cn;
      rewrite ^(.*) https://$host$1 permanent;
}

server {
      listen       443 ssl http2;
      server_name  idp.xxx.edu.cn;
      ssl_stapling on;
      ssl_stapling_verify on;
      index index.html index.htm;

      #access_log  /var/log/nginx/idp.xxx.edu.cn.access.log  json;
      access_log  /var/log/nginx/idp.xxx.edu.cn.access.log  main;

      # lets encrypt 自签证书用
      location ^~ /.well-known/acme-challenge {
          alias /var/www/dehydrated;
      }

      proxy_set_header X-Forwarded-For $remote_addr;
      # 如果 idp 前面还有一层代理，则按此配置获取上一层的 xff
      # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host            $http_host;
      proxy_set_header X-Forwarded-Proto $scheme;

      location /idp/ {
          proxy_pass      http://127.0.0.1:8080/idp/;
        }

      # carsi 接入单位，如果选择释放日志给 carsi，则配置此部分
      #location /auditlog/ {
      #  proxy_pass      http://127.0.0.1:8080/auditlog/;
      #  allow 115.27.243.6;
      #  deny all;
      #  }
    
      # idp-slo https://github.com/shanghai-edu/idp-slo
      #location /logout/ {
      #    proxy_pass      http://127.0.0.1:8080/logout/;
      #  }

      # 记得要把原 IdP 服务器上的 https 证书拷过来，或者另外传一份证书，否则 nginx 起不来
      ssl_certificate /home/letsencrypt/certs/idp.xxx.edu.cn/fullchain.pem;
      ssl_certificate_key /home/letsencrypt/certs/idp.xxx.edu.cn/privkey.pem;
      ssl_protocols TLSv1.2 TLSv1.3;
      ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
      ssl_prefer_server_ciphers on;
      ssl_session_cache shared:SSL:10m;
}
```

### 安装 IdP
注意安装时的证书密码和cookie密码尽量和原 IdP 的密码保持一致。
如果不确认可以查看原 IdP 服务器的 `/opt/shibboleth-idp/conf/idp.properties` 查看
```ini
idp.sealer.storeResource=%{idp.home}/credentials/sealer.jks
idp.sealer.versionResource=%{idp.home}/credentials/sealer.kver
idp.sealer.storePassword=password
idp.sealer.keyPassword=password
```
下载并安装 IdP4
```
cd /tmp
wget https://shibboleth.net/downloads/identity-provider/latest4/shibboleth-identity-provider-4.0.1.zip
unzip shibboleth-identity-provider-4.0.1.zip
cd shibboleth-identity-provider-4.0.1/bin/
./install.sh 
Buildfile: /home/shibboleth-identity-provider-4.0.1/bin/build.xml

install:
Source (Distribution) Directory (press <enter> to accept default): [/home/shibboleth-identity-provider-4.0.1] ? 

Installation Directory: [/opt/shibboleth-idp] ? 

INFO [net.shibboleth.idp.installer.V4Install:151] - New Install.  Version: 4.0.1
Host Name: [2001:da8:8005:a410:250:56ff:fe9d:5e5b%ens192] ? 
idp.example.com
INFO [net.shibboleth.idp.installer.V4Install:549] - Creating idp-signing, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth, keySize=3072
INFO [net.shibboleth.idp.installer.V4Install:549] - Creating idp-encryption, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth, keySize=3072
Backchannel PKCS12 Password:
Re-enter password: 
INFO [net.shibboleth.idp.installer.V4Install:592] - Creating backchannel keystore, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth, keySize=3072
Cookie Encryption Key Password:
Re-enter password: 
INFO [net.shibboleth.idp.installer.V4Install:633] - Creating backchannel keystore, CN = idp.example.com URI = https://idp.example.com/idp/shibboleth, keySize=3072
INFO [net.shibboleth.utilities.java.support.security.BasicKeystoreKeyStrategyTool:166] - No existing versioning property, initializing...
SAML EntityID: [https://idp.example.com/idp/shibboleth] ? 

Attribute Scope: [example.com] ? 

INFO [net.shibboleth.idp.installer.V4Install:433] - Creating Metadata to /opt/shibboleth-idp/metadata/idp-metadata.xml
INFO [net.shibboleth.idp.installer.BuildWar:72] - Rebuilding /opt/shibboleth-idp/war/idp.war, Version 4.0.1
INFO [net.shibboleth.idp.installer.BuildWar:81] - Initial populate from /opt/shibboleth-idp/dist/webapp to /opt/shibboleth-idp/webpapp.tmp
INFO [net.shibboleth.idp.installer.BuildWar:90] - Overlay from /opt/shibboleth-idp/edit-webapp to /opt/shibboleth-idp/webpapp.tmp
INFO [net.shibboleth.idp.installer.BuildWar:99] - Creating war file /opt/shibboleth-idp/war/idp.war

BUILD SUCCESSFUL
Total time: 24 seconds

chown tomcat:tomcat /opt/shibboleth-idp/
```

修改 `tomcat` 配置 `/opt/tomcat/latest/conf/Catalina/localhost/idp.xml` 加载 `idp.war`
```xml
<Context docBase="/opt/shibboleth-idp/war/idp.war"
         privileged="true"
         antiResourceLocking="false"
         swallowOutput="true">
</Context>
```
重启 tomcat
```
systemctl restart tomcat
```
尝试访问下 IdP 的本地状态接口，应该有数据了
```
curl http://127.0.0.1:8080/idp/profile/admin/metrics
```
# 上线

### 本地测试
在上线之间，务必先通过本地测试确保新的 IdP 服务器已经完全可用。

修改本地 `host`，让本机的域名解析指向新的 IdP 服务器地址。例如在 Windows 上，请修改 `C:\Windows\System32\drivers\etc\hosts` 文件，增加一条解析记录

将你的 IdP 域名在你自己的测试电脑上，指向新的服务器 IP 地址。
```
xxx.xxx.xxx.xxx idp.xxx.edu.cn
```

然后访问 https://whoami.cloud.sh.edu.cn/ 进行认证测试，确保各项属性均可获得

### 正式上线
#### 单机发布
如果 IdP 服务器是单机独立的模式，即 IdP 的域名是直接解析到服务器上的。

则修改 IdP 的域名解析，将解析从老 IdP 的地址更新为新 IdP 的地址。

在此过程中，域名缓存完成更新的用户，将会在新 IdP 上完成认证。域名缓存尚未更新的用户，将会在老 IdP 上完成认证。期间不会产生业务中断。

通常48小时内，所有运营商的 DNS 缓存均应该会完成更新。观察老 IdP 服务器上的日志，，当确认老 IdP 服务器上不再产生新的访问日志时，服务器即可完成下线。

#### 负载均衡（反向代理）发布
如果 IdP 服务器是通过另外的负载均衡设备进行发布，即 IdP 的域名是解析到负载均衡设备上，而非 IdP 服务器本身。

则此时可以利用负载均衡的能力，进行业务切换。以 nginx 为例。

首先创建一个 `upstream`

```
upstream idpserver {
       ip_hash; 
       #xxx.xxx.xxx.old 和 xxx.xxx.xxx.new 表示新老 idp 服务器的 ip 地址
       server xxx.xxx.xxx.old:8080;
       server xxx.xxx.xxx.new:8080;
       #检查服务器是否存活
       check interval=3000 rise=2 fall=5 timeout=1000 type=http;
}
```
修改 nginx server 内反向代理的配置，指向 `upstream`
```
      location /idp/ {
          proxy_pass      http://idpserver/idp/;
        }
```

此时由于存在负载均衡，IdP 的请求将分布在不同的 IdP 服务器上。观察业务日志，确定新 IdP 服务器上的用户正常登录。

随后在 `upstream` 中摘除老的 IdP 服务器，并 `reload` nginx 即可。注意使用 `reload` 而不是 `restart`，此时 nginx 会确保会话完成后再重启线程，即无中断的优雅重启。
```
systemctl reload nginx
```


# 2.6 反向代理

### 介绍
IdP 的服务发布，大致可以分为 `Tomcat` 直接发布，`AJP` 代理发布和 `HTTP` 反向代理发布三种。

由于 IdP 对会话校验严格，因此当采取 `HTTP` 反向代理模式发布时，必须确保 IdP 获取的请求 `URL` 中，协议为 `https`，否则会由于会话检查错误而报错。

另外，如果 IdP 的日志中，需要获取真实的用户的请求 IP，则也需要通过 `X-Forwarded-For` 方式获取，否则拿到的都是反向代理服务器的 IP 地址。

#### 反向代理配置
反向代理服务器上，需要增加 `X-Forwarded-For` 和 `X-Forwarded-Proto` 两个 `header` 信息，传递真实的请求 IP 和真实的请求协议。

以 `nginx` 为例：
```
      location /idp {
        proxy_pass      http://192.168.1.11:8080/idp;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
      }

```

#### tomcat 配置
在 tomcat 的 `server.xml`  的 `Engine` tag 中，增加下述配置。将 `internalProxies` 配置为您反向代理的实际地址。

```
    <Valve className="org.apache.catalina.valves.RemoteIpValve"
           internalProxies="192.168.1.14"
           remoteIpHeader="x-forwarded-for"
           protocolHeader="x-forwarded-proto"
    />
    </Engine>
  </Service>
</Server>
```
# 反向代理

### 介绍
IdP 的服务发布，大致可以分为 `Tomcat` 直接发布，`AJP` 代理发布和 `HTTP` 反向代理发布三种。

##### 什么是 AJP 代理？
`AJP` 是 `Apache` 提供的一种反向代理服务器的二进制协议。在 Apache 中通过 mod_proxy_ajp 模块发送 AJP 数据，另一端服务器——比如 Tomcat 则需要能够接受 mod_proxy_ajp 模块发送的 AJP 协议数据，进行交互。

由于 AJP 协议使用二进制传输方式，因此它比 HTTP 的文本传输方式更有效率。并且它能够直接向服务器端提供原始的 HTTP 头部信息，例如客户端 IP 地址等。

##### 什么是 HTTP 反向代理？
更多时候，我们会通过更通用的 `HTTP` 协议来实现反向代理。此时，由于代理服务器向后端的真实服务器也是通过 `HTTP` 协议发送请求。因此默认情况下，后端服务器所获取到的客户端地址，均是代理服务器的地址；后端服务器获取到的请求协议，则是后端服务器所发布的协议。

例如我们通过反向代理服务器发布了一个 `https://idp.example.org` 这样一个网站，并做了 SSL 卸载，与后端服务器采用 `http` 协议通讯。那么默认情况下，后端服务器所获取的请求 url 均是 `http://idp.example.org`。这在 Shibboleth-IdP 的场景中会存在问题。

当客户端请求代理服务器的 `https` 服务时，他所产生的 `cookie` 中所记录的 `url` 信息也都是 `https` 的请求。而此时如果服务器端获取到的是 `http` 请求，则服务器会认为这个请求非法，从而报错。

因此，我们需要将真实的请求协议，和真实的客户端地址，通过插入 `header` 的方式，提供给后端服务器。

#### 反向代理配置
反向代理服务器上，增加 `X-Forwarded-For` 和 `X-Forwarded-Proto` 两个 `header` 信息，传递真实的请求 IP 和真实的请求协议。

以 `nginx` 为例，其他反向代理的配置请咨询相关供应商获取技术支持。
```
      location /idp {
        proxy_pass      http://192.168.1.11:8080/idp;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
      }

```

#### tomcat 配置
在 tomcat 的 `server.xml`  的 `Engine` tag 中，增加下述配置。从而将传递给应用的客户端 `ip` 和请求协议替换为 `X-Forwarded-For` 和 `X-Forwarded-Proto` 中的值。

注意将 `internalProxies` 配置为您反向代理的内网地址。
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
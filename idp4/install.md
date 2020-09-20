# 2.8.1 IdP 安装


## 环境需求

- Oracle Jave 或者 OpenJDK  11，需下载对应的 the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 文件。虽然 JRE 可能可以使用，但是技术团队只支持JDK。	
- 需要支持Servlet API 3.1 的Servlet容器
	- Tomcat9+
	- Jetty9.4+
- 正式支持的版本 Tomcat 9+，过去的Tomcat版本在理论上可以支持，但并没有对以上版本进行测试，可能有 Bug。
- 没有正式支持OS供应商提供的任何“打包”容器。不会对这些容器进行测试，因此无法评估打包过程中可能出现的各种问题。
- 由于核心技术团队是以Jetty平台进行开发测试的，因此官方推荐使用Jetty 9 容器. 但是显然我们更习惯用 Tomcat, 所以示例都以 tomcat9 为参照。
- 无操作系统限制，但推荐使用Linux, OS X 和 Windows。


#### 不支持的版本
idP V4.0不支持以下Idp的老版本配置：

- Java10 或更早的版本。
- 不支持 Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files.
- Tomcat7 或更早版本。请注意如系统已安装了RHEL5 或 RHEL6（RHEL5 预装的是 Tomcat5，RHEL6 预装的是 Tomcat6），在安装Idp 3.0时，则需要先安装RHEL7的一个可选应用容器，这样才能可以使用Tomcat9。
- Jetty9.3 或更早的版本。

## 安装

#### 准备工作
- 一张 SSL 证书以开启 idP 的 https 访问
- 用于表示你 idP 的 entityID，安装程序默认会使用你的 hostname）
- 由于 saml 协议对时间敏感，确保时间同步. 时间服务器可以使用 `ntp.shec.edu.cn`
- 一个 metadata 的获取源，用于和互信的 sp 进行通讯。上海教育认证中心的 Metadata 地址是 https://ds.shec.edu.cn/metadata.xml

安装程序会为你建议或生成一些信息：
- idP 的 entityID
- 一对自签发的密钥和证书，用于：
	- 消息传输的验证
	- 默认的 https 通讯加密，使用8443端口
	- 其他系统和 idP 的信息加密和解密
- idP 自己加密 cookies 和其他数据的 密钥和密钥版本文件。（这是 java keystore 的格式 "JCEKS")
- 基于这些信息产生的 idP 的默认配置

安装好 tomcat8 并做好 java 环境, 细节略。

#### linux 下安装

Shibboleth idp 是一个标准的 Java web application，基于 Servlet 3.1 规范。你可以在所有兼容此规范的 Servlet 容器上运行 idP 。官方支持的版本是 Tomcat 和 Jetty，这里使用 Tomcat 为例。

1. 下载 [idP](http://shibboleth.net/downloads/identity-provider/latest) 最新的版本
2. 解压你下载的文件，例如 : 
	```
	unzip shibboleth-identityprovider-VERSION-bin.zip
	```
3. 进入解压的目录，例如：
	```
	cd shibboleth-identityprovider-VERSION
	```
4. 运行 ```./install.sh``` (linux) 或者 ```./install.bat``` (windows)
	
	- idP 的安装目录，本文中以 idp.home代替
5. 部署 idP 的 warfile。路径在  ```idp.home/war/idp.war```。后文会专门说明如何部署

##### Rebuild idP warfile
如果你要重新编译 idp warfie，运行 ```bin/build.sh``` 
```bash
opt/shibboleth-idp# bin/build.sh
```

#### Apache Tomcat 配置

##### 基本说明
本文中，idp.home 指代 idP 的安装路径，TOMCAT_HOME 指代 Tomcat 的安装路径

所有 Tomcat9 的版本都可以用，当然最好还是用[最新的版本](http://tomcat.apache.org/download-90.cgi)

##### 必须调整的配置

在 idP4 中，你必须指定 Tomcat 去读取 idP 的 warfile。创建 ```TOMCAT_HOME/conf/Catalina/localhost/idp.xml```文件，复制以下内容。（注意替换 idp.home 为你自己的 idP 安装路径）
```xml
<Context docBase="idp.home/war/idp.war"
         privileged="true"
         antiResourceLocking="false"
         swallowOutput="true">
 
</Context>
```
- Tomcat 默认监听 8080 和 8443 端口。一般我们需要把他改成 80 和 443，你可以在```TOMCAT_HOME/conf/server.xml```修改这个。本文示例中，我们将用 apache 来代理 Tomcat，所以不去管它。
- - Tomcat 默认没有提供 Java Server Tag Library，这使得 idP4 的 status 页面无法显示。解决的办法是下载 [jstl的jar包](https://build.shibboleth.net/nexus/service/local/repositories/thirdparty/content/javax/servlet/jstl/1.2/jstl-1.2.jar)，然后放在 tomcat 的 lib 里，例如 `/usr/share/tomcat/lib`
- 添加以下参数到 CATALINA_OPTS 的环境变量。
	- ```-Didp.home=<location> ``` <location> 替换为你实际的路径
	- v3.12以后的版本中，idp.home 也可以通过 web.xml 来指定。在 edit-webapp 目录下创建web.xml，然后重新 build idp.war
	```xml		
	<context-param>
	    <param-name>idp.home</param-name>
	    <param-value>/opt/idp</param-value>
	</context-param>
	```
	-  ```-XX:+UseG1GC``` 开启垃圾回收器，以在 metadata 比较大的时候获得好一点的性能
	-  ```-Xmx1500m``` JVM 的最大内存，如果联盟的 metadata 很大（超过 25M) 那么至少需要 1.5G 的内存。最好根据实际情况测试一下
	-  ```-XX:MaxPermSize=128m``` JVM最大允许分配的非堆内存，按需分配


##### 建议的配置调整
- 限制 Post 表单的大小。在 Tomact 的 Connector 中（AJP 的或者 http 的）配置 maxPostSize参数。100K(100000)是一个相对比较合理的数值。
- 关掉 Tomcat 的会话保持。在 ```TOMCAT_HOME/conf/context.xml```中取消 ``` <Manager pathname="" /> ``` 的注释。这可以防止在容器关闭时，idP 的会话错误。没必要通过 Tomcat 来为 idP 集群做会话保持。 

#### 使用反向代理

idP 部署在 tomcat 上，可以设置 http 对 tomcat 的代理，以便于维护和增强扩展性。
这里以 apache 做 ajp 反向代理为例

取消 tomcat 的 AJP 身份认证（如果需要的话）
修改```TOMCAT_HOME/conf/server.xml```
在 ```<!-- Define an AJP 1.3 Connector on port 8009 -->```下修改
```xml
<Connector port="8009" address="127.0.0.1"
               enableLookups="false" redirectPort="443" protocol="AJP/1.3"
               tomcatAuthentication="false" />
```

配置 apache 的 ajp 反向代理
修改 httpd.conf
```ProxyPass /idp/ ajp://localhost:8009/idp/```

#### 部署 ssl 证书

在 apache 上部署 ssl 证书可比 tomcat 容易多了.可以使用 lets encrypt

查看```/etc/httpd/conf.d/ssl.conf```
在对应路径下部署证书

#### idP 状态的查看

```idp.home/bin/status.sh```
```
### Operating Environment Information
operating_system: Linux
operating_system_version: 2.6.32-696.el6.x86_64
operating_system_architecture: amd64
jdk_version: 1.8.0_131
available_cores: 2
used_memory: 749 MB
maximum_memory: 1500 MB

### Identity Provider Information
idp_version: 3.3.1
start_time: 2017-05-04T00:23:39+08:00
current_time: 2017-05-04T00:24:01+08:00
uptime: 21304 ms

service: shibboleth.LoggingService
last successful reload attempt: 2017-05-03T16:22:09Z
last reload attempt: 2017-05-03T16:22:09Z

service: shibboleth.ReloadableAccessControlService
last successful reload attempt: 2017-05-03T16:22:16Z
last reload attempt: 2017-05-03T16:22:16Z

service: shibboleth.MetadataResolverService
last successful reload attempt: 2017-05-03T16:22:16Z
last reload attempt: 2017-05-03T16:22:16Z

service: shibboleth.RelyingPartyResolverService
last successful reload attempt: 2017-05-03T16:22:14Z
last reload attempt: 2017-05-03T16:22:14Z

service: shibboleth.NameIdentifierGenerationService
last successful reload attempt: 2017-05-03T16:22:14Z
last reload attempt: 2017-05-03T16:22:14Z

service: shibboleth.AttributeResolverService
last successful reload attempt: 2017-05-03T16:22:14Z
last reload attempt: 2017-05-03T16:22:14Z

        DataConnector staticAttributes: has never failed

service: shibboleth.AttributeFilterService
last successful reload attempt: 2017-05-03T16:22:14Z
last reload attempt: 2017-05-03T16:22:14Z
```

#### 服务的启停

Tomcat服务的启停
```
TOMCAT_HOME/bin/startup.sh
TOMCAT_HOME/bin/shutdown.sh
```
Apache服务的启停
```
systemctl start httpd
systemctl stop httpd
```

可以设置为系统自动启动服务
```
systemctl start httpd
systemctl enable httpd
```
修改```/etc/rc.local```
添加
```
TOMACT_HOME/bin/startup.sh
```


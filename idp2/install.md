# 2.1.1 IdP 安装

## IdP2 End of Life
IdP2 已经在 2016.7.31 停止更新,本章部分仅做 IdP2 的维护参考用,新安装用户请选择 IdP3

#### 准备工作

- 一张 SSL 证书以开启 idP 的 https 访问.可以使用 Lets encrypt
- 一个 metadata 的获取源，用于和互信的 sp 进行通讯。请向联盟方咨询 Metadata 的获取地址

安装时候它会自动为你生成以下信息：
- idP 的 entity ID
- 一对用于 SAML 通信时用的密钥 （不是用来 https 访问idP的）
- idP 的初始 metadata
- idP 的初始默认配置

#### 特别注意

- Ret hat/CentOS 一些老的版本里，默认使用的 GNU Java 编译的VM(gcj)。这是不支持的，请务必重新安装其他的JVM（比如 Oracle jdk）
- Debian7（“wheezy") 下的 使用 openjdk6 有一些问题，建议升级到 openjdk7
- 关于 openjdk，我们还是建议使用 Oracle 官方的标准 jdk。openjdk 有时候会有些意料之外的问题，比如内存泄漏，比如上面 Debian 的问题。你也可以猜到到对任何莫名其妙问题的解释都将指向在在Oracle的JVM上重新部署。

#### 安装 idP

Shibboleth idp V2 的版本是一个标准的 Java web application，基于 Servlet 2.4 规范。所以你可以在所有兼容此规范的 Servlet 容器上运行 idP 。如果你不知道该选哪一个，很多人使用 Apache Tomcat，也有很多人用 Jetty。

官方支持的容器版本是 Jetty7+ 和 Tomcat6+ 的版本。本文将以 Tomcat 为例，关于容器本身的一些文档，可以在对应的官网去查。

1. 下载 [idP](http://shibboleth.net/downloads/identity-provider/2.4.5/) 
2. 解压你下载的文件，例如 : 
	```
	unzip shibboleth-identityprovider-2.4.5-bin.zip
	```
3. 进入解压的目录，例如：
	```
	cd shibboleth-identityprovider-2.4.5
	```
4. 运行 ```./install.sh``` (linux) 或者 ```./install.bat``` (windows)
	- idP 的安装目录，本文中以 IDP_HOME 代替
5. 部署你的 idP war 文件到 java 容器里，文件的位置在 ```IDP_HOME/war/idp.war```

注意，idP 的 metadata 并不会在你修改了 idP 的配置后自动更新，你需要手动去更新这个

##### Tomcat 调优(可选）

环境部署
- 至少需要 Apache Tomcat 6.0.17 以上的版本
- Java 6 以上的版本

性能调优
- 添加以下参数到 JAVA_OPTS 的环境变量
	-  ```-Xmx512m``` 这里配置了你 Tomcat 所允许使用的最大内存，建议至少 512M
	-  ```-XX:MaxPermSize=128m``` JVM最大允许分配的非堆内存，按需分配
- 限制 Post 表单的大小。在 Tomact 的 Connector 中（AJP 的或者 http 的）配置 maxPostSize参数。100K(100000)是一个相对比较合理的数值。

部署 idp.war 时，最简单的办法当然是直接复制到 Tomcat 的 webapp 目录下，Tomcat 启动的时候会自动解压他。

另一种办法是告诉 Tomcat 到哪里去加载 idp.war。新建如下文件 ```TOMCAT_HOME/conf/Catalina/localhost/idp.xml``` 内容如下（替换IDP_HOME为你的实际路径)
```xml
<Context docBase="IDP_HOME/war/idp.war"
         privileged="true"
         antiResourceLocking="false"
         antiJARLocking="false"
         unpackWAR="false"
         swallowOutput="true" />
```

#### 使用反向代理

IdP 部署在 tomcat 上，可以对其进行反向代理来提供灵活性。

配置详见 [反向代理](https://eac.cloud.sh.edu.cn/document/proxy/)

#### 部署 ssl 证书

HTTP 的 SSL 证书，可以采购商用证书，也可以使用 Let's Encrypt 自签。

配置详见 [SSL证书](https://eac.cloud.sh.edu.cn/document/lets_encrypt/)
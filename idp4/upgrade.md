# IdP 升级

#### 准备工作
IdP 通过覆盖安装的方式进行升级，因此如果已经有点忘记了的话，建议回顾一下 IdP 的 [安装](https://eac.cloud.sh.edu.cn/document/idp4/install.html) 文档以熟悉流程。
同时，在升级前建议先阅读 [ReleaseNotes](https://wiki.shibboleth.net/confluence/display/IDP4/ReleaseNotes) 以了解版本变化。通常来说升级不需要任何的配置更改，但有时可能涉及一些安全问题时可能需要加以关注。

升级过程不会覆盖任何现有的配置文件，`views/templates`, `properties` 等这些都不会覆盖。但是 `idp.home/system` 和 `idp.home/webapp` 不包含在内，这部分内容可能会被升级文件覆盖，请务必留意。

根据设计，`idp.home/edit-webapp` 目录下的文件可用于在升级过程中保留更改，但是如果您修改了现有文件(而不是增加新的文件)，您应该始终将更改后的版本与升级后的文件进行比较，以了解新版版所带来的更改是否重要。同时，在 [ReleaseNotes](https://wiki.shibboleth.net/confluence/display/IDP30/ReleaseNotes) 也会强调此类的任何更改，请务必关注。

#### 升级
1. 下载最新版本的 [IdP](https://shibboleth.net/downloads/identity-provider/latest4/) 安装包。目前的最新版本是 `4.0.1`
2. 解压，例如 `tar -zxvf shibboleth-identity-provider-4.0.1.tar.gz ` 。升级完以后这些都可以删掉
3. 进入到解压的文件夹，`cd shibboleth-identity-provider-4.0.1`
4. 运行安装脚本: `./bin/install.sh` ,注意安装目录必须和当前 IdP 的目录相同，否则的话就是一次全新安装了
5. 运行 `idp.home/bin/build.sh` 重新编译 `idp.war`。这里 `idp.home` 是你的安装目录，比如 `/opt/shibboleth-idp/`
6. 重启 `tomcat`，升级完成。

#### IdP3 -> IdP4
如果您的 IdP3 所部署的 Java 环境已经是 Java 11，则您可以参照上述文档直接覆盖安装。如果 IdP3 上的 Java 环境是 Java 8，则必须先升级 Java 环境才能完成升级工作。

出于业务可靠性的保障要求，此时建议新部署一台服务器重新安装 IdP4 并迁移相关配置文件完成升级。

关于 IdP3 -> IdP4 的无感知升级，请详见 [IdP4 升级文档]()

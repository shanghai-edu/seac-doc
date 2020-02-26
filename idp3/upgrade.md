# 2.3.2 IdP 升级

#### 准备工作
IdP 通过覆盖安装的方式进行升级，因此如果已经有点忘记了的话，建议回顾一下 IdP 的 [安装](https://eac.cloud.sh.edu.cn/document/idp3/install.html) 文档以熟悉流程。
同时，在升级前建议先阅读 [ReleaseNotes](https://wiki.shibboleth.net/confluence/display/IDP30/ReleaseNotes) 以了解版本变化。通常来说升级不需要任何的配置更改，但有时可能涉及一些安全问题时可能需要加以关注。

升级过程不会覆盖任何现有的配置文件，`views/templates`, `properties` 等这些都不会覆盖。但是 `idp.home/system` 和 `idp.home/webapp` 不包含在内，这部分内容可能会被升级文件覆盖，请务必留意。

根据设计，`idp.home/edit-webapp` 目录下的文件可用于在升级过程中保留更改，但是如果您修改了现有文件(而不是增加新的文件)，您应该始终将更改后的版本与升级后的文件进行比较，以了解新版版所带来的更改是否重要。同时，在 [ReleaseNotes](https://wiki.shibboleth.net/confluence/display/IDP30/ReleaseNotes) 也会强调此类的任何更改，请务必关注。

#### 升级
1. 下载最新版本的 [IdP](http://shibboleth.net/downloads/identity-provider/latest/) 安装包。目前的最新版本是 `3.4.6`
2. 解压，例如 `tar -zxvf shibboleth-identity-provider-3.4.6.tar.gz ` 。升级完以后这些都可以删掉
3. 进入到解压的文件夹，`cd shibboleth-identity-provider-3.4.6`
4. 运行安装脚本: `./bin/install.sh` ,注意安装目录必须和当前 IdP 的目录相同，否则的话就是一次全新安装了
5. 运行 `idp.home/bin/build.sh` 重新编译 `idp.war`。这里 `idp.home` 是你的安装目录，比如 `/opt/shibboleth-idp/`
6. 重启 `tomcat`，升级完成。

#### CAS/OAuth
由于 `IdP 3.4.3` 之后的版本涉及内部 API 变更。因此如果升级前的版本小于等于 `3.4.3`，且对接了 CAS/OAuth，那么需要同时更新 CAS/OAuth 的插件，否则无法正常工作。

详情参见 [CAS 对接](https://eac.cloud.sh.edu.cn/document/idp3/idp_cas.html) 和 [OAuth 对接](https://eac.cloud.sh.edu.cn/document/idp3/idp_oauth.html) 中关于升级部分的描述。
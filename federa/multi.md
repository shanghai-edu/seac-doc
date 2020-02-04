1.4.1 多联盟集合

#### 集合
由于 SAML 本质上是点对点之间的关系.根据 SAML 的认证流程我们可以看到,在用户选择了 IdP 之后,所有的请求都在 IdP 和 SP 之间直接发生.

因此身份联盟本质上,指的就是联盟整个 Metadata 的集合,在 Metadata 内的 IdP 和 SP 构成信任关系.

多个不通的联盟,可以互相有交集.此时处于交集内的 IdP 可以同时处于联盟的 Metadata 内,从而可以同时支持多个联盟的 SP 服务,而无需部署安装多个 IdP 服务器.

![](https://eac.cloud.sh.edu.cn/images/federa.png)
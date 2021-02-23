# 发行说明

翻译自 [Shibboleth IdP4 RelaseNote](https://wiki.shibboleth.net/confluence/display/IDP4/ReleaseNotes)，不定期更新。

## “就地”升级！

如果要从V3或V4.0升级，请按照[升级](https://wiki.shibboleth.net/confluence/display/IDP4/Upgrading)页面上的说明进行操作，不要单独安装V4，然后尝试应用旧的配置文件。 您必须使用包含先前工作配置副本的安装目录“就地”升级。

在大多数情况下，如果不这样做，将导致系统无法正常工作，因为在两种情况下，安装程序的输出之间绝对存在本质上的差异，以防止您的配置被意外地更改。

对V4.1进行的内部更改的程度使该问题更加严重，并且轻易合并新旧文件的结果将是危险的。 重要的是，人们只有在升级并验证系统行为之后，才注意此警告并尝试对设置进行现代化设置（如果有的话）。

在升级系统之前，请查阅这些发行说明。 在升级之前，您应该查看所运行版本之后的所有版本的注释，包括参考较早的V3注释。

另外，请注意有关容器或Java兼容性的以下问题：

- [LDAPonJava](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPonJava)

## 已知错误

这些是代码中的已知问题，可能会影响部署，并且尚未解决。

- [IDP-1623](https://issues.shibboleth.net/jira/browse/IDP-1623) - NullPointerException during c14n （CLOSED） 
- [IDP-1654](https://issues.shibboleth.net/jira/browse/IDP-1654) - NullPointerException in AttributeResolver （CLOSED）。此错误会影响依赖新的“导出”功能的DataConnector，然后无法返回任何结果（通常但并非总是如此，因为它们被ActivationCondition覆盖了）。 目前，最简单的解决方法是确保此类连接器因错误而失败，或者由故障转移连接器进行备份，以确保返回某种虚拟结果。
- [IDP-1625](https://issues.shibboleth.net/jira/browse/IDP-1625) - CSRF token missing in Duo cancel request hyperlink （RESOLVED）。启用CSRF功能时，此问题破坏了Duo登录流程中的“取消”选项。 该问题已记录在此修复程序中，可以直接将其应用于duo.vm模板。 由于它涉及模板，因此修复程序将是手动的，并在下一版本的发行说明中进行了说明。

## 4.1.0 (未发布)
[问题](https://issues.shibboleth.net/jira/secure/IssueNavigator.jspa?reset=true&jqlQuery=filter%3D13571+&src=confmacro)

这是一个重要的新功能版本，其中包含比平时数量更多的新配置选项，但是这些选项是向后兼容的，并且大多数情况下简化了新的部署工作。 现有系统可以适于利用这些新功能，以减少未使用的配置文件的数量。 您可以在各个主题中找到详细信息，但是可能需要查看有关ModuleConfiguration的新资料，并考虑禁用与未使用功能关联的任何模块。 这将彻底删除一些未更改的文件并重命名其他文件，以便您可以查找和删除它们。

请注意，尽管许多文档都包含标有“ V4.1 +”的资料部分，但这并不意味着V4.0资料在升级的系统中“不起作用”。 这只是一种简化方法，使记录每个版本中“标准”的方法就更容易了。

### 重大变化

为了使系统更加易于使用插件，对一个非常少见的扩展点做了改变，这与我们对于次要发行版的常规兼容性规则不一致。 自定义PrincipalSerializer Bean的注册作为更高级的身份验证扩展的一部分，将不再被识别并且无法使用。 从V4.1开始，不再识别**DefaultPrincipalSerializers**和**shibboleth.PrincipalSerializers**列表bean。

该机制的替代品是新的自动连线服务接口[PrincipalService](http://shibboleth.net/cgi-bin/java-idp.cgi/net.shibboleth.idp.authn.principal.PrincipalService)。 可以在[Authentication](https://wiki.shibboleth.net/confluence/display/IDP4/Authentication)页面上找到有关此机制的更新信息。 升级后的系统不会彻底中断，但将无法找到和使用通过旧挂钩注册的任何自定义序列化程序。

### 更改为默认值

以下默认值对于新安装已更改，但对于正确升级的系统则不会更改（通常，这些默认值是通过取消属性或Bean的注释标记，启用相关行为）：

- 新的[idp.bindings.inMetadataOrder](https://wiki.shibboleth.net/confluence/display/IDP4/RelyingPartyConfiguration)属性默认为true，但在新安装中设置为false。 主要目的和影响将在下面的“注销行为的恢复”中进行讨论。

### 删除系统文件

在此发行版中，我们不再需要在安装树中包含“system”目录，而是将资源嵌入到系统的库中。 由于web.xml部署配置文件中的历史记录引用了此目录中的文件，因此不可能从升级的系统中完全删除它以保持兼容性。但是新安装的文件中将不包含它。 将来的发行版将需要对web.xml进行修改，以允许完全删除目录。

作为此更改的一部分，您可能还需要从services.xml中的**shibboleth.MessageSourceResources** bean中删除对“"%{idp.home}/system/messages/messages" 的旧的引用。

### 新的属性文件和属性查找

现在，许多以前只能使用XML文件和Spring配置进行配置的常用设置将作为属性公开（为如果且仅当原始bean不存在时，保证兼容性）。 各个子目录([conf/authn/authn.properties](https://wiki.shibboleth.net/confluence/display/IDP4/AuthenticationConfiguration), [conf/admin/admin.properties](https://wiki.shibboleth.net/confluence/display/IDP4/AdministrativeConfiguration), [conf/c14n/subject-c14n.properties](https://wiki.shibboleth.net/confluence/display/IDP4/SubjectCanonicalizationConfiguration))中都有新的属性文件，其中包含这些设置的示例。 虽然新安装会自动利用这些属性，但是（正确）升级的系统将没有其他调整。链接的主题讨论了，如果希望利用这些属性时，如何“启用”的方法。

另请注意，在此版本中，“查找”属性文件的过程已得到改进。 在较旧的版本中，并且**idp.searchForProperties**属性为false/missing时，加载的属性文件集仅由idp.additionalProperties设置控制。 默认情况下，新的安装或明确启用**idp.searchForProperties**属性的安装将在**conf**目录树中查找并加载所有扩展名为“ .properties”的文件。 外在属性仍可用于直接从其他位置加载文件。

### 恢复注销行为

添加SOAP Logout支持会产生意外的副作用，即导致有时会选择SOAP绑定，而不是标准的，支持基于前端绑定的更有效方法。 绑定选择的问题可以追溯到V2，是由IdP按元数据顺序选择引起的。 这已通过新属性idp.bindings.inMetadataOrder解决，可以将其设置为false以产生更理想的绑定选择顺序，该顺序在受支持时有利于Redirect和POST。 为了兼容性，它默认为true，但是对于新安装，它设置为false。

### IPv6客户端地址清理

在一个被补丁程序反转的修补upstream后，我们对代码应用了全局修复程序，以确保IPv6客户端地址始终公开，而无需在地址周围使用方括号，该方括号应限于“ address-as-host”格式，而不是裸地址。 这可以解决许多问题，但可能会导致日志文件发生更改，并且可能会影响未对处理这两种格式进行调整的脚本。

### 新的对象

- [shibboleth.BiConditions.AND](https://wiki.shibboleth.net/confluence/display/IDP4/PredefinedBeans)
- [shibboleth.BiConditions.OR](https://wiki.shibboleth.net/confluence/display/IDP4/PredefinedBeans)
- [shibboleth.BiConditions.NOT](https://wiki.shibboleth.net/confluence/display/IDP4/PredefinedBeans)
- [shibboleth.BiFunctions.Constant](https://wiki.shibboleth.net/confluence/display/IDP4/PredefinedBeans)
- [shibboleth.BiFunctions.Compose](https://wiki.shibboleth.net/confluence/display/IDP4/PredefinedBeans)
- [shibboleth.BasicKeyInfoGeneratorFactory](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.X509KeyInfoGeneratorFactory](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.metrics.ExposedProperties](https://wiki.shibboleth.net/confluence/display/IDP4/MetricsConfiguration)

### 改名的对象

- [shibboleth.IncludedSignatureAlgorithms](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.ExcludedSignatureAlgorithms](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.IncludedEncryptionAlgorithms](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.ExcludedEncryptionAlgorithms](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.authn.RemoteUser.allowedUsernames](https://wiki.shibboleth.net/confluence/display/IDP4/RemoteUserInternalAuthnConfiguration)
- [shibboleth.authn.RemoteUser.deniedUsernames](https://wiki.shibboleth.net/confluence/display/IDP4/RemoteUserInternalAuthnConfiguration)
- [shibboleth.consent.attribute-release.PromptedAttributeIDs](https://wiki.shibboleth.net/confluence/display/IDP4/ConsentConfiguration)
- [shibboleth.consent.attribute-release.IgnoredAttributeIDs](https://wiki.shibboleth.net/confluence/display/IDP4/ConsentConfiguration)

### 新的属性

由于前面的更改，增加了很多新属性，但这里提到了一些主要设置：

- [idp.bindings.inMetadataOrder](https://wiki.shibboleth.net/confluence/display/IDP4/RelyingPartyConfiguration)
- [idp.schemaValidation.strict](https://wiki.shibboleth.net/confluence/display/IDP4/SchemaValidationFilter)
- [idp.logout.preserveQuery](https://wiki.shibboleth.net/confluence/display/IDP4/LogoutConfiguration)
- [idp.security.basicKeyInfoFactory](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [idp.security.x509KeyInfoFactory](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)

## 4.0.1.1（2020年7月22日）
这是Windows安装软件包的服务版本，目的是解决Jetty中的安全问题，该问题已更新至9.4.30版。 维护自己的Java容器的人都不需要它，强烈建议继续使用现有的维护方法。

## 4.0.1（2020年6月3日）

[问题](https://issues.shibboleth.net/jira/secure/IssueNavigator.jspa?reset=true&jqlQuery=filter%3D13570+&src=confmacro)

这是一个修补程序版本，用于解决已发现的错误，修补由于缺少JNDI支持而引起的某些LDAP功能缺失，例如SASL身份验证和引用处理。 LDAP功能不是我们对修补程序发行版中的新功能的常规策略，并且在适用的情况下，配置选项以4.0.1表示。

修正了一个错误，该错误导致元数据解析器在包含空元素的无效元数据时崩溃。 以前的版本无意间接受并忽略了这些元素。 该修补程序添加了有关无效内容的警告，但更正了崩溃的错误。

对于Windows安装程序，Jetty已更新为9.4.28版，并且已解决了使用通配符证书的问题。 在反向通道部署中，此问题的完整解决方案涉及对我们提供的用于绕过证书验证的插件的更新，并且此插件已重命名以反映它是专为与Jetty 9.4一起使用而设计的。 **Jetty94**页面已更新以反映这一点。 较早的插件和反向通道XML配置目前仍然兼容，但是即使使用通配符或Multi-SAN证书，也不太可能失败。

### 更改为默认值

- 4.0.0包含的“ telephoneNumber”属性的代码转换规则已交换了SAML 1和SAML 2名称，因此升级的系统将出现错误，需要手工纠正。

### 已知的问题

- 在默认的web.xml文件中包含一个错误，使长时间尝试修复使用默认约束时的告警。 元素`<authn-constraint/>`应该是`<auth-constraint/>`。 这可能会导致警告或导致规则被忽略.

## 4.0.0（2020年3月11日）

[问题](https://issues.shibboleth.net/jira/secure/IssueNavigator.jspa?reset=true&jqlQuery=filter%3D12673+&src=confmacro)

这是第四代Identity Provider软件的第一版。 关键文档链接位于IDP4[主页](https://wiki.shibboleth.net/confluence/display/IDP4/Home)上，例如[SystemRequirements](https://wiki.shibboleth.net/confluence/display/IDP4/SystemRequirements), [Installation](https://wiki.shibboleth.net/confluence/display/IDP4/Installation), 和 [Upgrading](https://wiki.shibboleth.net/confluence/display/IDP4/Upgrading)材料。 注意新的 [SystemRequirements](https://wiki.shibboleth.net/confluence/display/IDP4/SystemRequirements) ，因为它们在Java和容器版本方面已经发生了很大的变化。

此版本应与Shibboleth的所有先前版本以及支持相同[标准](https://wiki.shibboleth.net/confluence/display/DEV/Technical+Specifications)的其他软件兼容。
作为一个重大升级，已解决的问题和添加的功能列表很多。 下面主要显示许多重要的更改，在注释或警告框中标注了特别值得注意的问题。

- 因为有API的添加，更改和删除，需要检查任何第三方扩展的代码兼容性，并重新编译以与此版本一起使用，并且部署程序创建的任何自定义脚本都应经过仔细的测试和审查。

### 更改为默认值

对于新安装和升级，以下默认设置均已更改，应检查以下可能的不兼容性：

- 如果部署时为调整，则idp.cookie.secure属性现在默认为true。 如果不需要HTTP兼容性，则需要调整该属性。 （请注意，容器的会话Cookie受web.xml中的策略控制，尽管我们也更改了默认设置，但只有没有自己的web.xml自定义项的升级程序才会受到此更改的影响。）
- 现在，默认的LDAP提供者为UnboundID。 有关更多信息，请参见下面的LDAP部分。.
- 删除了一些用于维护％T审核字段的V2样式格式（记录的时间戳）的旧代码，日志现在使用指定格式或默认格式输出所有日期/时间字段。 因此，对于某些部署情况，％T字段的格式可能会更改.
- 与此相关，**shibboleth.AuditDateTimeFormat** bean由Java   [DateTimeFormatter](http://shibboleth.net/cgi-bin/java-jdk.cgi/java.time.format.DateTimeFormatter)类而不是已弃用的joda-time版本解释为一种模式，并且这些类中的某些差异可能会导致日志更改。 如果您需要相同的输出，则需要进行审查.
- 下文所述的与属性相关的更改的副作用是，没有friendlyName属性的SAML 2.0 [AttributeEncoders](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeEncoderPluginConfiguration)默认使用该属性的ID填充FriendlyName。 目前，除了转换为使用[AttributeRegistryConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeRegistryConfiguration)表示规则之外，没有其他方法可以还原原始行为。

以下默认值对于新安装已更改，但对于正确升级的系统则不会更改（通常，这些默认值是通过取消属性或Bean的注释标记，启用相关行为）

- 现在默认的XML加密算法是AES128-GCM而不是AES128-CBC。 有关更多信息，请参见 [GCMEncryption](https://wiki.shibboleth.net/confluence/display/IDP4/GCMEncryption)，并请注意，新安装的系统将无法与大量没有现代算法支持的SAML SP交互
- 现在，当使用客户端会话存储功能时，HTML本地存储现在默认为启用（该功能过去也一直是默认选项）。
  - 此外，默认情况下还启用了注销功能**idp.session.trackSPSessions**和**idp.session.secondaryServiceIndex**属性。 不希望尝试支持注销传播的部署可以禁用它们。
- 默认情况下启用的配置文件集不再包含任何SAML 1.1支持或SAML 2.0属性查询。
- 现在，运行时使用的默认信任引擎将省略PKIX支持。 现在仅支持元数据互操作性配置文件（[Metadata Interoperability Profile](https://wiki.oasis-open.org/security/SAML2MetadataIOP)），这是默认行为，没有回退计划。
- 默认情况下配置为快速失败的一些 [ReloadableServices](https://wiki.shibboleth.net/confluence/display/IDP4/ReloadableServices) 已更改为非快速失败行为。
- 审计输出已更改为包括更有用的默认格式，并对日志记录特定字段的方式进行了各种改进，以减少重复和不必要的常量值.


### 删除过时的特性

删除了许多先前过时的功能，设置，API等。 使用这些过时的位，会在启动时或在软件的最新版本中使用时产生了标准警告，并且其中大多数现在不起作用。 强烈建议在升级之前消除警告

### 日期/时间API

整个代码库已从joda-time日期/时间API转换为Java 8中引入的新日期/时间API。此更改在代码中广泛存在，大多数使用长/长类型来表示转换的时间戳和持续时间 分为[Instant](http://shibboleth.net/cgi-bin/java-jdk.cgi/java.time.Instant) 和 [Duration](http://shibboleth.net/cgi-bin/java-jdk.cgi/java.time.Duration)类。

由于配置中对ISO时长表示法的广泛支持，因此更改对于部署人员来说几乎是不可见的。 已经废除一些有直接影响的领域，并将在以后的版本中将其删除：

- 将登录时间戳作为joda-time对象传递回[外部](https://wiki.shibboleth.net/confluence/display/IDP4/ExternalAuthnConfiguration) 登录流中。
- 主要（但不是必需）将joda-time DateTimeFormatter对象用于 [ExpiringPasswordInterceptConfiguration.](https://wiki.shibboleth.net/confluence/display/IDP4/ExpiringPasswordInterceptConfiguration)

还观察到，V3可能没有正确地解释没有“Z”结尾指示符作为兼容的UTC值的时间戳，这不是正确的假设。 在任何情况下，使用此类值都违反SAML规范，并且V4不处理此类值，并且在大多数情况下，拒绝它们。

### 谓词/函数API

整个代码库已从Guava Predicate / Function API转换为Java 8中引入的新功能API接口。此更改已在代码中广泛传播，但我们通过众多实用[utility/parent](https://wiki.shibboleth.net/confluence/display/IDP4/PredefinedBeans)bean和支持类提供了迁移帮助。 在旧的Guava Predicate.apply()签名和新的标准Predicate.test()签名之间进行转换。

我们希望大多数基于Guava谓词的自定义代码和脚本将保持不变，但在调用apply()方法以允许代码清理并防止将来出现问题时可能会产生警告。

在views/login.vm文件的底部附近有个疏忽。如果您碰巧正在使用"not-quite-deprecated"的扩展流功能，则需要在模板中调用"apply"，需要将其手动更改为"test"。 这也适用于V4.1.0之前的新安装，它将包含固定的模板。

#### 当心Guava的 "compose"方法

传统上使用Guava函数在大多数情况下可能会起作用，但是Spring的装配行为中存在一个已知问题，这会中断使用Guava的“ compose”帮助程序。 如果在任何地方都包含这样的参考的自定义bean装配，则可能必须对其进行更新以避免问题。
```xml
<bean class="com.google.common.base.Functions" factory-method="compose">
```
使用该语法通常会导致该bean的两个构造函数参数被交换/反转，从而颠倒了组合的顺序，通常充其量只能产生不好的输出，而最糟糕的是会导致崩溃甚至更具灾难性的结果。

不幸的是，由于Guava转换为Java 8的Function接口的方式，因此没有简单的解决方案。 结果，您能做的最好的事情就是简单地对此进行扫描并将上面的语法转换为：
```xml
<bean parent="shibboleth.Functions.Compose">
```
这在功能上是等效的，应该避免我们在Spring中看到的重新排序行为。

### 配置API

做了大量清理，以消除relying-party.xml文件中，用于配置和重写（overriding）的API中不一致的方法名称。 在少数特定情况下，由于类型不兼容，人们可能会在配置中遇到错误，这可能需要查看适当的Javadoc类。 但根据经验，现在所有属性名称都遵循以下约定：

- p:something 采用文字值，尔值，整数，长整数，字符串，持续时间等。
- p:somethingPredicate 当等效值等于布尔值，采用Java谓词
- p:somethingLookupStrategy 当等效值不等于布尔值，采用Java函数

例如，V3属性signAssertions具有谓词值，而V4版本采用文字布尔值，而signAssertionsPredicate采用谓词。 有许多这样的情况，但并不常用。

### LDAP变化

现在，默认的LDAP实例是UnboundID库，并且不再正式支持JNDI。 将来的版本中将完全不提供JNDI支持，并且我们目标是迁移至 [Ldaptive](http://www.ldaptive.org/)将来版本的原生实例。

此版本包括了配置的更新，特别是在 [LDAPAuthnConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPAuthnConfiguration)中，和[LDAPConnector.](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPConnector)的一小部分。 使用此功能的部署应在可能的情况下检查和更新其配置，及时更新并最大程度地减少将来的更改需求。 在许多情况下，可以直接替换ldap-authn-config.xml文件，而无需对[LDAPAuthnConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPAuthnConfiguration)页中“有关升级的注意事项”中指出的其他更改进行任何替换。

可以使用各种新的（或最近添加的）设置替换以前最常用的`<LDAPProperty>`元素，实现以非JNDI方式下的设置工作。 例如，现在可以直接通过V3.4补丁中添加的`<BinaryAttributes>`元素在[LDAPConnector](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPConnector)中指定二进制属性。 如果遇到无法明显替换的属性，请寻求帮助。

ldaptive  [JAAS](https://wiki.shibboleth.net/confluence/display/IDP4/JAASAuthnConfiguration)模块在超时设置的处理方面有不兼容的更改。 以毫秒为单位指定值将导致DateTime解析异常。 值必须转换为ISO语法（例如2000→PT2S）。

另外，以“idp.pool.LDAP”为前缀的各种LDAP池属性（例如**idp.pool.LDAP.validatePeriod**）以前默认为以秒表示的数字值（例如300 == 5分钟）， 现在将其解释为毫秒。如果将默认值与未修改的V3 ldap-authn-config.xml文件一起使用，则会导致池验证过于频繁。 进一步讨论参见 [LDAPAuthnConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPAuthnConfiguration)页面。

### 密码流扩展

[密码](https://wiki.shibboleth.net/confluence/display/IDP4/PasswordAuthnConfiguration)登录流程已经进行了重大的内部重构，以支持更高程度的灵活性。 重新设计将消除了一种我们认为不被广泛使用的现有扩展机制，描述参见V3文档的“[Custom Back-Ends.](https://wiki.shibboleth.net/confluence/display/IDP30/PasswordAuthnConfiguration#PasswordAuthnConfiguration-CustomBack-Ends)”标题。

### 属性相关变化

V4的主要新增功能是[AttributeRegistry](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeRegistryConfiguration) 服务，该服务取代或至少补充了用于转换属性的[AttributeEncoder](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeEncoderPluginConfiguration)机制。 尽管升级后的系统应保持不变，但建议部署前检查此新功能，并可以利用此功能大大简化其配置。 许多现有的编码器可能是可拆卸的。

- 请注意，由于现有编码器和新规则之间存在重叠，因此新的安装并使用旧配置情况下，可能会导致SAML消息中出现重复的属性。 通过从默认配置中排除新规则来防止升级安装中发生此情况。 当然，反之亦然：如果要应用新的默认规则，则必须在升级后进行该转换。

此新服务的结果是，调整[AttributeEncoders](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeEncoderPluginConfiguration)要求重新加载“ shibboleth.AttributeRegistryService”，而不是像以前的发行版一样重新加载解析程序服务。

万一您为属性定义提供了任何属性显示名称或描述元数据，但**没有**编码器，则兼容性支持**不会**将该显示元数据附加到生成的属性上。 这似乎不太可能，但这是一个变化。

在此版本中，属性ID的规则得到了加强，以排除空格和单引号。 现在是预先告警，这部分可能进一步强化。 我们强烈建议您坚持使用简单的ASCII字符。

围绕[IdPAttributeValue](http://shibboleth.net/cgi-bin/java-idp.cgi/net.shibboleth.idp.attribute.IdPAttributeValue)的类层次结构已得到简化，并消除了泛型的使用。 某些脚本可能会受到方法名称更改的影响。 不再有暴露的通用getValue方法，但通常getNativeValue返回相同的结果为保存范围的值，其中返回Pair。 此更改可防止非字符串值意外地“有损”转换为字符串。

如果一个[InputDataConnector](https://wiki.shibboleth.net/confluence/display/IDP4/InputDataConnector) 由allAttribute获得选项集供给情况下，从[TemplateAttributeDefinition](https://wiki.shibboleth.net/confluence/display/IDP4/TemplateAttributeDefinition) 语法中删除`<SourceAttribute>`元素，将在一些罕见场景中，与现有的结果不一致。 使用此语法要求所有输入都具有相同数量的值（或没有值）。 无需将`<SourceAttribute>`元素与allAttributes结合使用，只需枚举要从连接器获取的属性。

### 元数据变化

由于对某些SAML元数据内容进行了额外的“前期”处理，在[MetadataProvider](https://wiki.shibboleth.net/confluence/display/IDP4/MetadataConfiguration)的初始加载期间，不再容许某些过去被忽略的无效SAML构造，尤其是空XML元素，如`<md:ServiceName/>`, `<md:ServiceDescription/>`, 和`<md:Organization>` 的所有子元素.

根据标准，此类元素是无效的，并且IdP的行为是正确的，但是初始发行版中存在一个错误，该错误会导致NullPointerException而不是忽略该元素。 该崩溃已在V4.0.1中更正。 部署人员通常不应依赖IdP来容忍无效的SAML构造。

### 注销变化

现在，“单点注销”功能包括一些其他控制点，属性以及对默认视图的更改，它们可以更好地控制注销的进行方式，用户是否能够取消/中止它们以及其他要求的“控制流”增强功能。 这些在 [LogoutConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/LogoutConfiguration)主题中进行了讨论。

此外，SAML的SOAP注销已首次实现。遗憾的是，由于与绑定选择方式有关的一个原有的一直按元数据顺序排列相关的错误，这种更改可能会对现有部署产生一些影响。这可能导致选择SOAP优先于前通道绑定的错误。 对此修改已列入到将来的版本中。 随着对控制SOAP注销行为的各个方面的研究，本节将更新更多信息。

### 新的弃用

由于Spring错误导致配置重载问题，我们已弃用了HttpClient父Bean的原始集以及自定义HttpClient配置中从这些Bean继承的所有内容。 我们引入了正确的预定义设置的新父bean，并调整了引用它们的[HttpClientConfiguration ](https://wiki.shibboleth.net/confluence/display/IDP4/HttpClientConfiguration)文档。

现在，依赖于默认HttpClient配置的内部组件依赖于仅供内部使用的系统Bean定义，该定义与以前通过services.properties文件进行配置方法一致。

如果您有从过时的版本继承bean，将产生警告。 它们将从V5中删除。 这些用于配置客户端的缓存版本的多个扩展属性将被删除。

### CSRF预防

IdP包括一项新的[Cross-Site Request Forgery (CSRF) Protection](https://wiki.shibboleth.net/confluence/display/IDP4/Cross-Site+Request+Forgery+%28CSRF%29+Protection) 保护功能，该功能在新安装时默认启用。 如果遇到问题或升级时希望启用它，可以通过idp.csrf.enabled的开关属性实现。 该功能还可以轻松地融入自定义Webflow视图中。

### 可扩展的安装程序

对打包程序最感兴趣的，安装程序现在是用Java构建的，并具有明确的扩展点。 请参阅对V4安装程序 [V4安装程序编程](https://wiki.shibboleth.net/confluence/display/IDP4/Programming+the+V4+Installer)。

### 值得注意的新功能

- 前述新的 [AttributeRegistryConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeRegistryConfiguration) 和[Cross-Site Request Forgery (CSRF) Protection](https://wiki.shibboleth.net/confluence/display/IDP4/Cross-Site+Request+Forgery+%28CSRF%29+Protection)特性
- 通过exportAttributes设置直接从[DataConnectors](https://wiki.shibboleth.net/confluence/display/IDP4/DataConnectorConfiguration)生产可编码属性
- 更灵活的 [PasswordAuthnConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/PasswordAuthnConfiguration)允许将后端组合JAAS样式
- SAML代理登录流程([SAMLAuthnConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/SAMLAuthnConfiguration))和各种相关功能

### 新的对象

- [shibboleth.SameSiteCookieMap](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [shibboleth.authn.LDAP.authenticator](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPAuthnConfiguration)
- [shibboleth.LDAPAuthenticationFactory](https://wiki.shibboleth.net/confluence/display/IDP4/LDAPAuthnConfiguration)
- [shibboleth.authn.discoveryURL](https://wiki.shibboleth.net/confluence/display/IDP4/AuthenticationConfiguration)
- [shibboleth.authn.discoveryURLStrategy](https://wiki.shibboleth.net/confluence/display/IDP4/AuthenticationConfiguration)
- [shibboleth.authn.SAML.*](https://wiki.shibboleth.net/confluence/display/IDP4/SAMLAuthnConfiguration)objects
- [c14n/SAML2ProxyTransform c14n flow](https://wiki.shibboleth.net/confluence/display/IDP4/SAML2ProxyTransformPostLoginC14NConfiguration)
- [shibboleth.ProxyNameTransformPredicate](https://wiki.shibboleth.net/confluence/display/IDP4/SAML2ProxyTransformPostLoginC14NConfiguration)
- [shibboleth.ProxyNameTransformFormats](https://wiki.shibboleth.net/confluence/display/IDP4/SAML2ProxyTransformPostLoginC14NConfiguration)
- [shibboleth.PrincipalProxyRequestMappings](https://wiki.shibboleth.net/confluence/display/IDP4/AuthenticationConfiguration)
- [shibboleth.PrincipalProxyResponseMappings](https://wiki.shibboleth.net/confluence/display/IDP4/AuthenticationConfiguration)
- [shibboleth.authn.Password.Validators ](https://wiki.shibboleth.net/confluence/display/IDP4/PasswordAuthnConfiguration)list and related validators
- Many objects connected to [AttributeRegistryConfiguration](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeRegistryConfiguration)
- shibboleth.SubjectLocalityAddressStrategy
- shibboleth.DefaultVelocityEngineProperties
- shibboleth.VelocityEngineProperties
- [shibboleth.ExceptionResolverViewExtenderFunction](https://wiki.shibboleth.net/confluence/display/IDP4/ErrorHandlingConfiguration)
- [shibboleth.HttpClientFactory](https://wiki.shibboleth.net/confluence/display/IDP4/HttpClientConfiguration)
- [shibboleth.FileCachingHttpClientFactory](https://wiki.shibboleth.net/confluence/display/IDP4/HttpClientConfiguration)
- [shibboleth.MemoryCachingHttpClientFactory](https://wiki.shibboleth.net/confluence/display/IDP4/HttpClientConfiguration)

### 新的属性

- [dp.cookie.sameSite](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [idp.cookie.sameSiteCondition](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [idp.csrf.enabled](https://wiki.shibboleth.net/confluence/display/IDP4/Cross-Site+Request+Forgery+%28CSRF%29+Protection)
- [idp.csrf.token.parameter](https://wiki.shibboleth.net/confluence/display/IDP4/Cross-Site+Request+Forgery+%28CSRF%29+Protection)
- [idp.sealer.keyStrategy](https://wiki.shibboleth.net/confluence/display/IDP4/SecurityConfiguration)
- [idp.service.metadata.enableByReferenceFilters](https://wiki.shibboleth.net/confluence/display/IDP4/ReloadableServices)
- [idp.service.attribute.registry.*](https://wiki.shibboleth.net/confluence/display/IDP4/ReloadableServices)
- [idp.service.attribute.resolver.stripNulls](https://wiki.shibboleth.net/confluence/display/IDP4/AttributeResolverConfiguration)
- [idp.service.managedBean.*](https://wiki.shibboleth.net/confluence/display/IDP4/ReloadableServices)
- [idp.logout.promptUser](https://wiki.shibboleth.net/confluence/display/IDP4/LogoutConfiguration)
- [idp.cas.relyingPartyIdFromMetadata](https://wiki.shibboleth.net/confluence/display/IDP4/CASServiceSAMLMetadata)
- [idp.saml.honorWantAssertionsSigned](https://wiki.shibboleth.net/confluence/display/IDP4/SAML2SSOConfiguration)
- [idp.saml.honorWantAuthnRequestsSigned](https://wiki.shibboleth.net/confluence/display/IDP4/SAML2SSOConfiguration)
- idp.velocity.space.gobbling

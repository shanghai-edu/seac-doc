# 属性定义

在配置完属性连接器后，我们需要定义具体的属性，来完成属性的获取。Shibboleth 支持多种不同的属性获取模式

### AttributeEncoder
在 IdP4 中，现在我们可以在 `/opt/shibboleth-idp/conf/attributes` 中提前定义好 AttributeEncoder, 这样在 AttributeDefinition 中就可以避免重复配置，显得配置文件臃肿。

为了避免冲突，修改 `/opt/shibboleth-idp/conf/attributes/default-rules.xml`，将默认 `import` 注释掉，修改为 [seac.xml](https://eac.cloud.sh.edu.cn/download/seac.xml) 和 [carsi.xml](https://eac.cloud.sh.edu.cn/download/carsi.xml)

- 注意如果 tomcat 的权限不是 root，在上传 `seac.xml` 和 `carsi.xml` 后务必更新下相关目录的权限，例如：
```
chown -R tomcat:tomcat /opt/shibboleth-idp/
```

```xml
<!--
    <import resource="inetOrgPerson.xml" />
    <import resource="eduPerson.xml" />
    <import resource="eduCourse.xml" />
    <import resource="samlSubject.xml" />
-->
    <import resource="seac.xml" />
    <import resource="carsi.xml" />
```

这两个文件联盟已经提前准备好，点击前述的链接直接下载即可，无需手动配置。

### AttributeDefinition
#### Simple 模式
Simple 是最简单的属性获取模式。只需要定义属性的数据来源，和属性的名称即可。

例如以下的3个属性，`uid` 和 `cn` 来自于 `myLDAP` 数据连接器。，而 `domainName` 则来自基于 `staticAttributes` 数据连接器。
```xml
   <AttributeDefinition xsi:type="Simple" id="uid">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
   </AttributeDefinition>


   <AttributeDefinition xsi:type="Simple" id="cn">
        <InputDataConnector ref="myLDAP" attributeNames="cn"/>
   </AttributeDefinition>

   <AttributeDefinition xsi:type="Simple" id="domainName">
        <InputDataConnector ref="staticAttributes" attributeNames="domainName"/>
   </AttributeDefinition>
```

#### SubjectDerivedAttribute 模式
在进行 CAS/OAuth2 对接时，我们通过`xsi:type="SubjectDerivedAttribute"` 模式，从插件中获取属性的配置，例如下面的示例的映射关系如下所示：

- ID_NUMBER -> uid
- USER_NAME -> cn
- TYPE_NAME -> typeOf

```xml
   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="uid" principalAttributeName="ID_NUMBER">
   </AttributeDefinition>

   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="cn" principalAttributeName="USER_NAME">
   </AttributeDefinition>

   <AttributeDefinition xsi:type="SubjectDerivedAttribute" id="typeOf" principalAttributeName="TYPE_NAME">
   </AttributeDefinition>
```

#### ScriptedAttribute 模式
Shibboleth 支持内嵌脚本的方式来对属性进行自定义调整，如下例所示，表示根据 `uid` 的长度来进行身份的判断
```xml
   <AttributeDefinition xsi:type="ScriptedAttribute" id="typeOf">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
        <Script><![CDATA[
        var localpart = "";
        if(uid.getValues().get(0).length==8) localpart = "teacher";
        else localpart = "student";
        typeOf.addValue(localpart);
            ]]></Script>
    </AttributeDefinition>
``` 

#### 完整示例
[CARSI-IdP 接入](https://eac.cloud.sh.edu.cn/document/idp4/idp_carsi.html) 提供了一份同时接入 SEAC 和 CARSI 的完整配置示例。

可以在此基础上进一步调整，供参考。
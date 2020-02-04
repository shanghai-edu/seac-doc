# 2.1.3 属性对接

idP 能够在认证时去读取用户的身份属性信息，从而提供给应用更多的内容，进而对用户提供更多层次的服务。

可以在 ```IDP_HOME/conf/attribute-resolver.xml```

#### 定义 idP 所需要获取的属性

```
    <resolver:AttributeDefinition xsi:type="ad:Simple" id="uid" sourceAttributeID="uid">
        <resolver:Dependency ref="myLDAP" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:uid" />
       <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:2.5.4.2" friendlyName="uid" />
	</resolver:AttributeDefinition>
    <resolver:AttributeDefinition xsi:type="ad:Simple" id="cn" sourceAttributeID="cn">
        <resolver:Dependency ref="myLDAP" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:cn" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:2.5.4.3" friendlyName="cn" />
    </resolver:AttributeDefinition>
    <resolver:AttributeDefinition xsi:type="ad:Simple" id="email" sourceAttributeID="mail">
        <resolver:Dependency ref="myLDAP" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:email" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:0.9.2342.19200300.100.1.3" friendlyName="email" />
    </resolver:AttributeDefinition>
    <resolver:AttributeDefinition xsi:type="ad:Simple" id="domainName" sourceAttributeID="domainName">
        <resolver:Dependency ref="staticAttributes" />
        <resolver:AttributeEncoder xsi:type="enc:SAML1String" name="urn:mace:dir:attribute-def:domainName" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="urn:oid:2.5.4.5" friendlyName="domainName" />
    </resolver:AttributeDefinition>
```

这里的xsi:type="enc:SAML2String" name="urn:oid:2.5.4.2"，需要遵循联盟内的属性标准，可与联盟的管理单位联系确认。

##### 采用 ldap 获取属性
修改属性连接器配置，此处为 ```myLDAP``` ,t添加 ldap 相关配置信息。如果 ldap 查询响应过慢会导致认证出错，则应增加 ```searchTimeLimit="30000"``` 限制查询时间，保障至少认证通过。
示例
```
    <resolver:DataConnector id="myLDAP" xsi:type="dc:LDAPDirectory"
        ldapURL="ldap://ldap.example.org:389" 
        baseDN="dc=example,dc=org" 
        principal="<ldapservicedn>"
        principalCredential="<password>"
        searchTimeLimit="30000">
        <dc:FilterTemplate>
            <![CDATA[
                (uid=$requestContext.principalName)
            ]]>
        </dc:FilterTemplate>
        <dc:ReturnAttributes>uid cn mail</dc:ReturnAttributes>
    </resolver:DataConnector>

```
#### 采用 静态连接器 获取属性
有些属性信息是全局的，比如某个 idp 中用户的所属来源域(domainName)。此时可以给他赋予静态的属性

```
        <resolver:DataConnector id="staticAttributes" xsi:type="dc:Static">
                <dc:Attribute id="domainName">
                        <dc:Value>example.org</dc:Value>
                </dc:Attribute>
        </resolver:DataConnector>
```

##### 采用 jbdc 的方式获取属性
首先将所使用数据库对应的 JDBC 的 驱动(jar 包) 放入 ```IDP_HOME/lib``` 中。

然后修改属性连接器配置，此处为 ```generic``` ,添加数据库的相关配置信息。
示例
```
    <resolver:DataConnector id="generic" xsi:type="dc:RelationalDatabase">
        <dc:ApplicationManagedConnection jdbcDriver="com.mysql.jdbc.Driver"
                                         jdbcURL="jdbc:mysql://mysql:3306/account"
                                         jdbcUserName=”<databaseuser>" 
                                         jdbcPassword="<password>t" />
        <dc:QueryTemplate>
            <![CDATA[
                SELECT * FROM auth_account_identity WHERE account = '$requestContext.principalName'
            ]]>
        </dc:QueryTemplate>
        <dc:Column columnName="account" attributeID="uid" type="String" />
        <dc:Column columnName="name" attributeID="cn" type="String" />
        <dc:Column columnName="user_style" attributeID="typeOf" type="String" />
    </resolver:DataConnector>

```

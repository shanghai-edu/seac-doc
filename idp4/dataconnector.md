# 数据连接器

### 静态属性
静态属性是指配置为固定值的静态属性，以下是一个静态属性的连接器
```xml
    <DataConnector id="staticAttributes" xsi:type="Static">
        <Attribute id="domainName">
            <Value>ecnu.edu.cn</Value>
        </Attribute>
        <Attribute id="eduPersonEntitlement">
            <Value>urn:mace:dir:entitlement:common-lib-terms</Value>
        </Attribute>
    </DataConnector>
```
### LDAP 获取属性
以下是从 LDAP 获取属性的参考配置的连接器
```xml
    <DataConnector id="myLDAP" xsi:type="LDAPDirectory"
        ldapURL="%{idp.attribute.resolver.LDAP.ldapURL}"
        baseDN="%{idp.attribute.resolver.LDAP.baseDN}" 
        principal="%{idp.attribute.resolver.LDAP.bindDN}"
        principalCredential="%{idp.attribute.resolver.LDAP.bindDNCredential}"
        useStartTLS="%{idp.attribute.resolver.LDAP.useStartTLS:true}"
        connectTimeout="%{idp.attribute.resolver.LDAP.connectTimeout}"
        responseTimeout="%{idp.attribute.resolver.LDAP.responseTimeout}">
        <FilterTemplate>
            <![CDATA[
                %{idp.attribute.resolver.LDAP.searchFilter}
            ]]>
        </FilterTemplate>
        <ConnectionPool
            minPoolSize="%{idp.pool.LDAP.minSize:3}"
            maxPoolSize="%{idp.pool.LDAP.maxSize:10}"
            blockWaitTime="%{idp.pool.LDAP.blockWaitTime:PT3S}"
            validatePeriodically="%{idp.pool.LDAP.validatePeriodically:true}"
            validateTimerPeriod="%{idp.pool.LDAP.validatePeriod:PT5M}"
            validateDN="%{idp.pool.LDAP.validateDN:}"
            validateFilter="%{idp.pool.LDAP.validateFilter:(objectClass=*)}"
            expirationTime="%{idp.pool.LDAP.idleTime:PT10M}"/>
    </DataConnector>
```

### 插件（CAS/Oauth）获取属性

在最新的wiki中已经指出了，不推荐使用SubjectDerivedAttribute模式，转为使用[SubjectDataConnector](https://wiki.shibboleth.net/confluence/display/IDP4/SubjectDataConnector)模式。

其中exportAttributes为从插件侧获取的属性名。

以下是从 CAS/Oauth 获取属性的参考配置的连接器

```xml
<DataConnector id="mySubjectData" xsi:type="Subject" exportAttributes="UID XM TYPEOF" />
```



### 数据库获取属性

shibboleth 也支持从数据库中读取属性

下载对应的驱动放入lib文件夹中，此处实例为mysql

安装 Java 的 MysQL 驱动，注意在 CentOS7 上用 yum 安装时，会自动依赖安装 java8，装完别忘记把 java8 卸载掉。
```
yum -y install mysql-connector-java
cp /usr/share/java/mysql-connector-java.jar /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/mysql-connector-java.jar
chown tomcat:tomcat /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/mysql-connector-java.jar
yum remove java-1.8*
/opt/shibboleth-idp/bin/build.sh
```

修改 `global.xml`
```xml
   <bean id="MysqlSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close"
          p:driverClassName="%{datasource.driverClass}"
          p:url="%{datasource.jdbcUrl}" 
          p:username="%{datasource.user}" 
          p:password="%{datasource.password}" 
          p:maxIdle="5"  
          p:testOnBorrow="true"
          p:validationQuery="select 1" 
          p:validationQueryTimeout="5" />
```

修改 `idp.properties`
```ini
datasource.driverClass=com.mysql.jdbc.Driver
datasource.jdbcUrl=jdbc:mysql://database.url:3306/yourdatabase
datasource.user=user
datasource.password=password
```

修改 `attributes-resolver.xml`
配置属性连接器
```xml
<DataConnector xsi:type="RelationalDatabase" id="mysqldb" >
       <BeanManagedConnection>MysqlSource</BeanManagedConnection>
       <QueryTemplate>
       <![CDATA[
              SELECT * FROM you_table WHERE Username ='$resolutionContext.principal'
              ]]>
       </QueryTemplate>

       <Column columnName="UID" attributeID="uid" />
</DataConnector>
```
配置属性定义
```xml
<AttributeDefinition id="uid" xsi:type="Simple">
       <InputDataConnector ref="mysqldb" attributeNames="uid"/>
</AttributeDefinition>
```
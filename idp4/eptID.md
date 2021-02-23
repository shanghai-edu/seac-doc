# eptID（NameID 生成） 配置

eptID —— 即  eduPersonTargtedID ，他是根据用户名，IdP 的 entityID 和 SP 的 entityID 再加盐 Hash 后得到的脱敏字符串。

对于某一个特定的 SP 而言，他可用于在不获取用户真实信息的前提下，区分不同用户。由于不同的 SP 之间得到的 eptID 不同，因此无法相互关联挖掘，从而保障用户隐私不被泄露。

#### NameID 生成
修改 `saml-nameid.properties` ，配置 `nameid` 生成所需的各项参数。

注意如果是从 IdP3 升级到 IdP4，则务必确保 ComputedId 切换而来，则务必保证 `idp.persistentId.sourceAttribute`，`idp.persistentId.salt`, `idp.persistentId.encoding` 与之前 ComutedId 中相关配置一致。否则会导致新生成的 eptID 与先前不一致。

特别注意 `idp.persistentId.encoding` 新安装的默认值是 `BASE32`，而在 IdP3 中默认是 `BASE64`，**务必**注意升级前后的一致性。
```ini
idp.persistentId.sourceAttribute = eduPersonPrincipalName
idp.persistentId.salt = xxxxxxxxxxxxxxxxxxxx
idp.persistentId.encoding = BASE32
```

修改 `saml-nameid.xml` ,取消 `<ref bean="shibboleth.SAML2PersistentGenerator" />` 部分的注释。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
                           
       default-init-method="initialize"
       default-destroy-method="destroy">

    <!-- ========================= SAML NameID Generation ========================= -->

    <!--
    These generator lists handle NameID/Nameidentifier generation going forward. By default,
    transient IDs for both SAML versions are enabled. The commented examples are for persistent IDs
    and generating more one-off formats based on resolved attributes. The suggested approach is to
    control their use via release of the underlying source attribute in the filter policy rather
    than here, but you can set a property on any generator called "activationCondition" to limit
    use in the most generic way.
    
    Most of the relevant configuration settings are controlled using properties; an exception is
    the generation of arbitrary/custom formats based on attribute information, examples of which
    are shown below.
    
    -->

    <!-- SAML 2 NameID Generation -->
    <util:list id="shibboleth.SAML2NameIDGenerators">

        <ref bean="shibboleth.SAML2TransientGenerator" />

        <!-- Uncommenting this bean requires configuration in saml-nameid.properties. -->
        
        <ref bean="shibboleth.SAML2PersistentGenerator" />
        

        <!--
        <bean parent="shibboleth.SAML2AttributeSourcedGenerator"
            p:omitQualifiers="true"
            p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"
            p:attributeSourceIds="#{ {'mail'} }" />
        -->

    </util:list>

    <!-- SAML 1 NameIdentifier Generation -->
    <util:list id="shibboleth.SAML1NameIdentifierGenerators">

        <ref bean="shibboleth.SAML1TransientGenerator" />

        <!--
        <bean parent="shibboleth.SAML1AttributeSourcedGenerator"
            p:omitQualifiers="true"
            p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"
            p:attributeSourceIds="#{ {'mail'} }" />
        -->

    </util:list>

</beans>
```

#### ComputedId 模式
ComputedId 模式下，每次属性释放时都会自动计算生成 eptID，但不会对 eptID 进行存储。

修改 `attribute-resolver.xml` 配置文件，添加 `ComputedId"` 相关的 `DataConnector`，其相关参数则会直接引用 `saml-nameid.properties` 中的配置。

```xml
<AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
    <InputDataConnector ref="ComputedIDConnector" attributeNames="computedID"/>
    <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
</AttributeDefinition>

<DataConnector id="ComputedIDConnector" xsi:type="ComputedId"
            generatedAttributeID="computedID"
            salt="%{idp.persistentId.salt}"
            algorithm="%{idp.persistentId.algorithm:SHA}"
            encoding="%{idp.persistentId.encoding:BASE32}">
        <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}" />
</DataConnector>
```

`eduPersonPrincipalName` 作为生成 `eduPersonTargetedID` 的源属性，将 IdP 对外释放的 ePTID 属性和 `eduPersonPrincipalName` 属性进行关联。

#### 数据库模式
数据库模式下，IdP 会将生成的 NameID（我们会创建一个 `shibboleth.SAML2PersistentGenerator` 的 NameID 生成器，此时他生成的即是 etpID），写入到数据库中。

后续可以通过数据库来查询的方式，来反查某个 eptID 对应的实际用户名。
##### 安装 mysql5.7
```
cd /etc/yum.repos.d/
wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
```
修改 `/etc/my.cnf`
```ini
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

character-set-server=utf8mb4

server_id=100
log_bin = /var/lib/mysql/bin-log
log_bin_index = /var/lib/mysql/mysql-bin.index
expire_logs_days = 7

[client]
default-character-set = utf8mb4
```
启动并初始化 mysql
```
systemctl start mysql
systemctl enable mysqld
mysql_secure_installation
```
创建一个数据库实例 idp_db，并创建 idp 用户 
```sql
mysql> > create database idp_db;
mysql> > CREATE TABLE shibpid (
    localEntity VARCHAR(255) NOT NULL,
    peerEntity VARCHAR(255) NOT NULL,
    persistentId VARCHAR(50) NOT NULL,
    principalName VARCHAR(50) NOT NULL,
    localId VARCHAR(50) NOT NULL,
    peerProvidedId VARCHAR(50) NULL,
    creationDate TIMESTAMP NOT NULL,
    deactivationDate TIMESTAMP NULL,
    PRIMARY KEY (localEntity, peerEntity, persistentId)
);
mysql> CREATE INDEX shibpid_getbysourcevalue_index ON shibpid(localEntity, peerEntity, localId, deactivationDate);
mysql> > create user 'idp'@'localhost' identified by 'password';
mysql> > grant all on idp_db.shibpid to 'idp'@'localhost';
mysql> > flush privileges;
mysql> > exit;
```

##### 配置 IdP

安装 Java 的 MysQL 驱动，注意在 CentOS7 上用 yum 安装时，会自动依赖安装 java8，装完别忘记把 java8 卸载掉。
```
yum -y install mysql-connector-java
cp /usr/share/java/mysql-connector-java.jar /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/mysql-connector-java.jar
chown tomcat:tomcat /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/mysql-connector-java.jar
yum remove java-1.8*
/opt/shibboleth-idp/bin/build.sh
```
修改`global.xml`， 配置数据库连接
```xml
<bean id="MyDataSource" class="org.apache.commons.dbcp2.BasicDataSource"
p:driverClassName="com.mysql.jdbc.Driver"
p:url="jdbc:mysql://localhost:3306/idp_db"
p:username="username"
p:password="password"   
p:maxIdle="5"
p:maxWaitMillis="15000"
p:testOnBorrow="true"
p:validationQuery="select 1"
p:validationQueryTimeout="5" />
```

修改 `saml-nameid.properties` ，配置 `nameid` 生成所以来的数据库源。
```ini
idp.persistentId.sourceAttribute = eduPersonPrincipalName
idp.persistentId.salt = xxxxxxxxxxxxxxxxxxxx
idp.persistentId.encoding = BASE64
idp.persistentId.dataSource = MyDataSource
```
修改 `attribute-resolver.xml`，增加相关配置
```xml
<AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
    <InputDataConnector ref="myStoredId" attributeNames="persistentID"/>
    <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
    <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
</AttributeDefinition>
<DataConnector id="myStoredId" xsi:type="StoredId" generatedAttributeID="persistentID" salt="%{idp.persistentId.salt}" queryTimeout="0">
    <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}"/>
    <BeanManagedConnection>MyDataSource</BeanManagedConnection>
</DataConnector>
```

#### 隐藏属性确认

在 IdP4 中，eptID 不再是默认的隐藏属性，因此在用户确认页面上会看到这个字段。

如果需要隐藏掉他的话，则修改 `/opt/shibboleth-idp/conf/intercept/consent-intercept-config.xml` 配置文件，在 `shibboleth.consent.attribute-release.BlacklistedAttributeIDs` 增加配置即可。
```xml
    <util:list id="shibboleth.consent.attribute-release.BlacklistedAttributeIDs">
        <value>samlPairwiseID</value>
        <value>eduPersonTargetedID</value>
    </util:list>
```
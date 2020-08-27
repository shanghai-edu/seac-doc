# 2.3.12 连接数据库读取属性

shibboleth idp支持从数据库中读取属性

下载对应的驱动放入lib文件夹中，此处实例为mysql

```
yum -y install mysql-connector-java
cp /usr/share/java/mysql-connector-java.jar /opt/shibboleth-idp/edit-webapp/WEB-INF/lib/mysql-connector-java.jar
/opt/shibboleth-idp/bin/build.sh
```

修改global.xml
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

修改idp.properties
```properties
datasource.driverClass=com.mysql.jdbc.Driver
datasource.jdbcUrl=jdbc:mysql://database.url:3306/yourdatabase
datasource.user=user
datasource.password=password
```

修改attributes-resolver.xml
```xml
<AttributeDefinition id="uid" xsi:type="Simple">
<InputDataConnector ref="mysqldb" attributeNames="uid"/>
<AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:2.5.4.2" encodeType="false"/>
<AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:2.5.4.2" friendlyName="uid" encodeType="false"/>
</AttributeDefinition>


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
# CARSI-IdP 接入

对于高校 IdP 而言，通常需要同时接入 CARSI 和 SEAC。由于协议的相通性，只需要一个 IdP 服务器，修改部分配置即可完成接入。

本文列出一个同时接入 CARSI 和 SEAC 的配置示例，供调试参考

### attributes/default-rules.xml
修改 `/opt/shibboleth-idp/conf/attributes/default-rules.xml` ，确保已经定义 [seac.xml](https://eac.cloud.sh.edu.cn/download/seac.xml) 和 [carsi.xml](https://eac.cloud.sh.edu.cn/download/carsi.xml) 的属性。

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

    <!-- Default Attribute transcoding rules. -->

    <!--
    Many if not most of these attributes are not suited or may even be actively discouraged
    from use in federated protocols, but this is merely a set of well-known definitions, not
    a recommended set to support or use.
    -->

<!--
    <import resource="inetOrgPerson.xml" />
    <import resource="eduPerson.xml" />
    <import resource="eduCourse.xml" />
    <import resource="samlSubject.xml" />
-->
    <import resource="seac.xml" />
    <import resource="carsi.xml" />

</beans>
```

### attribute-resolver
修改 `attribute-resolver.xml` 确保两个联盟所需的属性均已经定义
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 
    This file is an EXAMPLE configuration file. While the configuration
    presented in this example file is semi-functional, it isn't very
    interesting. It is here only as a starting point for your deployment
    process.
    
    Very few attribute definitions and data connectors are demonstrated,
    and the data is derived statically from the logged-in username and a
    static example connector.

    Attribute-resolver-full.xml contains more examples of attributes,
    encoders, and data connectors. Deployers should refer to the Shibboleth
    documentation for a complete list of components and their options.
-->
<AttributeResolver
        xmlns="urn:mace:shibboleth:2.0:resolver" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">


    <!-- ========================================== -->
    <!--      Attribute Definitions                 -->
    <!-- ========================================== -->

   <AttributeDefinition xsi:type="ScriptedAttribute" id="eduPersonScopedAffiliation">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
        <Script><![CDATA[
        var localpart = "";
        if(uid.getValues().get(0).length==8) localpart = "teacher";
        else localpart = "student";
        eduPersonScopedAffiliation.addValue(localpart + "@%{idp.scope}");
            ]]></Script>
    </AttributeDefinition>

   <AttributeDefinition xsi:type="Simple" id="uid">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
   </AttributeDefinition>

   <AttributeDefinition xsi:type="Simple" id="cn">
        <InputDataConnector ref="myLDAP" attributeNames="cn"/>
   </AttributeDefinition>

   <AttributeDefinition xsi:type="Simple" id="domainName">
        <InputDataConnector ref="staticAttributes" attributeNames="domainName"/>
   </AttributeDefinition>

   <AttributeDefinition xsi:type="ScriptedAttribute" id="typeOf">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
        <Script><![CDATA[
        var localpart = "";
        if(uid.getValues().get(0).length==8) localpart = "teacher";
        else localpart = "student";
        typeOf.addValue(localpart);
            ]]></Script>
    </AttributeDefinition>

    <AttributeDefinition id="eduPersonEntitlement" xsi:type="Simple">
        <InputDataConnector ref="staticAttributes" attributeNames="eduPersonEntitlement" />
    </AttributeDefinition>

    <AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
        <InputDataConnector ref="myStoredId" attributeNames="persistentID"/>
        <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
        <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
    </AttributeDefinition>

    <!-- ========================================== -->
    <!--      Data Connectors                       -->
    <!-- ========================================== -->

    <DataConnector id="myStoredId" xsi:type="StoredId" generatedAttributeID="persistentID" salt="%{idp.persistentId.salt}" queryTimeout="0">
        <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}"/>
        <BeanManagedConnection>MyDataSource</BeanManagedConnection>
    </DataConnector>

    <DataConnector id="ComputedIDConnector" xsi:type="ComputedId"
            generatedAttributeID="computedID"
            salt="%{idp.persistentId.salt}"
            algorithm="%{idp.persistentId.algorithm:SHA}"
            encoding="%{idp.persistentId.encoding:BASE32}">
        <InputAttributeDefinition ref="%{idp.persistentId.sourceAttribute}" />
    </DataConnector>

    <DataConnector id="staticAttributes" xsi:type="Static">
        <Attribute id="domainName">
            <Value>xxx.edu.cn</Value>
        </Attribute>
        <Attribute id="eduPersonEntitlement">
            <Value>urn:mace:dir:entitlement:common-lib-terms</Value>
        </Attribute>
    </DataConnector>

        <DataConnector id="myLDAP" xsi:type="LDAPDirectory"
        ldapURL="%{idp.attribute.resolver.LDAP.ldapURL}"
            baseDN="%{idp.attribute.resolver.LDAP.baseDN}" 
        principal="%{idp.attribute.resolver.LDAP.bindDN}"
            principalCredential="%{idp.attribute.resolver.LDAP.bindDNCredential}"
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
                expirationTime="%{idp.pool.LDAP.idleTime:PT10M}"
                failFastInitialize="%{idp.pool.LDAP.failFastInitialize:false}" />
        </DataConnector>

</AttributeResolver>
```
### attribute-filter
修改 `attribute-filter.xml` 确保两个联盟的属性都已经得到释放，注意属性策略可能会随着联盟发展进一步更新。建议采取[自动同步方案](https://eac.cloud.sh.edu.cn/document/idp3/idp_filter.html)获取最新的属性策略。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 
    This file is an EXAMPLE policy file.  While the policy presented in this 
    example file is illustrative of some simple cases, it relies on the names of
    non-existent example services and the example attributes demonstrated in the
    default attribute-resolver.xml file.

    This example does contain some usable "general purpose" policies that may be
    useful in conjunction with specific deployment choices, but those policies may
    not be applicable to your specific needs or constraints.    
-->
<AttributeFilterPolicyGroup id="ShibbolethFilterPolicy"
        xmlns="urn:mace:shibboleth:2.0:afp"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd">

     <AttributeFilterPolicy id="carsiAttrFilterPolicy">
         <PolicyRequirementRule xsi:type="ANY" />
         <AttributeRule attributeID="eduPersonScopedAffiliation" permitAny="true" />
         <AttributeRule attributeID="eduPersonTargetedID" permitAny="true" />
     </AttributeFilterPolicy>

     <AttributeFilterPolicy id="carsiAttrFilterToSPPolicy">
         <PolicyRequirementRule xsi:type="OR">
             <Rule xsi:type="Requester" value="https://sptest.pku.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://dspre.carsi.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://ds.carsi.edu.cn/shibboleth-sp/carsifed" />
             <Rule xsi:type="Requester" value="https://sdauth.sciencedirect.com/" />
             <Rule xsi:type="Requester" value="https://sptest.ecnu.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://whoami.cloud.sh.edu.cn/shibboleth" />
         </PolicyRequirementRule>
         <AttributeRule attributeID="eduPersonEntitlement" permitAny="true" />
     </AttributeFilterPolicy>

     <AttributeFilterPolicy id="seaciAttrFilterToSPPolicy">
         <PolicyRequirementRule xsi:type="OR">
             <Rule xsi:type="Requester" value="https://sptest.ecnu.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="Eduroam" />
             <Rule xsi:type="Requester" value="http://mooc.sjtu.edu.cn/SummerElect/Login" />
             <Rule xsi:type="Requester" value="https://kxxfx.shec.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="age06-sp" />
             <Rule xsi:type="Requester" value="https://sp.neumedias.com" />
             <Rule xsi:type="Requester" value="https://cea.shec.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="usst-onthehub" />
             <Rule xsi:type="Requester" value="https://sp.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://service.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://sp-huiyuan.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://sp-sjtu.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://sp.etextbook.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://whoami.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://sog-test.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://sog-seman.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://sog-zongguan.cloud.sh.edu.cn/shibboleth" />
             <Rule xsi:type="Requester" value="https://resource.cloud.sh.edu.cn/shibboleth" />
         </PolicyRequirementRule>
            <AttributeRule attributeID="uid" permitAny="true" />
            <AttributeRule attributeID="cn" permitAny="true" />
            <AttributeRule attributeID="domainName" permitAny="true" />
            <AttributeRule attributeID="typeOf" permitAny="true" />
     </AttributeFilterPolicy>
</AttributeFilterPolicyGroup>
```

### metadata-provider
修改 `metadata-provider.xml` , 确保两个联盟的 metadata 都已经获取
```xml
    <MetadataProvider id="HTTPMetadata"
         xsi:type="FileBackedHTTPMetadataProvider"
         backingFile="/opt/shibboleth-idp/metadata/carsifed-metadata.xml"
         minRefreshDelay="PT5M"
         maxRefreshDelay="PT10M"
         metadataURL="https://www.carsi.edu.cn/carsimetadata/carsifed-metadata.xml"> 
        <MetadataFilter xsi:type="SignatureValidation" certificateFile="/opt/shibboleth-idp/credentials/dsmeta.pem" />
        <MetadataFilter xsi:type="EntityRoleWhiteList">
            <RetainedRole>md:SPSSODescriptor</RetainedRole>
        </MetadataFilter>
    </MetadataProvider>
    <MetadataProvider id="HTTPMetadataShec"
                      xsi:type="FileBackedHTTPMetadataProvider"
                      backingFile="/opt/shibboleth-idp/metadata/ds.shec.edu.cn.xml"
                      minRefreshDelay="PT5M" 
                      maxRefreshDelay="PT10M" 
                      metadataURL="https://ds.shec.edu.cn/metadata.xml">
    </MetadataProvider>
```
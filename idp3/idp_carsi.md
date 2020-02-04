# 2.3.7 CARSI-IdP 接入

对于高校 IdP 而言，通常需要同时接入 CARSI 和 SEAC。由于协议的相通性，只需要一个 IdP 服务器，修改部分配置即可完成接入。本文列出需要修改配置的部分，供调试参考

### attribute-resolver
在 `attribute-resolver.xml` 中，增加属性获取的配置，确保两个联盟所需的属性都已经获取
```xml

   <AttributeDefinition xsi:type="ScriptedAttribute" id="eduPersonScopedAffiliation">
        <InputDataConnector ref="myLDAP" attributeNames="userType"/>
        <Script><![CDATA[
        var localpart = "";
        if(userType.getValues().get(0)=="教师") localpart = "staff";
        else if(userType.getValues().get(0)=="全日制学生") localpart = "student";
        else localpart = "other";
        eduPersonScopedAffiliation.addValue(localpart + "@%{idp.scope}");
            ]]></Script>
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonScopedAffiliation" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.9" friendlyName="eduPersonScopedAffiliation" encodeType="false" />
    </AttributeDefinition>

   <AttributeDefinition xsi:type="Simple" id="uid">
        <InputDataConnector ref="myLDAP" attributeNames="sAMAccountName"/>
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:uid" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.2" friendlyName="uid" encodeType="false" />
   </AttributeDefinition>


   <AttributeDefinition xsi:type="Simple" id="cn">
        <InputDataConnector ref="myLDAP" attributeNames="displayName"/>
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:cn" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.3" friendlyName="cn" encodeType="false" />
   </AttributeDefinition>

   <AttributeDefinition xsi:type="Simple" id="domainName">
        <InputDataConnector ref="staticAttributes" attributeNames="domainName"/>
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:domainName" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.5" friendlyName="domainName" encodeType="false" />
   </AttributeDefinition>


   <AttributeDefinition xsi:type="ScriptedAttribute" id="typeOf">
        <InputDataConnector ref="myLDAP" attributeNames="userType"/>
        <Script><![CDATA[
        var localpart = "";
        if(userType.getValues().get(0)=="教师") localpart = "teacher";
        else if(userType.getValues().get(0)=="全日制学生") localpart = "student";
        else localpart = "other";
        typeOf.addValue(localpart);
            ]]></Script>
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:typeOf" encodeType="false" />
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:2.5.4.100.2" friendlyName="typeOf" encodeType="false" />
    </AttributeDefinition>

    <AttributeDefinition id="eduPersonEntitlement" xsi:type="Simple">
        <InputDataConnector ref="staticAttributes" attributeNames="eduPersonEntitlement" />
        <AttributeEncoder xsi:type="SAML1String" name="urn:mace:dir:attribute-def:eduPersonEntitlement" encodeType="false"/>
        <AttributeEncoder xsi:type="SAML2String" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.7" friendlyName="eduPersonEntitlement" encodeType="false"/>
    </AttributeDefinition>

    <AttributeDefinition id="eduPersonTargetedID" xsi:type="SAML2NameID" nameIdFormat="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">
        <InputDataConnector ref="ComputedIDConnector" attributeNames="computedID"/>
        <AttributeEncoder xsi:type="SAML1XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" encodeType="false"/>
        <AttributeEncoder xsi:type="SAML2XMLObject" name="urn:oid:1.3.6.1.4.1.5923.1.1.1.10" friendlyName="eduPersonTargetedID" encodeType="false"/>
    </AttributeDefinition>
    <DataConnector id="ComputedIDConnector" xsi:type="ComputedId" generatedAttributeID="computedID" salt="fECMt30aWePJN3fcwg0XVSHwgfBRfz+2XPZXA/4T5uw=" encoding="BASE64">
        <InputAttributeDefinition ref="eduPersonPrincipalName" />
    </DataConnector>

	<DataConnector id="staticAttributes" xsi:type="Static">
		<Attribute id="domainName">
			<Value>xxx.edu.cn</Value>
		</Attribute>
        <Attribute id="eduPersonEntitlement">
            <Value>urn:mace:dir:entitlement:common-lib-terms</Value>
        </Attribute>
	</DataConnector>   

```
### attribute-filter
修改 `attribute-filter.xml` 确保两个联盟的属性都已经得到释放
```xml
        <AttributeFilterPolicy id="Per-Attribute-singleValued">
            <PolicyRequirementRule xsi:type="ANY" />
            <AttributeRule attributeID="eduPersonScopedAffiliation" permitAny="true" />
            <AttributeRule attributeID="eduPersonTargetedID" permitAny="true" />
            <AttributeRule attributeID="eduPersonEntitlement" permitAny="true" />
            <AttributeRule attributeID="uid" permitAny="true" />
            <AttributeRule attributeID="cn" permitAny="true" />
            <AttributeRule attributeID="domainName" permitAny="true" />
            <AttributeRule attributeID="typeOf" permitAny="true" />
        </AttributeFilterPolicy>
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
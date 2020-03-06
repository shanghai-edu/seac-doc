# 2.3.7 属性释放配置

### 释放属性
修改 `attribute-filter.xml`, 将需要释放的属性加入其中
```xml
<?xml version="1.0" encoding="UTF-8"?>
<AttributeFilterPolicyGroup id="ShibbolethFilterPolicy"
         xmlns="urn:mace:shibboleth:2.0:afp"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd">

        <AttributeFilterPolicy id="Per-Attribute-singleValued">
            <PolicyRequirementRule xsi:type="ANY" />
            <AttributeRule attributeID="uid" permitAny="true" />
            <AttributeRule attributeID="cn" permitAny="true" />
            <AttributeRule attributeID="domainName" permitAny="true" />
            <AttributeRule attributeID="typeOf" permitAny="true" />
        </AttributeFilterPolicy>

</AttributeFilterPolicyGroup>
```
如果只对某个 sp 释放特定属性的话，参考如下示例
```xml
    <AttributeFilterPolicy id="example1">
        <PolicyRequirementRule xsi:type="Requester" value="https://sp.example.org" />
        <AttributeRule attributeID="uid" permitAny="true" />
    </AttributeFilterPolicy>
```

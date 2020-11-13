# 2.3.7 属性释放配置

### 释放属性
修改 `attribute-filter.xml`, 增加 `seaciAttrFilterToSPPolicy` 策略释放上海教育认证所需的属性，在 `carsiAttrFilterToSPPolicy` 中增加上海的2个测试sp地址。可参考如下示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
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
             <Rule xsi:type="Requester" value="https://resource.cloud.sh.edu.cn/shibboleth" />
         </PolicyRequirementRule>
            <AttributeRule attributeID="uid" permitAny="true" />
            <AttributeRule attributeID="cn" permitAny="true" />
            <AttributeRule attributeID="domainName" permitAny="true" />
            <AttributeRule attributeID="typeOf" permitAny="true" />
     </AttributeFilterPolicy>
</AttributeFilterPolicyGroup>
```

# 属性释放配置

### 释放属性
修改 `attribute-filter.xml`, 增加 `seaciAttrFilterToSPPolicy` 策略释放上海教育认证所需的属性，在 `carsiAttrFilterToSPPolicy` 中增加上海的2个测试sp地址。可参考如下示例

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
             <Rule xsi:type="Requester" value="https://ds.hainnu.edu.cn/shibboleth" />    
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
             <Rule xsi:type="Requester" value="https://ds.hainnu.edu.cn/shibboleth" />
         </PolicyRequirementRule>
            <AttributeRule attributeID="uid" permitAny="true" />
            <AttributeRule attributeID="cn" permitAny="true" />
            <AttributeRule attributeID="domainName" permitAny="true" />
            <AttributeRule attributeID="typeOf" permitAny="true" />
     </AttributeFilterPolicy>
</AttributeFilterPolicyGroup>
```



#### 自动同步属性策略

我们建议您采取自动同步的方式随时更新属性策略配置，即省去手动配置的麻烦，也能保障策略及时更新。

##### 备份现有的配置
```
cp /opt/shibboleth-idp/conf/attribute-filter.xml /opt/shibboleth-idp/conf/attribute-filter.xml.bak
```

##### 拉取仓库
```
[root@idp /]# cd /home/
[root@idp home]# git clone https://github.com/shanghai-edu/attribute-filter-policy.git
正克隆到 'attribute-filter-policy'...
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 10 (delta 1), reused 7 (delta 1), pack-reused 0
Unpacking objects: 100% (10/10), done.
[root@idp home]# 
```
##### 尝试手动执行脚本
```
[root@idp conf]# /home/attribute-filter-policy/update_policy.sh     
Already up-to-date.
Configuration reloaded for 'shibboleth.AttributeFilterService'
```
应该会看到 `Configuration reloaded for 'shibboleth.AttributeFilterService'` 的提示

##### 放入定时任务
```
15 3 * * * (/home/attribute-filter-policy/update_policy.sh) > /tmp/update_policy.log 2>&1
```

后续每天夜里应该都会自动同步策略了，也可以根据实际需要将定时任务的周期进一步放短或者放长。
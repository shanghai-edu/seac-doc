# 2.1.4 属性释放配置

# 属性过滤配置

Shibboleth 强调隐私的保护，因此对于 idP 所释放的属性，可以通过属性过滤器做对应的控制，限制其对不同的应用释放不同类别的属性。

#### attribute-filter 配置

修改IDP_HOME/conf/attribute-filter.xml
这里可以设定哪些属性能够释放给哪些SP。

可仿造其 example 配置

```
    <afp:AttributeFilterPolicy id="releaseTransientIdToAnyone">
        <afp:PolicyRequirementRule xsi:type="basic:ANY"/>
        <afp:AttributeRule attributeID="transientId">
            <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
        <afp:AttributeRule attributeID="uid">
                <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
        <afp:AttributeRule attributeID="cn">
                <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
        <afp:AttributeRule attributeID="email">
                <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
        <afp:AttributeRule attributeID="domainName">
                <afp:PermitValueRule xsi:type="basic:ANY"/>
        </afp:AttributeRule>
    </afp:AttributeFilterPolicy>
```



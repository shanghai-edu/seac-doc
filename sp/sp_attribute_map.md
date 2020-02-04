# 3.1.2 属性过滤器

修改属性过滤器配置文件，```/etc/attribute-map.xml```
增加 SEAC 属性规范规定的属性
```
  <Attribute name="urn:mace:dir:attribute-def:uid" id="uid" />
  <Attribute name="urn:mace:dir:attribute-def:mail" id="email" />
  <Attribute name="urn:mace:dir:attribute-def:cn" id="cn" />
  <Attribute name="urn:mace:dir:attribute-def:domainName" id="domainName" />
  <Attribute name="urn:mace:dir:attribute-def:typeOf" id="typeOf" />
  <Attribute name="urn:oid:2.5.4.2" id="uid" />
  <Attribute name="urn:oid:2.5.4.3" id="cn" />
  <Attribute name="urn:oid:2.5.4.5" id="domainName" />
  <Attribute name="urn:oid:2.5.4.100.2" id="typeOf" />
  <Attribute name="urn:oid:0.9.2342.19200300.100.1.3" id="mail" />
```
# 1.3.1 最小属性集

#### v2 最小属性集(草案)

|字段|SAML OID|说明|
|--|--|--|
|eduPersonPrincipalName|urn:oid:1.3.6.1.4.1.5923.1.1.1.6|用户名+域名后缀,等效于 v1 的 uid@domainName|	
|eduPersonNickname|urn:oid:1.3.6.1.4.1.5923.1.1.1.2|用户姓名,等效于 v1 的 cn|
|eduPersonPrimaryAffiliation|urn:oid:1.3.6.1.4.1.5923.1.1.1.5|机构域名,例如 xxx.edu.cn,等效于 v1 的 domainName|
|eduPersonScopeAffiliation|urn:oid:1.3.6.1.4.1.5923.1.1.1.9|用户身份,等效于 v1 的 typeOf@domainName|
|eduPersonUniqueId|urn:oid:1.3.6.1.4.1.5923.1.1.1.13|等效于 v1 的 eduID|


#### v1 最小属性集(历史兼容)

|字段|SAML OID|说明|
|--|--|--|
|uid|urn:oid:2.5.4.2|用户名|
|cn|urn:oid:2.5.4.3|用户姓名|
|domainName|urn:oid:2.5.4.5|机构域名,例如 xxx.edu.cn|
|typeOf|urn:oid:2.5.4.100.2|用户身份,例如 student,teacher|
|eduID|urn:oid:2.5.4.100.10|eduID|

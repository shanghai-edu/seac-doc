# 1.3.1 最小属性集

#### v2 最小属性集
##### SEAC 核心属性(草案)
|字段|OID|说明|来源|
|--|--|--|--|
|shEduPersonUserId|1.3.6.1.4.1.55229.1.1.1.1|用户在子域的标识，通常等于用户名，等效于 v1 的 uid||
|cn|2.5.4.3|用户姓名|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)，[RFC4519](https://tools.ietf.org/html/rfc4519)|
|shEduPersonHomeOrganization|1.3.6.1.4.1.55229.1.1.1.4|子域域名||
|eduPersonAffiliation|1.3.6.1.4.1.5923.1.1.1.1|身份|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)|	
|shEduId|1.3.6.1.4.1.55229.1.1.1.12|等效于 v1 的 eduID||

##### 联盟核心属性
如果期望 IdP 同时处于多个联盟，比如 SEAC 和 CARSI，那么需要至少释放其他联盟约定的核心属性
|字段|OID|说明|来源|
|--|--|--|--|
|eduPersonPrincipalName|1.3.6.1.4.1.5923.1.1.1.6|用户名+域名后缀,等效于 v1 的 uid@domainName|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)|	


#### v1 最小属性集(历史兼容)
下列属性仅用于兼容，将于未来废弃

|字段|SAML OID|说明|
|--|--|--|
|uid|2.5.4.2|用户名|
|cn|2.5.4.3|用户姓名|
|domainName|2.5.4.5|机构域名,例如 xxx.edu.cn|
|typeOf|2.5.4.100.2|用户身份,例如 student,teacher|
|eduID|2.5.4.100.10|eduID|

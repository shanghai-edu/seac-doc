# 1.3.4 完整属性集

### SEAC 属性列表（草案）

|字段|OID|说明|来源|
|--|--|--|--|
|shEduPersonUserId|1.3.6.1.4.1.55229.1.1.1.1|用户在子域的标识，通常等于用户名，等效于 V1 的 uid||
|shEduPersonDateOfBirth|1.3.6.1.4.1.55229.1.1.1.2|生日||
|shEduPersonGender|1.3.6.1.4.1.55229.1.1.1.3|性别||
|shEduPersonHomeOrganization|1.3.6.1.4.1.55229.1.1.1.4|子域域名||
|shEduPersonHomeOrganizationType|1.3.6.1.4.1.55229.1.1.1.5|子域类别||
|shEduPersonDepartment|1.3.6.1.4.1.55229.1.1.1.6|对于高校学生，院系。对于高校教职工，部门||
|shEduPersonMajor|1.3.6.1.4.1.55229.1.1.1.7|对于高校学生，专业||
|shEduPersonMatriculationDate|1.3.6.1.4.1.55229.1.1.1.8|入学日期，到年||
|shEduPersionStageOfStudy|1.3.6.1.4.1.55229.1.1.1.9|学段||
|shEduPersonGrade|1.3.6.1.4.1.55229.1.1.1.10|对普教，年级||
|shEduPersonClass|1.3.6.1.4.1.55229.1.1.1.11|对普教，班级||
|shEduPersonSchool|1.3.6.1.4.1.55229.1.1.1.12|对普教，学校||
|shEduId|1.3.6.1.4.1.55229.1.1.1.13|eduID||
|shEduPersonCourse|1.3.6.1.4.1.55229.1.1.1.14|对普教，任教学科||
|eduPersonScopedAffiliation|1.3.6.1.4.1.5923.1.1.1.9|用户身份+scope 后缀|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)|	
|eduPersonTargetedID|1.3.6.1.4.1.5923.1.1.1.10|hash 脱敏的用户唯一标识|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)|
|eduPersonEntitlement|1.3.6.1.4.1.5923.1.1.1.7|标识用户访问特定资源的权限的URI|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)|
|cn|2.5.4.3|用户姓名|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)，[RFC4519](https://tools.ietf.org/html/rfc4519)|
|eduPersonAffiliation|1.3.6.1.4.1.5923.1.1.1.1|身份|[eduPerson](https://wiki.refeds.org/display/STAN/eduPerson)|	
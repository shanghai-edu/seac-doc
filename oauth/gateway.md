# 3.3.2 授权网关模式

### 背景
#### 个性化展示
由于 OAuth2 模式本质上是针对 Shibboleth-SP 的一个代理。因此当 OAuth2 由上海教育认证中心提供时，Shibboleth 层面的 SP 也将均显示为 “上海教育认证中心”，并且 OAuth2 授权时，所提供的认证页面也必须是认证中心所提供的固定页面，无法进一步定制和优化。

#### 个性化授权
并非所以的应用都无差别对认证中心下的所有用户均提供服务。很多时候，应用都有特定的服务对象和人群，可能只针对部分 IdP，甚至只针对部分 IdP 上的部分用户提供服务。传统上需要应用方进行额外的授权控制处理，这无疑增加了应用的接入成本。

### 解决方案
我们的方案是，Shibboleth-OAuth2-Gateway（简称 SOG） 授权网关 + Shibboleth-OAuth2-Gateway-Manager（简称 SOGM）授权网关管理平台。

#### Shibboleth-OAuth2-Gateway（SOG 授权网关）
SOG 是一个封装好的 Shibboleth-SP + OAuth2 的代理服务。他既可以作为 Shibboleth 的 SP 接入 Shibboleth 认证联盟，也可以作为 OAuth2 Server 来提供 OAuth2 应用的接入。

在 OAuth2 的授权发起时，SOG 通过与 SOGM 通讯，获得该应用所服务的 IdP 列表，并在 DS 页面上，仅展示该部分 IdP 。在 OAuth2 授权过程中，用户完成 Shibboleth 认证以后，SOG 通过与 SOGM 通讯判定该用户是否有权限访问该应用。

SOG 可以由上海教育认证中心托管，提供云 SOG 服务，也可以独立部署自行维护，因此所以内容皆可自定义，满足个性化需求。
#### Shibboleth-OAuth2-Gateway-Manager（SOGM 授权网关管理平台）
默认情况下 SOG 本身不保存任何数据。所有所需的数据，诸如应用列表，用户列表等均由 SOGM 以接口方式提供。

SOG 与 SOGM 的通讯过程，全部基于 SOG 的 Shibboleth SP 秘钥进行加密签名，从而确保通讯的安全可靠，SOGM 通过联盟 Metadata 内的 SP 公钥进行签名验证，确保仅联盟内的 SP 可获取数据。

### 申请使用授权网关
请发送邮件至 its@cloud.sh.edu.cn 申请

### 调用示例

授权网关仅提供 OAuth2 授权码模式的授权，以下假定 SOG 的域名为
https://sog.example.org ，并以此为例提供调用示例。

本章的调用示例使用 [Google OAuth2.0 Playground](https://developers.google.com/oauthplayground/) 

#### 获取授权码
##### 请求方式
`GET /oauth/v1/authorize`
##### 请求参数
字段名|类型|是否必须|备注
--|--|--|--
scope|string|否|如果提供的话，必须是 SEAC-Basic
state|sring|否|如果提供的话，返回时会原样带回
redirect_uri|string|是|应用重定向的地址
response_type|string|是|必须是 code
client_id|string|是|应用的 client_id

##### 返回参数
字段名|类型|备注
--|--|--
code|string|返回的授权码，有效期 60 秒

##### 请求示例
Request
```
HTTP/1.1 302 Found
Location: https://sog.example.org/oauth/v1/authorize?scope=SEAC-Basic&redirect_uri=https%3A%2F%2Fdevelopers.google.com%2Foauthplayground&response_type=code&client_id=9ae61d477a4d9f61&access_type=offline
```
Response
```
GET /oauthplayground/?code=5acd6f86fa80a7db85c52183c53a44192cc4be6f HTTP/1.1
Host: developers.google.com
```
#### 获取 token
##### 请求方式
`POST /oauth/v1/token`
##### 请求参数
字段名|类型|是否必须|备注
--|--|--|--
code|string|是|获取到的授权码
redirect_uri|string|是|应用重定向的地址，必须和申请授权码时的地址完全一致
grant_type|string|是|必须是 authorization_code
client_id|string|是|应用的 client_id
client_secret|string|是|应用的 client_secret

##### 返回参数
字段名|类型|备注
--|--|--
access_token|string|返回的 access_token
token_type|string|token 的类型
expires_in|string|token 的有效期，单位是秒
refresh_token|string|返回的 refresh_token，SOG 默认不支持 refresh_token，可无视
scope|string|请求的 scope

##### 请求示例
Request
```
POST /oauth/v1/token HTTP/1.1
Host: sog.example.org
Content-length: 223
content-type: application/x-www-form-urlencoded
user-agent: google-oauth-playground
code=5acd6f86fa80a7db85c52183c53a44192cc4be6f&redirect_uri=https%3A%2F%2Fdevelopers.google.com%2Foauthplayground&client_id=9ae61d477a4d9f61&client_secret=b01c9fa2a6bba7424d60fd515902708d&scope=&grant_type=authorization_code
```
Response
```
HTTP/1.1 200 OK
Content-length: 179
X-powered-by: PHP/7.4.5
Server: nginx
Connection: keep-alive
Pragma: no-cache
Cache-control: no-store
Date: Wed, 27 May 2020 05:07:14 GMT
Content-type: application/json
{
  "access_token": "0331732743b5fa9d32db82bcb4af62798f5e3098", 
  "token_type": "Bearer", 
  "expires_in": 3600, 
  "refresh_token": "5bda2ff8db9f4b22b3ac77a55e0245aa7416ae70", 
  "scope": "SEAC-Basic"
}
```
#### 获取用户信息
##### 请求方式
`GET /oauth/v1/userinfo`
或
`POST /oauth/v1/userinfo`
##### 请求参数
字段名|类型|是否必须|备注
--|--|--|--
access_token|string|是|请求的 access_token

access_token 可以是 Bearer 方式，或者请求参数方式携带，都支持。

##### 返回参数
|字段|类型|说明|
|--|--|--|
|shEduPersonUserId|string|用户在子域的标识，通常等于用户名|
|shEduPersonDateOfBirth|string|生日|
|shEduPersonGender|string|性别|
|shEduPersonHomeOrganization|string|子域域名|
|hEduPersonHomeOrganizationName|string|子域名称|
|shEduPersonHomeOrganizationType|string|子域类别|
|shEduPersonDepartment|string|对于高校学生，院系。对于高校教职工，部门|
|shEduPersonMajor|string|对于高校学生，专业|
|shEduPersonMatriculationDate|string|入学日期，到年|
|shEduPersionStageOfStudy|string|学段|
|shEduPersonGrade|string|对普教，年级|
|shEduPersonClass|string|对普教，班级|
|shEduPersonSchool|string|对普教，学校|
|shEduId|string|eduID|
|eduPersonPrincipalName|string|用户名@域名，例如 200000@ecnu.edu.cn|
|eduPersonAffiliation|string|身份类别，取值为：faculty, student, staff, alum, member, affiliate, employee|
|eduPersonScopeAffiliation|string|eduPersonAffiliation+@域名，例如 faculty@ecnu.edu.cn|
|mail|string|邮箱|
|cn|string|姓名|
|mobile|string|手机号|


这里 eduPersonPrincipalName 是必然返回的字段，其他都可能为空

Request
```
GET /oauth/v1/userinfo HTTP/1.1
Host: sog.example.org
Content-length: 0
Authorization: Bearer 0331732743b5fa9d32db82bcb4af62798f5e3098
```
Response
```
HTTP/1.1 200 OK
Content-length: 510
Content-location: https://sog.example.org/oauth/v1/userinfo
X-powered-by: PHP/7.4.5
Server: nginx
Connection: keep-alive
Date: Wed, 27 May 2020 05:08:48 GMT
Content-type: text/html; charset=UTF-8
{
  "shEduPersonMatriculationDate": "", 
  "shEduPersonSchool": "", 
  "cn": "冯骐", 
  "shEduId": "", 
  "shEduPersonMajor": "", 
  "mobile": "", 
  "shEduPersonUserId": "", 
  "shEduPersonGrade": "", 
  "shEduPersonGender": "", 
  "shEduPersonHomeOrganizationType": "", 
  "eduPersonAffiliation": "faculty", 
  "eduPersonScopeAffiliation": "", 
  "shEduPersonHomeOrganization": "ecnu.edu.cn", 
  "mail": "", 
  "shEduPersonClass": "", 
  "eduPersonPrincipalName": "20150073@ecnu.edu.cn", 
  "shEduPersonHomeOrganizationName": "华东师范大学", 
  "shEduPersonDepartment": "", 
  "shEduPersonDateOfBirth": "", 
  "shEduPersionStageOfStudy": ""
}
```


#### 获取用户信息-兼容接口
该接口兼容认证中心模式的 oauth2 属性接口，可便于原认证中心模式下的应用快速迁移。
##### 请求方式
`GET /oauth/legacy/userinfo`
或
`POST /oauth/legacy/userinfo`
##### 请求参数
字段名|类型|是否必须|备注
--|--|--|--
access_token|string|是|请求的 access_token

access_token 可以是 Bearer 方式，或者请求参数方式携带，都支持。

##### 返回参数
|字段|类型|说明|
|--|--|--|
|ZYM|string|子域标识符，等同于 oauth/v1/userinfo 中的 shEduPersonHomeOrganization|
|ZYMC|string|子域名称，等同于 oauth/v1/userinfo 中的 shEduPersonHomeOrganizationName|
|RYLX|string|人员类型，等同于 oauth/v1/userinfo 中的 eduPersonAffiliation|
|UID|string|用户名，等同于 oauth/v1/userinfo 中的 shEduPersonUserId|
|XM|string|姓名，等同于 oauth/v1/userinfo 中的 cn|

Request
```
GET /oauth/legacy/userinfo HTTP/1.1
Host: sog.example.org
Content-length: 0
Authorization: Bearer 53164bdf78d7f7755b3cc65666c8aa20cfacd139
```
Response
```
HTTP/1.1 200 OK
Content-length: 139
Content-location: https://sog.example.org/oauth/legacy/userinfo
X-powered-by: PHP/7.4.5
Server: nginx
Connection: keep-alive
Date: Sun, 28 Jun 2020 02:08:26 GMT
Content-type: text/html; charset=UTF-8
{
  "type": 1, 
  "user": {
    "ZYM": "ecnu.edu.cn", 
    "ZYMC": "华东师范大学", 
    "RYLX": "faculty", 
    "UID": "20150073", 
    "XM": "冯骐"
  }
}
```
## Authorization Code模式

### 认证步骤概述

Authorization Code模式用于应用使用EAC进行用户认证的场景，其具体认证步骤如下：
1. 引导用户进入认证中心认证页面进行认证，用户通过认证后认证中心将给予用户授权code到回传地址；
2. 通过用户授权code换取access_token
3. 通过access_token获取用户基本信息

### 认证接口

#### 引导用户进入认证中心认证页面

##### 认证中心认证请求页面地址：
https://oauth.cloud.sh.edu.cn

##### 参数提交方式：GET

##### 参数说明

凡例：
###### 参数名: 约束条件
参数说明
>参数样例

###### client_id:	必有参数
上海教育认证中心给予的应用id，一般为八位随机码
> RscG5Arm

###### redirect_uri: 必有参数
认证完成后的回调地址，即认证完成后认证中西会将用户引导到这个地址，并且以GET形式给予code授权码
> http://www.abc.com/

###### state:	可选参数
回传参数，提交的值将在认证完成后返回code时原样返回

###### scope: 可选参数
请求的code的授权范围，EAC的scope由EAC管理员控制，建议不提交此参数或填写BASE
>BASE

###### grant_type: 可选参数
授权模式，默认值为authorization_code，建议不提交此参数或填写authorization_code
> authorization_code

```
完整的地址样例：
https://oauth.cloud.sh.edu.cn?client_id=RscG5Arm&redirect_uri=http://www.abc.com
或
https://oauth.cloud.sh.edu.cn?client_id=RscG5Arm&redirect_uri=http://www.abc.com&state=abc123&scope=BASE&grant_type=authorization_code
认证完成后认证中心将用户引导到回调地址样例：
http://www.abc.com?code=code5236602f5d174f268078bf680ea12770&state=abc123
```

#### 通过用户授权code换取access_token

##### 接口页面地址：
https://oauth.cloud.sh.edu.cn/oauth/GetAccessToken

##### 参数提交方式：GET

##### 参数说明

###### client_id:	必有参数
上海教育认证中心给予的应用id，一般为八位随机码
>	RscG5Arm

###### code: 必有参数
用户通过认证后认证中心给予的用户授权code，使用一次立即失效
> code5236602f5d174f268078bf680ea12770

###### client_secret: 必有参数
上海教育认证中心给予的应用secret
> 1STkFBC1Y5xEde2Y

###### scope: 可选参数
请求的code的授权范围，EAC的scope由EAC管理员控制，建议不提交此参数或填写BASE
>	BASE

###### grant_type:可选参数
授权模式，默认值为authorization_code，建议不提交此参数或填写authorization_code
> authorization_code


```
完整的地址样例：
https://oauth.cloud.sh.edu.cn/oauth/GetAccessToken?client_id=RscG5Arm&code=code5236602f5d174f268078bf680ea12770&client_secret=1STkFBC1Y5xEde2Y
或
https://oauth.cloud.sh.edu.cn/oauth/GetAccessToken?client_id=RscG5Arm&code=code5236602f5d174f268078bf680ea12770&client_secret=1STkFBC1Y5xEde2Y&scope=BASE&grant_type=authorization_code
```

##### 返回参数：
返回参数为json型字符串，定义如下

###### type:
操作结果码，1为成功，其余皆为失败
> 1

###### access_token:
可用于获取用户信息的令牌，获取token成功时有此参数  
> 808db567cd84bd49cc3ac5e3f2901da

###### expires_in:
token有效期，秒数，获取token成功时有此参数
>	7200

###### message:
错误信息，type不等于1时有此参数
> code失效


```
返回值样例：
{"type":0," message":"code失效"}
或
{"type":1,"access_token":"access808db567cd84bd49cc3ac5e3f2901da","expires_in":"7200"}
```

#### 通过access_token获取用户基本信息

##### 接口页面地址：
https://oauth.cloud.sh.edu.cn/oauth/GetUserInfo

##### 参数提交方式：GET

##### 参数说明

###### access_token: 必有参数
上一节获取到的access_token
>808db567cd84bd49cc3ac5e3f2901da

```
完整的地址样例：
https://oauth.cloud.sh.edu.cn/oauth/GetUserInfo?access_token=access808db567cd84bd49cc3ac5e3f2901da
```

##### 返回参数：
返回参数为json型字符串，定义如下

###### type:
操作结果码，1为成功，其余皆为失败
> 1

###### user:
登录用户的对象信息，属性如下：

>UID：用户ID，由子域方决定的唯一标识，大部分学校为学号、工号、卡号，基础教育子域为UUID


>XM：用户姓名


>RYLX：人员类型，即“学生”或“教师”


>ZYM：子域名，子域单位的长域名


>ZYMC：子域名城，子域单位的中文名称

###### message:
错误信息，type不等于1时有此参数
> code失效


```
返回值样例
{"type":0,"message":"提供的令牌已过期或已销毁"}
或
{"type":1,
"user":{"UID":"100012345","XM":"张三","RYLX":"教师" ,"ZYM":”shnu.edu.cn”,”ZYMC”:”上海师范大学” }}
```

### 补充说明

#### Scope

EAC OAuth服务的scope由EAC控制，应用方无需再请求时附加该参数，获取到的token访问范围按EAC后台设置的scope为准。

#### Refresh_token

为解决子域用户生命周期结束的问题，EAC OAuth服务不提供Refresh Token刷新令牌，即当 Access Token过期后，应用必须重新引导用户进行认证，才可获取新的Access Token，以此来保障当用户身份已在IDP中过期，不会发生OAuth中Token还能继续使用的问题。

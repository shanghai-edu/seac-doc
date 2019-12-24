## Client Credentials模式

### 使用步骤概述

Client Credentails模式用于应用使用SEAC提供的API，实现获取用户身份信息、注册用户、查询行为等功能，属于服务器间行为，与用户端无关，其具体使用步骤如下：

1. 向EAC申请申报请求的服务器地址，SEAC加入地址白名单后才会处理白名单内服务器发来的请求；
2. 服务器端使用httpClient向SEAC Oauth服务器请求token；
3. 使用token调用调用已经授权使用的API；

### 接口定义

#### 请求token

##### 认证中心认证请求地址：
https://oauth.cloud.sh.edu.cn/oauth/gettoken

##### 参数提交方式：GET

##### 参数说明

###### client_id:	必有参数
上海教育认证中心给予的应用id，一般为八位随机码
> PkN2WW5a

###### client_secret:	必有参数
上海教育认证中心给予的应用secret
> Jb2kF6aAAWmYx1G9


```
完整的请求地址样例：
https://oauth.cloud.sh.edu.cn/oauth/gettoken?client_id=PkN2WW5a&client_secret=Jb2kF6aAAWmYx1G9
```

##### 返回参数:

返回参数为json型字符串，定义如下:

###### type:
操作结果码，1为成功，其余皆为失败（错误码）
> 1

###### access_token:
成功时有此参数
> client05b630707e704559a6c7578ae2a663a7

###### expires_in:
成功时有此参数，token的剩余有效时间（秒）
> 576554

###### message:
失败时有此参数，错误信息，type不等于1时有此参数
> 请求地址未被授权


```
认证完成后认证中心将用户引导到回调地址样例：
{"type":1,"access_token":"client05b630707e704559a6c7578ae2a663a7","expires_in":"576554"}
或
{"type":10002,"message":"secret错误"}
或
{"type":10013,"message":"请求地址未被授权"}
或
{"type":10006,"message":"提供的令牌已过期或已销毁"}
```

#### 注册用户接口

##### 请求地址：
https://oauth.cloud.sh.edu.cn/api/regUser?token=client05b630707e704559a6c7578ae2a663a7

##### 参数提交方式：POST

数据以json格式，请使用utf-8编码

##### 参数说明
以json格式组织注册用户的数组，每个用户对象属性如下

###### userid:	必有参数
用户ID，将成为登录用户名 	
> zhangsan

###### name:	必有参数
用户姓名
> 张三

###### identity:	可选参数
身份，教师或学生
>	教师

###### mobile: 可选参数
手机号码
>13912341234

###### idpno:	必有参数
该用户所属于的子域编号，由SEAC发放，应用方请联系SEAC管理员获取 该参数
>2019050710540958176175a35fea559

###### eduid:	必有参数
上海教育服务号，需应用方有用户身份证信息生产，算法由SEAC提供
> 85ce01f9-7140-5d03-bdf7-a59ffae5ec63

###### usersecret:	可选参数
用户密码，若注册时不提供该参数，该用户需在微信服务号“上海教育认证中心”中激活账号并获取初始密码
>1qaJ3Er4


```
完整的请求数据样例：
[
    {
      "userid": "zhangsan",
      "name": "张三",
      "identity": "教师",
      "mobile": "13912341234",
      "idpno": "2019050710540958176175a35fea559",
      "eduid": "4788e071-bdc7-54a8-a187-0c6d6da9a237",
      "usersecret": "1q2w3e4r"
    },
    {
      "userid": "lisi",
      "name": "李四",
      "identity": "学生",
      "mobile": "13212341234",
      "idpno": "2019050710540958176175a35fea559",
      "eduid": "85ce01f9-7140-5d03-bdf7-a59ffae5ec63",
      "usersecret": "1q2w3e4r"
    }
]
```

##### 返回参数
返回参数为json型字符串，定义如下

###### result:
操作结果码，0为成功，其余皆为失败（错误码）
>	0

###### message
本次操作的信息
> 注册处理完成

###### users:
本次注册所有收到的注册用户信息的结果数组

Users数组中的每个对象的属性定义如下

> resCode
> 该用户注册时结果，0为成功，其余皆为错误号
>> 0

> resMessage
> 该用户注册操作的结果提示，错误时为错误提示
>> 注册成功。

> userid
>该注册用户的userid，即对应请求时对象中的userid
>> zhangsan

> eduid
> 该注册用户的eduid，即对应请求时对象中的eduid
>> 4788e071-bdc7-54a8-a187-0c6d6da9a237

> idpno
> 该注册用户的idpno，即对应请求时对象中的idpno
>> 201905121132308847180d01e71bac2

```
完整的请求返回样例：
{
    "result":0,
    "message":"注册处理完成。",
    "users":[
        {
            "resCode":40004,
            "resMessage":"参数idpno不正确。",
            "userid":"zhangsan",
            "eduid":"4788e071-bdc7-54a8-a187-0c6d6da9a237",
            "idpno":"201905121132308847180d01e71bac2"
        },
        {
            "resCode":40001,
            "resMessage":"账号已经注册，无法再次注册。",
            "userid":"lisi",
            "eduid":"85ce01f9-7140-5d03-bdf7-a59ffae5ec63",
            "idpno":"201905121132308847180d01e71bac1"
        },
        {
            "resCode":0,
            "resMessage":"注册成功。",
            "userid":"wangwu",
            "eduid":"85ce01f9-7140-5d03-bdf7-a59ffae5ec62",
            "idpno":"201905121132308847180d01e71bac1"
        }
    ]
}

```
### 补充说明

#### Token有效期
Client模式的token有效期为首次请求后的7天，过期后需重新申请。

#### Token请求次数限制
token请求限制为每日100次，请缓存或持久化存储该token，勿频繁请求token。

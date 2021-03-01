---
layout:     post
title:      "微信扫码登陆"
subtitle:   " \"微信扫码登陆预研\""
date:       2021-03-01 12:00:00
author:     "言良"
header-img: "img/post-bg-2015.jpg"
tags:
    - wechat
---

> “Yeah It's on. ”

## 微信扫码登录



### 注册阶段

1. 注册[微信开发平台](https://open.weixin.qq.com/)

2. 创建网站应用
   登录微信开发平台->管理中心->网站应用->创建网站应用，需要填写网站应用名称（名称中不能包含微信两个字），网站应用简介，应用官网（需要提供一个有效的网址），网站信息登记表（表中的网站应用名称需要和网页上填的保持一致，表中的开发者账号需要和当前注册网站应用的账号保持一致）

3. 开发者资质认证
   登录微信开发平台->账号中心->开发者资质认证



### 开发阶段

1. 请求code第三方使用网站应用授权登录前请注意已获取相应网页授权作用域（scope=snsapi_login），则可以通过在PC端打开以下链接： https://open.weixin.qq.com/connect/qrconnect?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect 若提示“该链接无法访问”，请检查参数是否填写错误，如redirect_uri的域名与审核时填写的授权域名不一致或scope不为snsapi_login。

   **参数说明**

   | 参数          | 是否必须 | 说明                                                         |
      | :------------ | :------- | :----------------------------------------------------------- |
   | appid         | 是       | 应用唯一标识                                                 |
   | redirect_uri  | 是       | 请使用urlEncode对链接进行处理                                |
   | response_type | 是       | 填code                                                       |
   | scope         | 是       | 应用授权作用域，拥有多个作用域用逗号（,）分隔，网页应用目前仅填写snsapi_login |
   | state         | 否       | 用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数加session进行校验 |

   **返回说明**

   用户允许授权后，将会重定向到redirect_uri的网址上，并且带上code和state参数

   ```text
   redirect_uri?code=CODE&state=STATE
   ```

   若用户禁止授权，则重定向后不会带上code参数，仅会带上state参数

   ```text
   redirect_uri?state=STATE
   ```

2. 通过code获取access_token通过code获取access_token

   ```text
   https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
   ```

   **参数说明**

   | 参数       | 是否必须 | 说明                                                    |
      | :--------- | :------- | :------------------------------------------------------ |
   | appid      | 是       | 应用唯一标识，在微信开放平台提交应用审核通过后获得      |
   | secret     | 是       | 应用密钥AppSecret，在微信开放平台提交应用审核通过后获得 |
   | code       | 是       | 填写第一步获取的code参数                                |
   | grant_type | 是       | 填authorization_code                                    |

   **返回说明**

   正确的返回：

   ```text
   { 
   "access_token":"ACCESS_TOKEN", 
   "expires_in":7200, 
   "refresh_token":"REFRESH_TOKEN",
   "openid":"OPENID", 
   "scope":"SCOPE",
   "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
   }
   ```

   **参数说明**

   | 参数          | 说明                                                         |
      | :------------ | :----------------------------------------------------------- |
   | access_token  | 接口调用凭证                                                 |
   | expires_in    | access_token接口调用凭证超时时间，单位（秒）                 |
   | refresh_token | 用户刷新access_token                                         |
   | openid        | 授权用户唯一标识                                             |
   | scope         | 用户授权的作用域，使用逗号（,）分隔                          |
   | unionid       | 当且仅当该网站应用已获得该用户的userinfo授权时，才会出现该字段。 |

   错误返回样例：

   ```text
   {"errcode":40029,"errmsg":"invalid code"}
   ```

   **刷新access_token有效期**

   access_token是调用授权关系接口的调用凭证，由于access_token有效期（目前为2个小时）较短，当access_token超时后，可以使用refresh_token进行刷新，access_token刷新结果有两种：

   ```text
   1. 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间；
   2. 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。
   ```

   refresh_token拥有较长的有效期（30天），当refresh_token失效的后，需要用户重新授权。

   **请求方法**

   获取第一步的code后，请求以下链接进行refresh_token：

   ```text
   https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
   ```

   **参数说明**

   | 参数          | 是否必须 | 说明                                          |
      | :------------ | :------- | :-------------------------------------------- |
   | appid         | 是       | 应用唯一标识                                  |
   | grant_type    | 是       | 填refresh_token                               |
   | refresh_token | 是       | 填写通过access_token获取到的refresh_token参数 |

   **返回说明**

   正确的返回：

   ```text
   { 
   "access_token":"ACCESS_TOKEN", 
   "expires_in":7200, 
   "refresh_token":"REFRESH_TOKEN", 
   "openid":"OPENID", 
   "scope":"SCOPE" 
   }
   ```

   **参数说明**

   | 参数          | 说明                                         |
      | :------------ | :------------------------------------------- |
   | access_token  | 接口调用凭证                                 |
   | expires_in    | access_token接口调用凭证超时时间，单位（秒） |
   | refresh_token | 用户刷新access_token                         |
   | openid        | 授权用户唯一标识                             |
   | scope         | 用户授权的作用域，使用逗号（,）分隔          |

   错误返回样例：

   ```text
   {"errcode":40030,"errmsg":"invalid refresh_token"}
   ```

   注意：

   ```text
   1、Appsecret 是应用接口使用密钥，泄漏后将可能导致应用数据泄漏、应用的用户数据泄漏等高风险后果；存储在客户端，极有可能被恶意窃取（如反编译获取Appsecret）；
   2、access_token 为用户授权第三方应用发起接口调用的凭证（相当于用户登录态），存储在客户端，可能出现恶意获取access_token 后导致的用户数据泄漏、用户微信相关接口功能被恶意发起等行为；
   3、refresh_token 为用户授权第三方应用的长效凭证，仅用于刷新access_token，但泄漏后相当于access_token 泄漏，风险同上。
   
   建议将secret、用户数据（如access_token）放在App云端服务器，由云端中转接口调用请求。
   ```

3. 通过access_token调用接口获取access_token后，进行接口调用，有以下前提：

   ```text
   1. access_token有效且未超时；
   2. 微信用户已授权给第三方应用帐号相应接口作用域（scope）。
   ```

   对于接口作用域（scope），能调用的接口有以下：

   | 授权作用域（scope） | 接口                      | 接口说明                                             |
      | :------------------ | :------------------------ | :--------------------------------------------------- |
   | snsapi_base         | /sns/oauth2/access_token  | 通过code换取access_token、refresh_token和已授权scope |
   | snsapi_base         | /sns/oauth2/refresh_token | 刷新或续期access_token使用                           |
   | snsapi_base         | /sns/auth                 | 检查access_token有效性                               |
   | snsapi_userinfo     | /sns/userinfo             | 获取用户个人信息                                     |

   其中snsapi_base属于基础接口，若应用已拥有其它scope权限，则默认拥有snsapi_base的权限。使用snsapi_base可以让移动端网页授权绕过跳转授权登录页请求用户授权的动作，直接跳转第三方网页带上授权临时票据（code），但会使得用户已授权作用域（scope）仅为snsapi_base，从而导致无法获取到需要用户授权才允许获得的数据和基础功能。 接口调用方法可查阅[《微信授权关系接口调用指南》](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Authorized_Interface_Calling_UnionID.html)

   F.A.Q

    1. 什么是授权临时票据（code）？ 答：第三方通过code进行获取access_token的时候需要用到，code的超时时间为10分钟，一个code只能成功换取一次access_token即失效。code的临时性和一次保障了微信授权登录的安全性。第三方可通过使用https和state参数，进一步加强自身授权登录的安全性。
    2. 什么是授权作用域（scope）？ 答：授权作用域（scope）代表用户授权给第三方的接口权限，第三方应用需要向微信开放平台申请使用相应scope的权限后，使用文档所述方式让用户进行授权，经过用户授权，获取到相应access_token后方可对接口进行调用。
    3. 网站内嵌二维码微信登录JS代码中style字段作用？ 答：第三方页面颜色风格可能为浅色调或者深色调，若第三方页面为浅色背景，style字段应提供"black"值（或者不提供，black为默认值），则对应的微信登录文字样式为黑色。相关效果如下：

   ![img](https://res.wx.qq.com/op_res/QWBllsRJkSzl2bX590fsHHL89vhVeAcck2qLZ8pvPYN03imZryim1Bil6dAfs_SK) ![img](https://res.wx.qq.com/op_res/13w7hjq0ZVNMNLcn91w9Cw0TZ7CtCsSwGW5mZ9GbuzzCzT0-nV5CCQ-BzJZDAtEU)

   若提供"white"值，则对应的文字描述将显示为白色，适合深色背景。相关效果如下：

   ![img](https://res.wx.qq.com/op_res/WAvW71FoNpu-NzpZf_bpCRiS2ZgO6vANtVp0d6ilxP-gkko9QkyOjoxjmMxC8Gll) ![img](https://res.wx.qq.com/op_res/iDyHm_ENEu4jVbXwVZoZKeEw36RA0dafqFGU-v-xTBCFc_sfb64wPxJlHPFDvdD9)

   4.网站内嵌二维码微信登录JS代码中href字段作用？ 答：如果第三方觉得微信团队提供的默认样式与自己的页面样式不匹配，可以自己提供样式文件来覆盖默认样式。举个例子，如第三方觉得默认二维码过大，可以提供相关css样式文件，并把链接地址填入href字段

   ```text
   .impowerBox .qrcode {width: 200px;}
   .impowerBox .title {display: none;}
   .impowerBox .info {width: 200px;}
   .status_icon {display: none}
   .impowerBox .status {text-align: center;} 
   ```

   相关效果如下：

   ![img](https://res.wx.qq.com/op_res/fW6MmqxGxJAzfluM9TDV58JifCanyliNg-y026kpPVVrkfEAACN6b021LbOZD3pa) ![img](https://res.wx.qq.com/op_res/LebglCIrjHSPDMK7mGrkR1ihb1o4jyDA5vSOB7X_3NPwBq5f5bXkocVhPkQHJVpD)

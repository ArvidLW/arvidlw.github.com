# angular+springboot

前后端分离

统一日志平台代码

## jwt与RSA认证访问 

jwt认证方式

后端：rsa认证加入到spring boot的rsa认证过程中，具有两种认证方法

前端：jwt认证，jwtinterceptor, 自动对请求进行拦截检查，所以对需要加入了rsa认证请求链接，要在jwt 的ignores中。

1、jwt认证
2、rsa认证，前台仅接收view-log/tokencode?appId=xx&appName=xx的页面, 不进行jwt验证(验证token的格式与是否过期,@delon\auth\fesm5\auth.js); 
后台可以接收jwt,rsa认证访问所有api。
其中，tokencode是指的由ticket + urlbase64(rsa公钥加密json字符串后的内容)
其中，json字符串为 "{\"user\": \"xxx\", \"password\":\"xxx\"}"
其中，appId=xx&appName=xx,表示应用id与应用名(只用于显示,也可以不传入，默认名为current app)

```
http://localhost:4205/#/view-log/ticketWIm3B-NRN-vT-5V2gIrQHt5nsqGYX02z5aveSuMigV5X1g2FCUEzrTD0S5koK-Q2ynmb6bwyePgtzoC6JJwpvW2eM8p8HpSffbiIXYY1VgI7T4_VQBshq7nUlD3nT5iBC1IvVeKXp0teNqdeh8JQIUQC_CxaIO7QgSdCFV4Zi9I?appId=321&appName=cmblog_BDP1_LT18_DEV
```

3、直接请求后台例子

```
POST /api/logScan/viewLogContentList HTTP/1.1
Host: localhost:9005
Authorization: ticketWIm3B-NRN-vT-5V2gIrQHt5nsqGYX02z5aveSuMigV5X1g2FCUEzrTD0S5koK-Q2ynmb6bwyePgtzoC6JJwpvW2eM8p8HpSffbiIXYY1VgI7T4_VQBshq7nUlD3nT5iBC1IvVeKXp0teNqdeh8JQIUQC_CxaIO7QgSdCFV4Zi9I
Content-Type: application/json

{
	"appId": "321",
	"endTime": 1542954106999,
	"startTime": 1542953206000,
	"pageSize": 10,
	"pageNo": 0
}
```

4、过程

[思维导图-认证相关]: http://naotu.baidu.com/file/4446a9a4d613dd618cd50ed75e130e5a?token=9f7fd6dab03363da

后台接收两种认证方式，但前台一个页面只支持一种认证。

由于当jwt忽略检查页面token时，也不会在授权后，为访问后台的请求自动加上token。

故前台页面一个页面是有一种认证方式。

token,是前台登录请求后台得到的一个全局的变量。

```

jwt页面:浏览器--> 访问前台 --没有token--> 先登录 --登录成功返回token保存acl--> 访问页面 --> 检查jwttoken格式（3部分，类型、内容、签名） --> 加上token认证 --proxy--> 后台

不检查jwt页面:浏览器--> 访问前台 --是否在忽略检查token列表中--> 是 ----> 访问页面 -- 加上rsa认证 --> proxy --> 后台
```
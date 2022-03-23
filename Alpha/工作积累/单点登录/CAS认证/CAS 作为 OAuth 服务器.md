# OAuth/OpenID 身份验证

允许 CAS 充当 OAuth/OpenID 身份验证提供程序。请[查看规范](https://oauth.net/2/)以了解更多信息。

**CAS 作为 OAuth 服务器**

本页具体描述了如何启用对 CAS 的 OAuth/OpenID 服务器支持。如果您想让 CAS 充当 OAuth/OpenID 客户端与其他提供商（例如 Google、Facebook 等）通信，[请参阅此页面](https://apereo.github.io/cas/6.3.x/integration/Delegate-Authentication.html)。





```
{
    "@class":"org.apereo.cas.support.oauth.services.OAuthRegisteredService",
    "clientId":"1",
    "clientSecret":"123456",
    "serviceId":"^(https|http):.*",
    "name":"oauth-10000004",
    "id":10000004,
    "supportedGrantTypes":[
        "java.util.HashSet",
        [
            "authorization_code"
        ]
    ],
    "supportedResponseTypes":[
        "java.util.HashSet",
        [
            "code"
        ]
    ],
    "generateRefreshToken":false,
    "attributeReleasePolicy":{
        "@class":"org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
    }
}
```



获取code

```
https://172.16.174.81/cas/oauth2.0/authorize?response_type=code&client_id=1&redirect_uri=http://www.yoodb.com
```

获取token 

```
http://172.16.174.81/cas/oauth2.0/accessToken?grant_type=authorization_code&client_id=1&client_secret=123456&redirect_uri=http://www.yoodb.com&code=OC-2-
vHAjVqsz5QAN4WruSi2g11tYnpNdnMD
```


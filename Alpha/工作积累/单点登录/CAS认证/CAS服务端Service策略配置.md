

# CAS单点登录(五)——Service配置及管理

https://blog.csdn.net/Anumbrella/article/details/82119246



## 持久化策略

https://blog.csdn.net/u010475041/article/details/78018100



### Service配置

CAS客户端接入称之为【**service**】，必须经过CAS的允许才能进行登录，当然不同的客户端可以做不同的事情，其中包括：

1. 自定义主题（各客户端登录页自定义）
2. 自定义属性（服务属性（固定）与用户属性（动态））
3. 自定义协议
4. 自定义登录后跳转方式，跳转路径
5. 授权策略（拒绝属性、可登录时间范围限制、等等）
6. 拒绝授权模式

> service是使用型cas是服务型，cas好比游乐园，service好比来游乐园的游客；游客需要进入游乐园，那么游客需要门票，获取门票有多种方式，可以用手机校验码或者身份证进行获取。具体service作为客户端使用好比买门票，必须填写身份证号、手机号、付款等流程，当然也可以通过不同渠道购买，cas也有不同的客户端实现，cas client、pac4j等 。被接入的service无需进行输入密码即可进入系统，好比A-service（OA系统）登录了，B-service（账单系统），C-service（CRM系统）无需再次登录。

#### 持久化策略

- InMemory XML(通过spring bean进行内存存储)
- JSON(通过json文件存储) **`推荐`**
- YAML(通过yml文件存储)
- Mongo(文档数据库持久化)**`推荐`** 
- JPA(关系型数据库持久化)
- DynameDb
- LDAP
- Cochbase

sso初步上线时推荐采用json文件存储，后面逐步多服务注入时推荐采用Mongo进行存储，采用cas-management进行采用UI进行管理我们的数据，目前阶段，持久化策略必须和cas进行配置一致才能生效。

#### service允许客户端配置

对所有`http://localhost`开头请求的service进行允许认证，在`resources/services`下新建文件`HTTPLocalhost-10000000.json`。

```json
{
  "@class": "org.jasig.cas.services.RegexRegisteredService",
  "serviceId": "^(http)://localhost.*",
  "name": "本地服务",
  "id": 10000000,
  "description": "这是一个本地允许的http服务，通过localhost访问都允许通过",
  "evaluationOrder": 1
}
```

> json文件名字规则为${name}-${id}.json，id必须为json文件内容id一致

| 参数            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| @class          | 必须为org.apereo.cas.services.RegisteredService的实现类，对其他属性进行一个json反射对象，常用的有RegexRegisteredService，匹配策略为id的正则表达式 |
| serviceId       | 唯一的服务id                                                 |
| name            | 服务名称，会显示在默认登录页                                 |
| id              | 全局唯一标志                                                 |
| description     | 服务描述，会显示在默认登录页                                 |
| evaluationOrder | 匹配争取时的执行循序                                         |
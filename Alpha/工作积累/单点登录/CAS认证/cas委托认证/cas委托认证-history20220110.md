# CAS委托认证

## 一、概述

CAS 服务器可以使用[pac4j 安全引擎](https://github.com/pac4j/pac4j)充当客户端，并将身份验证委托给：

- 另一个CAS服务器
- OAuth提供商: Facebook, Twitter, Google, LinkedIn, Yahoo and several other providers
- OpenID 提供商: myopenid.com
- SAML identity 提供商
- An OpenID Connect identity provider

> CAS委托认证就是将CAS对接到第三方服务进行认证，需要第三方认证流程符合规范的流程，CAS已经提供现成的github、Facebook、QQ等对接。

## 三、委托认证

CAS委托认证时序图

![image-20220110225825370](images\image-20220110225825370.png)



## 二、接入说明

### 1、引入依赖

> Maven

```xml
<dependency>
    <groupId>org.jasig.cas</groupId>
    <artifactId>cas-server-support-pac4j-webflow</artifactId>
    <version>${cas.version}</version>
</dependency>
```

> Gradle

```json
implementation "org.apereo.cas:cas-server-support-pac4j-webflow:${project.'cas.version'}"
```

### 2、配置

添加所需的客户端，身份提供商是一种服务器，可以验证用户（如谷歌，雅虎...），而代替CAS服务器。例如， 如果你想将CAS身份验证委托给 Twitter， 你必须为提供商添加一个 Oauth 客户端： Twitter 。对于每个委托身份验证机制，您必须定义相应的客户端。

> 对于标准的OAuth服务的接入，客户端可以在属性文件中`cas.properties`定义。

下面以接入吉大的OAuth服务为例，在CAS上配置对接的OAuth的客户端信息，在spring配置文件中定义如下：WEB-INF/spring-configuration/pac4jContext.xml

```xml
<bean id="uiam" class="org.uiam.oauth.client.UIAMOAuthWrapperClient">
        <property name="key" value="this_is_the_key" />
        <property name="secret" value="this_is_the_secret" />
        <property name="uiamOAuthUrl" value="http://localhost:8081/uias" />
</bean>

<bean id="client1" class="org.pac4j.oauth.client.CasOAuthWrapperClient">
  <property name="key" value="this_is_the_key" />
  <property name="secret" value="this_is_the_secret" />
  <property name="casOAuthUrl" value="http://mycasserver2/oauth2.0" />
</bean>

<bean id="client2" class="org.pac4j.cas.client.CasClient">
  <property name="casLoginUrl" value="http://mycasserver2/login" />
</bean>
```

请注意，对于每个OAuth提供商，CAS 服务器被视为OAuth客户端，因此应在OAuth提供商处宣布为OAuth客户端。申报后，OAuth提供者会提供钥匙（key）和秘密（secret），该密钥key和秘密secret必须beans中property属性进行定义。

### 3、在登录页面上添加链接以对远程提供商进行身份验证

所有可用的客户端将自动显示在登录页面上，作为"或使用"标签下的可点击按钮。如果您自定义登录页面，则可以访问显示的文本（主要是客户端的名称）和用于重定向对象中的身份提供程序的网址（这是一张到网址的名称地图）。`pac4jUrls`

### 4、已验证用户的标识符

在成功委托身份验证后，在 CAS 服务器内创建了具有特定标识符的用户：此标识符只能从外部身份提供商（如 1234）收到的技术标识符中创建，也可以创建为"键入标识符"（如 FacebookProfile#1234），即默认标识符。

这可以在cas配置文件`cas.properties`中定义是否取消前置。

```json
cas.pac4j.client.authn.typedidused=true
```

如果取消则返回用户唯一标识不会附带上#以及#前置的内容。

### 5、委托认证 demo下载演示

demo地址： https://github.com/leleuj/cas-pac4j-oauth-demo

### 6、如何在 CAS 应用程序方面使用此支持？

授权身份验证返回的信息

一旦您已配置（见上文信息），您的 CAS 服务器将充当非真实、CAS、OpenID（连接）或 SAML 客户端，用户将能够在 OAuth/CAS/OpenID/SAML 提供商（如 Facebook）进行身份验证，而不是直接在 CAS 服务器内进行身份验证。在 CAS 服务器中，此类委托身份验证后，用户具有特定的身份验证数据。

对象有：`Authentication`

- 属性 （身份验证） 设置为*`组织. jasig. cas. 支持. pac4j. 身份验证. 处理器. 支持. 客户验证汉德勒`*`AuthenticationManager.AUTHENTICATION_METHOD_ATTRIBUTE`
- 属性*`客户名`*设置为身份验证过程中使用的提供商类型。
- 对象的对象有：`Principal``Authentication`
  - 此提供商的用户标识符++用户标识符（即`#``FacebookProfile#0000000001`)
  - 由从提供商检索的数据填充的属性（姓名、姓氏、出生日...）

### 7、如何向 CAS 客户端应用程序发送配置文件属性？

在 CAS 应用程序中，通过服务票证验证，将用户信息推至 CAS 客户端，从而推至应用程序本身。用户标识符始终被推入 CAS 客户端。对于用户属性，它既涉及服务器上的配置，也涉及验证服务票证的方式。在 CAS 服务器端，要向 CAS 客户端推送属性，应将其配置在预期服务中：

```properties
{
  "@class" : "org.jasig.cas.services.RegexRegisteredService",
  "serviceId" : "sample",
  "name" : "sample",
  "id" : 100,
  "description" : "sample",
  "attributeReleasePolicy" : {
    "@class" : "org.jasig.cas.services.ReturnAllowedAttributeReleasePolicy",
    "allowedAttributes" : [ "java.util.ArrayList", [ "name", "first_name", "middle_name" ] ]
  }
}
```

在 CAS 客户端，要接收属性，您需要使用 SAML 验证或 CAS 3.0 验证，即 url。`/p3/serviceValidate`

### 8、如何在 CAS 应用程序中重新创建用户配置文件？

在 CAS 服务器中，完整的用户配置文件是已知的，但当属性被发送回 CAS 客户端应用程序时，会出现某种"CAS 序列化"，使数据在原始状态下无法恢复。不过，您现在可以从 CAS 返回的数据中完全重建原始用户配置文件。`Assertion`验证服务票证后，CAS 客户端端中提供了一个，您可以从该客户端获得使用 pac4j 库的经过验证的用户的标识符和属性：`Assertion`

```java
final AttributePrincipal principal = assertion.getPrincipal();
final String id = principal.getName();
final Map<String, Object> attributes = principal.getAttributes();
```

由于标识符在其定义中存储配置文件类型（），您可以使用该方法重新创建原始配置文件：`*clientName#idAtProvider*``org.pac4j.core.profile.ProfileHelper.buildProfile(id, attributes)`

```java
final FacebookProfile rebuiltProfileOnCasClientSide =
    (FacebookProfile) ProfileHelper.buildProfile(id, attributes);
```

## 四、接入示例

### 1、委托流程展示

![image-20210324142531624](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210324142531624.png)



![image-20210324143015063](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210324143015063.png)



![image-20210324143115466](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210324143115466.png)

#### 请求过程

```
step 1:
http://localhost:8080/cas/login?client_name=scdx&service=http://localhost:8080/shiro-cas.do?system=4a
step 2:
http://eop.paas.sc.ctc.com/serviceAgent/v2/uam/oauth/token?client_id=key&client_secret=sercret&redirect_uri=http://localhost:8080/cas/login?client_name=scdx&service=http://localhost:8080/shiro-cas.do?system=4a
step 3:
http://localhost:8080/shiro-cas.do?system=4a&ticket=ST-4-EdiaJCmUXtJ1egVMB7yG-tianrong-cas&locale=zh_CN
step 4:
http://localhost:8080/cas/serviceValidate?ticket=ST-4-AhqAvl9uHywhCpNPcDwN-tianrong-cas&service=http%3A%2F%2Flocalhost%3A8080%2Fshiro-cas.do%3Fsystem%3D4a
step 5:
http://localhost:8080/gotoWebPortal.do
```



### 2、代码接入示例

1、在cas-server-webapp中的pom.xml中加入以下dependency用于支持oauth

```xml
<dependency>  
    <groupId>org.pac4j</groupId>  
    <artifactId>pac4j-oauth</artifactId>  
    <version>${pac4j.version}</version>  
</dependency>
```

2、在cas-server-support-pac4j项目的pom.xml增加必需的pac4j-\* libraries

```xml
<dependency>  
    <groupId>org.jasig.cas</groupId>  
    <artifactId>cas-server-support-pac4j</artifactId>  
    <version>${cas.version}</version>  
</dependency>  
```

3、修改login-webflow.xml

增加oauth用户验证逻辑，把处理oauth的client action添加到webflow中在login-webflow.xml中,这clientAction添加在webflow的最前面.它的任务是oauth用户验证的callback的调用.

```xml
<action-state id="clientAction">  
    <evaluate expression="clientAction" />  
    <transition on="success" to="sendTicketGrantingTicket" />  
    <transition on="error" to="ticketGrantingTicketCheck" />  
    <transition on="stop" to="stopWebflow" />  
</action-state>  
<view-state id="stopWebflow" />  
```

4、pac4jContext.xml中配置委托认证客户端

```xml
<bean id="uiamwrapper1" class="org.uiam.oauth.client.UIAMOAuthWrapperClient">
    <property name="key" value="this_is_the_key" />
    <property name="secret" value="this_is_the_secret" />
    <property name="uiamOAuthUrl" value="http://localhost:8081/uias" />
</bean>
```

5、重写OAuth2委托认证代码逻辑

初始化client、通过构造函数初始化各种客户端

6、ClientAuthenticationHandler处理

```java
protected HandlerResult doAuthentication(final Credential credential) throws GeneralSecurityException, PreventedException {
    final ClientCredential clientCredentials = (ClientCredential) credential;
    logger.debug("clientCredentials  {}", clientCredentials);

    final Credentials credentials = clientCredentials.getCredentials();
    final String clientName = credentials.getClientName();
    logger.debug("clientName:  {}", clientName);

    // get client
    final Client<Credentials, UserProfile> client = this.clients.findClient(clientName);
    logger.debug("client: {}", client);

    // web context
    final HttpServletRequest request = WebUtils.getHttpServletRequest();
    final HttpServletResponse response = WebUtils.getHttpServletResponse();
    final WebContext webContext = new J2EContext(request, response);

    // get user profile
    final UserProfile userProfile = client.getUserProfile(credentials, webContext);
    logger.debug("userProfile: {}", userProfile);

    return createResult(clientCredentials, userProfile);
}
```



![image-20210329172038718](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210329172038718.png)

完成登录。

### 3、登录认证过程

1、浏览器访问4A地址，在用户没有登录的情况下，会被shiroFilter拦截，指向用户的登录链接：/login_cas_host_mapping_login.do；然后在login_cas_host_mapping_login请求映射类中重定向到cas的登录地址:

```
http://localhost:8080/cas/login?service=http://localhost:8080/shiro-cas.do?system=4a
```

shiroFilter具体配置在application-context-shiro-cas-host-mapping.xml中

2、访问三方认证（OAuth2）地址，拿code

```
http://localhost:8081/uias/uias/bypass.do?response_type=code&client_id=key&redirect_uri=http://localhost:8080/cas/login?client_name=xjga&service=http://localhost:8080/shiro-cas.do?system=4a
```

3、三方认证完成返回Code，回调cas

```
http://localhost:8080/cas/login?client_name=xjga&service=http://localhost:8080/shiro-cas.do?system=4a
```

4、进入cas的ClientAction

​	a.获取clientname，根据clientname拿到Client客户端对象

​	b.根据客户端对象拿到credentials（证明），ClientAuthenticationHandler的doAuthentication方法进行处理

​	c.进入客户端对象的getAccessToken方法，请求OAuth服务器拿token

​	d.cas端根据token请求OAuth服务器拿用户信息，解析返回的用户信息，转化为cas认证的用户信息

​	e.ClientAuthenticationHandler的doAuthentication返回HandlerResult

```java
createResult(clientCredentials, userProfile)
clientCredentials:客户端名称和code
userProfile:认证用户的信息
```

```
credentials.setUserProfile(profile);
credentials.setTypedIdUsed(this.typedIdUsed);
```

​	f.生成tgt对象

![image-20210330173214568](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210330173214568.png)

5、CAS服务器委托认证已完成

CAS服务器中，经过这种委托身份验证后，用户具有特定的身份验证数据。

该`Authentication`对象具有：

- 该属性`AuthenticationManager.AUTHENTICATION_METHOD_ATTRIBUTE`（authenticationMethod）设置为*`org.jasig.cas.support.pac4j.authentication.handler.support.ClientAuthenticationHandler`*
- 该属性*`clientName`*设置为身份验证过程中使用的提供程序的类型。

该`Principal`对象的`Authentication`对象具有：

- 标识符是个人档案类型+`#`该提供者的用户标识符（即`FacebookProfile#0000000001`）
- 由提供者检索的数据填充的属性（名字，姓氏，生日…）

cas根据请求的service，定向到CAS的应用（4A）

```
http://localhost:8080/shiro-cas.do?system=4a&ticket=ST-4-EdiaJCmUXtJ1egVMB7yG-tianrong-cas&locale=zh_CN
```

cas处理完成之后回调4A

PersonDirectoryPrincipalResolver

```java
final Map<String, List<Object>> attributes = retrievePersonAttributes(principalId, credential);
```

retrievePersonAttributes方法：

```java
protected Map<String, List<Object>> retrievePersonAttributes(final String principalId, final Credential credential) {
    final IPersonAttributes personAttributes = this.attributeRepository.getPerson(principalId);
    final Map<String, List<Object>> attributes;

    if (personAttributes == null) {
        attributes = null;
    } else {
        attributes = personAttributes.getAttributes();
    }
    return attributes;
}
```

扩展CAS中获取用户属性的bean，在CAS客户端验证ST时，返回自定义属性

```java
ExtendPersonAttributeDao
```

getPerson方法：

![image-20210401141811520](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210401141811520.png)

```java
final DefaultAuthenticationContext ctx = new DefaultAuthenticationContext(authentication, service);
ctx.setCredentialProvided(this.providedCredential != null);
```

![image-20210401142809591](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210401142809591.png)

6、4A侧shiro-cas.do请求拦截ShiroCasHostMappingRealm类doGetAuthenticationInfo

​	a.拿到token和ticket

​	b.CAS客户端上，要接收属性，您需要使用SAML验证或CAS 3.0验证，即`/p3/serviceValidate`

7、ST在cas认证完成之后，提取用户的唯一属性

在CAS服务器中，完整的用户配置文件是已知的，但是当将属性发送回CAS客户端应用程序时，会出现某种“ CAS序列化”，这使得不容易将数据还原到其原始状态。

不过，您现在可以根据CAS中返回的数据完全重建原始用户配置文件`Assertion`。

验证服务票证之后，`Assertion`CAS客户端中提供了一个，您可以使用pac4j库从中获取经过身份验证的用户的标识符和属性：

```java
final AttributePrincipal principal = assertion.getPrincipal();
final String id = principal.getName();
final Map<String, Object> attributes = principal.getAttributes();
```

由于标识符将配置文件的类型存储在其自己的定义（`*clientName#idAtProvider*`）中，因此您可以使用该`org.pac4j.core.profile.ProfileHelper.buildProfile(id, attributes)`方法来重新创建原始配置文件：

```java
final FacebookProfile rebuiltProfileOnCasClientSide =
    (FacebookProfile) ProfileHelper.buildProfile(id, attributes);
```

8、查询用户是否存在，根据用户名称在4A做登录，初始化权限等逻辑。

```
doAuthentication认证结束之后CAS进行了什么操作？
```

​	1、返回失败：INVALID_TICKET

​	ticket验证具有时效性，debug模式下尽量尽快进行通过验证，否则可能返回INVALID_TICKET。或者"renew"属性设置为true，而不是主从认证。

​	2、返回成功：

​	shiro-cas.do验证ST准确性，回调CAS

CasHostMappingCasFilter

```
CasHostMappingToken casHostMappingToken = new CasHostMappingToken(ticket);
casHostMappingToken.setHttpRequest(httpRequest);
```

ShiroCasHostMappingRealm

```
CasHostMappingToken casToken = (CasHostMappingToken) token;
```

AbstractUrlBasedTicketValidator

```java
String serverResponse = this.retrieveResponseFromServer(new URL(validationUrl), ticket);
if (serverResponse == null) {
    throw new TicketValidationException("The CAS server returned no response.");
} else {
    this.logger.debug("Server response: {}", serverResponse);
    return this.parseResponseFromServer(serverResponse);
}
```

校验结果： /p3/serviceValidate

```xml
<cas:serviceResponse xmlns:cas='http://www.yale.edu/tp/cas'>
    <cas:authenticationSuccess>
        <cas:user>UIAMOAuthWrapperProfile#510184199711902909</cas:user>            
            <cas:attributes>
                    <cas:access_token>001</cas:access_token>               
            </cas:attributes>     
            <cas:attributes>                       <cas:longTermAuthenticationRequestTokenUsed>false</cas:longTermAuthenticationRequestTokenUsed>
                        <cas:isFromNewLogin>true</cas:isFromNewLogin>
                        <cas:authenticationDate>2021-03-30T15:50:27.546+08:00</cas:authenticationDate>
            </cas:attributes>
    </cas:authenticationSuccess>
</cas:serviceResponse>
```

校验结果： /serviceValidate

```xml
<cas:serviceResponse xmlns:cas='http://www.yale.edu/tp/cas'>
    <cas:authenticationSuccess>
        <cas:user>UIAMOAuthWrapperProfile#510184199711902909</cas:user>
            <cas:attributes>               
                    <cas:access_token>001</cas:access_token>               
            </cas:attributes>
    </cas:authenticationSuccess>
</cas:serviceResponse>
```

#### needs_client_redirection

```
localhost:8080/scdx/callback/getToken.do?code=SADASIHASUDHASUHDUASH   =>>  http://localhost:8080/cas/login?client_name=scdx&service=http://localhost:8080/shiro-cas.do?system=4a
needs_client_redirection=
http://localhost:8080/cas/login?client_name=xjga?service=http://localhost:8080/shiro-cas.do?system=4a
```

![image-20210402095746855](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210402095746855.png)



![image-20210402113648547](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210402113648547.png)



![image-20210402125100315](C:\Users\pitt1\AppData\Roaming\Typora\typora-user-images\image-20210402125100315.png)







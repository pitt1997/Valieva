# CAS 4.2.x 版本代理认证实现

# 一、CAS代理认证

**请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-Proxy-Authentication.html)了解更多详情。**

默认情况下启用对 CAS v1+ 协议的代理身份验证支持，因此利用代理身份验证功能完全是 CAS 客户端配置的问题。

# 二、什么是CAS的代理认证

举例这样一个场景：有两个服务，分别是【运维服务】和【资源管理服务】，这两个服务都集成了CAS，所有的请求都要经过CAS Server【认证中心】的认证。由于【运维服务】内部会去调用【资源管理服务】，但是【运维服务】的请求会被【资源管理服务】配置的CAS拦截器【AuthenticationFilter】拦截并重定向到CAS Server【认证中心】进行引导用户进行登录认证，这样我们的【运维服务】就没法直接访问【资源管理服务】，针对这种应用场景，CAS提供了【代理认证 】模式解决这个问题。

# 三、CAS代理模式原理

CAS代理模式的主要原理如下：**运维服务(代理端)请求资源管理服务(被代理端)**，运维服务首先通过CAS Server的认证，然后向CAS Server申请一个针对资源管理服务的【**proxy ticket**】，之后在访问资源管理服务的请求中将【**proxy ticket**】以参数ticket的形式传递过去，资源管理服务的AuthenticationFilte拦截到该请求，但是发现这个请求携带了ticket参数，于是交给后续的**Ticket Validation Filter**处理，**Ticket Validation Filter**将会传递该ticket到CAS Server进行认证，由于该ticket是由CAS Server针对于资源管理服务发行的，资源管理服务申请校验的时候自然验证成功，这样运维服务就可以访问到资源管理服务。下面CAS官网描述的CAS Proxy的UML图：

![image-20210802215303378](.\images\image-20210802215303378.png)

# 四、CAS代理模式相关配置说明

CAS Proxy代理模式实现的核心过滤器是**Cas20ProxyReceivingTicketValidationFilter**，对于代理端，这个过滤器要配置在**AuthenticationFilter**之前，并且**Cas20ProxyReceivingTicketValidationFilter在代理端和被代理端的配置是不一样的**，这个会在项目实战中具体指出。在我们的项目中我们使用的是**Cas30ProxyReceivingTicketValidationFilter**，Cas30ProxyReceivingTicketValidationFilter是Cas20ProxyReceivingTicketValidationFilter的子类。

CAS是基于HTTP2和HTTP3协议的，任何一个组件都可以通过特定的URL访问。**Cas30ProxyReceivingTicketValidationFilter**使用/p3/开头请求。

| **URI**             | **描述**                                  |
| ------------------- | ----------------------------------------- |
| /login              | 登录                                      |
| /logout             | 销毁CAS会话（注销）                       |
| /validate           | service ticket validation                 |
| /serviceValidate    | service ticket validation [CAS 2.0]       |
| /proxyValidate      | service/proxy ticket validation [CAS 2.0] |
| /proxy              | proxy ticket service [CAS 2.0]            |
| /p3/serviceValidate | service ticket validation [CAS 3.0]       |
| /p3/proxyValidate   | service/proxy ticket validation [CAS 3.0] |

**Cas20ProxyReceivingTicketValidationFilter**相关参数说明：

| 属性                            | 描述                                                         | 需要 |
| ------------------------------- | ------------------------------------------------------------ | :--- |
| casServerUrlPrefix              | CAS服务器URL的开始，即https://localhost:8443/cas             | 是   |
| serverName                      | 此应用程序所在的服务器的名称。服务URL将使用此动态构建,即[https://localhost:8443](https://localhost:8443/)（您必须包含协议，但如果端口是标准端口，则端口是可选的） | 是   |
| renew                           | 指定是否renew=true应该发送到CAS服务器。有效值是true/false（或根本没有值）。请注意，renew不能将其指定为本地init-param设置 | 否   |
| redirectAfterValidation         | 是否在故障单验证后重定向到相同的URL，但没有参数中的故障单。默认为true | 否   |
| useSession                      | 是否在会话中存储断言。如果不使用会话，则每个请求都需要票证。默认为true | 否   |
| exceptionOnValidationFailure    | 是否在票证验证失败时抛出异常。默认为true                     | 否   |
| proxyReceptorUrl                | 要查看PGTIOU/PGT来自CAS服务器的响应的URL 。应该从上下文的根来定义。例如，如果您的应用程序部署在/cas-client-app您想要的代理服务器URL中，/cas-client-app/my/receptor您需要配置proxyReceptorUrl/my/receptor | 否   |
| acceptAnyProxy                  | 指定是否有任何代理正常。默认为false。                        | 否   |
| allowedProxyChains              | 指定代理链。每个可接受的代理链应包含一个由空格分隔的URL列表（用于完全匹配）或URL的正则表达式（由^字符开始）。每个可接受的代理链应该出现在自己的行上 | 否   |
| proxyCallbackUrl                | 用于提供CAS服务器以接受代理授予票证的回叫URL                 | 否   |
| proxyGrantingTicketStorageClass | 指定具有无参数构造函数的ProxyGrantingTicketStorage类的实现。 | 否   |
| sslConfigFile                   | 包含用于客户端SSL配置的SSL设置的属性文件的引用，用于反向通道调用期间。该配置包括用于键protocol默认为SSL，keyStoreType，keyStorePath，keyStorePass，keyManagerType默认为SunX509和certificatePassword。 | 否   |
| encoding                        | 指定客户端应使用的编码字符集                                 | 否   |
| secretKey                       | proxyGrantingTicketStorageClass它使用的密钥，如果它支持加密。 | 否   |
| cipherAlgorithm                 | 该算法使用的proxyGrantingTicketStorageClass是否支持加密。默认为DESede | 否   |
| millisBetweenCleanUps           | 清理任务的启动延迟从存储中删除过期票证。默认为60000 msec     | 否   |
| ticketValidatorClass            | 要使用/创建票证验证程序类                                    | 否   |
| hostnameVerifier                | 主机名验证程序类名称，用于进行反向通话                       | 否   |

# 五、项目实战【环境搭建】

| 项目名                  | 请求地址                  | 角色      |
| ----------------------- | ------------------------- | --------- |
| cas-server-spring-demo  | http://localhost:8080/cas | cas服务端 |
| cas-proxy-client-demo   | http://localhost:8888/    | 代理端    |
| cas-proxyed-client-demo | http://localhost:8889/    | 被代理端  |

### 代码下载

[CAS源码仓库地址](https://github.com/apereo/cas-overlay-template) 

[CAS服务器demo下载地址](https://github.com/pitt1997/cas-webapp)【cas-server-spring项目是基于4.2.7版本的分支进行二次开发】

```
├── README.md
├── cas-base-server
│   ├── cas-server-spring			CAS服务端
├── cas-proxy-sso-demo				
│   ├── cas-proxy-client-demo		CAS代理客户端
│   ├── cas-proxyed-client-demo		CAS被代理客户端
```

## CAS单点登录服务端实现

### 部署说明

`proxyCallbackUrl`回调地址官方默认必须要配置一个`https`的协议，这也就意味着我们的线上项目，必须支持`https`才可以。而我们的线上项目，大多只支持`http`协议，这该怎么办呢？

经过debug查询cas-server源码发现，最终cas是根据一个有关代理认证策略的正则有关系，默认不允许颁发`PGT`，于是就修改了json格式`service`文件服务中的策略问题得以解决。

**cas代理认证取消HTTPS，支持HTTP方式**

默认的json服务策略中，不支持代理返回PGT，需要在自定义的`service`中增加`proxyPolicy`策略。在pattern中，设置协议的正则规则即可。

```json
{
  "@class": "org.jasig.cas.services.RegexRegisteredService",
  "id": 10000003,
  "serviceId" : "^(https|http|imaps)://.*",
  "name" : "HTTPS and HTTP and IMAPS",
  "description" : "This service definition authorizes all application urls that support HTTPS and HTTP and IMAPS protocols.",
  "evaluationOrder" : 10000,
  "theme": "cas-proxy",
  "attributeReleasePolicy": {
    "@class": "org.jasig.cas.services.ReturnAllAttributeReleasePolicy"
  },
  "proxyPolicy": {
    "@class": "org.jasig.cas.services.RegexMatchingRegisteredServiceProxyPolicy",
    "pattern": "^(https|http)?://.*"
  }
}
```

#### Service配置【cas代理支持HTTP和HTTPS方式】

如果你认真的看源代码，会观察到cas-server的HttpBasedServiceCredentialsAuthenticationHandler类中authenticate方法，如下

```java
public HandlerResult authenticate(Credential credential) throws GeneralSecurityException {
    HttpBasedServiceCredential httpCredential = (HttpBasedServiceCredential)credential;
    if (!httpCredential.getService().getProxyPolicy().isAllowedProxyCallbackUrl(httpCredential.getCallbackUrl())) {
        LOGGER.warn("Proxy policy for service [{}] cannot authorize the requested callback url [{}].", httpCredential.getService().getServiceId(), httpCredential.getCallbackUrl());
        throw new FailedLoginException(httpCredential.getCallbackUrl() + " cannot be authorized");
    } else {
        LOGGER.debug("Attempting to authenticate [{}]", httpCredential);
        URL callbackUrl = httpCredential.getCallbackUrl();
        if (!this.httpClient.isValidEndPoint(callbackUrl)) {
            throw new FailedLoginException(callbackUrl.toExternalForm() + " sent an unacceptable response status code");
        } else {
            return new DefaultHandlerResult(this, httpCredential, this.principalFactory.createPrincipal(httpCredential.getId()));
        }
    }
}
```

`getProxyPolicy（）`方法获取service的策略并根据`isAllowedProxyCallbackUrl`方法判断正则表达式是否允许使用https或http，那么直接设置正则就好了。将上面的`"pattern": "^http?://.*"` 修改为`"pattern": "^(https|http)?://.*"`即可，我已经测试过没问题。

使用https，你需要配置client支持https才行。配置很简单，查看我的github源码即可。 如果你不会配置，可能会遇到cas-server向客户端发送代理回调时出现`PKIX path building failed`等问题。查看下面的常见问题解决即可。

除了以上说的还有很多配置策略以及节点，具体看官方文档，配置不同的RegisteredService也会有稍微不一样

#### cas.properties配置【cas允许http认证】

> cas.properties配置项修改

```
tgc.secure=false
```

#### cas.properties配置【默认密码】

CAS4.2.7版本默认的密码在配置文件：**casuser/Mellon**，可修改为：admin/admin

```
cas.authn.accept.users=admin::admin
```

## CAS【代理客户端】实现

cas客户端是一个SpringBoot微应用，引入cas客户端相关依赖，实现请求过滤器，从而实现通过CAS单点登录服务器进行用户登录的功能。

### 目录结构

```
|- cas-client
    |- src
        |- main
            |- bin
                |- startup.sh   启动脚本
            |- java             Java源代码
            |- resources        Java配置资源文件
    |- web                      前端文件
    |- assembly.xml             工程打包配置文件
    |- pom.xml                  工程配置文件
```

### 引入依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${springboot.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
            <version>${springboot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jasig.cas.client</groupId>
            <artifactId>cas-client-core</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
</dependencies>
```

### 代理方配置

既然`Cas20ProxyReceivingTicketValidationFilter`是一个Ticket Validation Filter，所以之前我们介绍的Ticket Validation Filter需要配置的参数，在这里也需要配置，所不同的是对于代理端的`Cas20ProxyReceivingTicketValidationFilter`必须指定另外的两个参数：`proxyCallbackUrl`和`proxyReceptorUrl`。

- proxyCallbackUrl：用于指定一个回调地址，在代理端通过Cas Server校验ticket成功后，Cas Server将回调该地址以传递`pgtId`和`pgtIou`，`Cas20ProxyReceivingTicketValidationFilter`在接收到对应的响应后会将它们保存在内部持有的`ProxyGrantingTicketStorage`中。之后在对传递过来的`ticket`进行`validate`的时候又会根据`pgtIou`从`ProxyGrantingTicketStorage`中获取对应的`pgtId`，用以保存在`AttributePrincipal`中，而`AttributePrincipal`又会保存在`Assertion`中。`proxyCallbackUrl`因为是指定Cas Server回调的地址，所以其必须是一个可以供外部访问的绝对地址。此外，因为Cas Server默认只回调使用安全通道协议`https`进行通信的地址，所以我们的`proxyCallbackUrl`需要是一个使用`https`协议访问的地址。
- proxyReceptorUrl：该地址是`proxyCallbackUrl`相对于代理端的一个地址， `Cas20ProxyReceivingTicketValidationFilter`将根据该地址来决定请求是否来自Cas Server的回调。

下面是一个`Cas30ProxyReceivingTicketValidationFilter`在代理端配置的示例

```java
/**
 * Cas30ProxyReceivingTicketValidationFilter 验证过滤器
 * 该过滤器负责对Ticket的校验工作，必须启用它
 */
@Bean
public FilterRegistrationBean filterValidationRegistration() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new Cas30ProxyReceivingTicketValidationFilter());
    // 设定匹配的路径
    registration.addUrlPatterns("/*");
    Map<String, String> initParameters = new HashMap();
    initParameters.put("casServerUrlPrefix", CasConfig.CAS_SERVER_PATH);
    initParameters.put("serverName", CasConfig.SERVER_NAME);
    
    initParameters.put("proxyCallbackUrl", CasConfig.PROXY_CALLBACK_URL);
    initParameters.put("proxyReceptorUrl", "/proxy/callback");
    
    // 是否对serviceUrl进行编码，默认true：设置false可以在302对URL跳转时取消显示;jsessionid=xxx的字符串
    // 观察CommonUtils.constructServiceUrl方法可以看到
    initParameters.put("encodeServiceUrl", "false");
    
    registration.setInitParameters(initParameters);
    // 设定加载的顺序
    registration.setOrder(1);
    return registration;
}
```

代理请求（Cas代理回调处理）示例

```java
@RestController
@RequestMapping("/proxy")
public class CasProxyController {

    @GetMapping("/users")
    public String proxyUsers(HttpServletRequest request, HttpServletResponse response) {
        String result = "无结果";
        try {
            String serviceUrl = "http://localhost:8889/user/users";
            AttributePrincipal principal = (AttributePrincipal) request.getUserPrincipal();
            // 1、获取到AttributePrincipal对象
//            AttributePrincipal principal = AssertionHolder.getAssertion().getPrincipal();
            if (principal == null) {
                return "用户未登录";
            }

            //2、获取对应的(PT)proxy ticket
            String proxyTicket = principal.getProxyTicketFor(serviceUrl);
            if (proxyTicket == null) {
                return "PGT 或 PT 不存在";
            }

            //3、请求被代理应用时将获取到的proxy ticket以参数ticket进行传递
            String url = serviceUrl + "?ticket=" + URLEncoder.encode(proxyTicket, "UTF-8");
            result = HttpProxy.httpRequest(url, "", HttpMethod.GET);

            System.out.println("result结果：" + result);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return result;
    }


    @GetMapping("/books")
    public String proxyBooks(HttpServletRequest request, HttpServletResponse response) {

        String result = "无结果";
        try {
            String serviceUrl = "http://client2.com:8889/book/books";

            //1、获取到AttributePrincipal对象
            AttributePrincipal principal = AssertionHolder.getAssertion().getPrincipal();
            if (principal == null) {
                return "用户未登录";
            }

            //2、获取对应的(PT)proxy ticket
            String proxyTicket = principal.getProxyTicketFor(serviceUrl);
            if (proxyTicket == null) {
                return "PGT 或 PT 不存在";
            }

            //3、请求被代理应用时将获取到的proxy ticket以参数ticket进行传递
            String url = serviceUrl + "?ticket=" + URLEncoder.encode(proxyTicket, "UTF-8");
            result = HttpProxy.httpRequest(url, "", HttpMethod.GET);

            System.out.println("result结果：" + result);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return result;
    }
}
```

## CAS【被代理客户端】实现

在被代理端`Cas30ProxyReceivingTicketValidationFilter`是扮演Ticket Validation Filter的角色，它可以验证正常通过Cas Server登录认证成功后返回的`ticket`，也可以认证来自其它代理端传递过来的`proxy ticket（PT）`，当然，最终的认证都是通过Cas Server来完成的。既然`Cas30ProxyReceivingTicketValidationFilter`在被代理端是作为Ticket Validation Filter来使用的，可以有的参数其都可以配置。在被代理端需要配置一个参数用以表示接受来自哪些应用的代理，这个参数可以是`acceptAnyProxy`，也可以是`allowedProxyChains`。

- acceptAnyProxy：表示接受所有的，其对应的参数值是true或者false
- allowedProxyChains：指定具体接受哪些应用的代理，多个应用就写多行，`allowedProxyChains`的值对应的是代理端提供给Cas Server的回调地址，如果使用前文示例的代理端配置，我们就可以指定被代理端的`allowedProxyChains`为`https://elim:8043/app1/proxyCallback`，这样当app1作为代理端来访问该被代理端时就能通过验证，得到正确的响应。

下面是一个被代理端配置`Cas30ProxyReceivingTicketValidationFilter`的配置示例。 多了一个`acceptAnyProxy`参数而已。

```java
@Bean
public FilterRegistrationBean filterValidationRegistration() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new Cas30ProxyReceivingTicketValidationFilter());
    // 设定匹配的路径
    registration.addUrlPatterns("/*");
    Map<String, String> initParameters = new HashMap();
    initParameters.put("casServerUrlPrefix", CasConfig.CAS_SERVER_PATH);
    initParameters.put("serverName", CasConfig.SERVER_NAME);
    initParameters.put("encodeServiceUrl", "false");
    
    initParameters.put("acceptAnyProxy", "true");   // 接收任何代理
    
    registration.setInitParameters(initParameters);
    // 设定加载的顺序
    registration.setOrder(1);
    return registration;
}
```



# 六、测试

1、http://localhost:8888/代理客户端访问受限地址，先进行CAS服务器登录

2、CAS服务器登录后，访问代理的url时http://localhost:8888/proxy/users，直接返回给被代理端的对应数据



# 七、参考

- CAS代理协议流程图：https://github.com/X-rapido/CAS_SSO_Record/blob/master/assets/pdf/cas_proxy_protocol.pdf
- CAS协议3.0规范，参考：https://apereo.github.io/cas/5.2.x/protocol/CAS-Protocol-Specification.html
- cas-client.jar 配置项参数，参考：https://github.com/apereo/java-cas-client
- https://apereo.atlassian.net/wiki/spaces/CASC/pages/103252629/Yale+CAS+Clients+2.0.11
- [Cas 5.2.x版本使用 —— 代理认证实现SSO（十四） - Java开发 - 程序喵 (ibloger.net)](http://www.ibloger.net/article/3129.html)
- cas客户端实现参考：http://www.scassis.cn/blog/2019/02/02/cas-client/#more
- cas server:http://www.scassis.cn/blog/2019/02/02/cas-server/
- cas服务端搭建：https://github.com/johnsonmoon/cas-example/blob/master/cas-server/README.md

- cas 4.2.7demo搭建：https://blog.csdn.net/pucao_cug/article/details/70182968
- cas client：https://github.com/apereo/java-cas-client

# CAS单点登录技术支持文档

## 4.2.x

该文档所涉及的CAS版本是4.2.x

CAS官网地址：https://apereo.github.io/cas/4.2.x/

CAS源码地址：https://github.com/apereo/cas/tree/4.2.x

CAS源码4.2.7版地址：https://github.com/apereo/cas/releases/tag/v4.2.7

CAS安装：https://apereo.github.io/cas/4.2.x/installation/Maven-Overlay-Installation.html

## 企业单点登录

- Java（Spring Webflow/MVC servlet）服务器组件
- 可插拔身份验证支持（LDAP、数据库、X.509、2-factor）
- 支持多种协议（CAS、SAML、OAuth、OpenID）
- 跨平台客户端支持（Java、.Net、PHP、Perl、Apache 等）

## CAS架构

CAS 系统架构包括CAS 服务器和客户端两个物理组件，它们通过各种协议进行通信。

![image-20210723222557897](C:\Users\17996\AppData\Roaming\Typora\typora-user-images\image-20210723222557897.png)

### CAS服务器

CAS 服务器是构建在 Spring Framework 上的 **Java servlet**，其主要职责是通过【**发布票据**】和【**验证票据**】来验证用户并授予对启用 CAS 的服务的【**CAS 客户端**】的访问权限。当服务器在成功登录后向用户发出票据授予【**票据TGT**】时，将创建 SSO 会话。【**服务票据ST**】根据用户的请求通过浏览器重定向使用【**票据TGT**】作为令牌颁发给服务。【**ST**】随后在CAS服务器上通过反向通道通信进行【**验证**】，这些交互在【**CAS协议文档**】中有非常详细的描述。

### CAS客户端

平台：

- Apache httpd 服务器（[mod_auth_cas 模块](https://github.com/Jasig/mod_auth_cas)）
- Java（[Java CAS 客户端](https://github.com/apereo/java-cas-client)）
- .NET（[.NET CAS 客户端](https://github.com/apereo/dotnet-cas-client)）
- PHP ( [phpCAS](https://github.com/Jasig/phpCAS) )
- Perl (PerlCAS)
- python（pycas）
- Ruby（rubycas 客户端）

### 协议

客户端通过多种支持的协议中的任何一种与服务器进行通信。所有支持的协议在概念上都是相似的，但有些协议具有使它们适合特定应用程序或用例的特性或特性。比如**CAS协议支持委托（代理）认证**，**SAML协议支持属性发布和单点退出**。

支持的协议：

- [CAS（版本 1、2 和 3）](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol.html)
- [SAML 1.1](https://apereo.github.io/cas/4.2.x/protocol/SAML-Protocol.html)
- [OpenID](https://apereo.github.io/cas/4.2.x/protocol/OpenID-Protocol.html)
- [OAuth (1.0, 2.0)](https://apereo.github.io/cas/4.2.x/protocol/OAuth-Protocol.html)

### 组件

- Web (Spring MVC/Spring Webflow)
- [售票处](https://apereo.github.io/cas/4.2.x/installation/Configuring-Ticketing-Components.html)
- [验证](https://apereo.github.io/cas/4.2.x/installation/Configuring-Authentication-Components.html)

几乎所有的部署考虑和组件配置都涉及这三个子系统。【**Web 层**】**是与包括CAS客户端在内的所有外部系统进行通信的端点**。【**Web层**】**委托票务子系统为【CAS客户端】访问生成票证**。SSO 会话开始于在成功【**验证**】时颁发授予票证的票证，因此【**票证子系统**】经常委托给身份验证子系统。【**身份验证系统**】通常仅在 SSO 会话开始时处理请求，但在其他情况下也可以调用它（例如强制身份验证）。

### 框架

CAS 使用了 Spring 框架的许多方面；最值得注意的是 [Spring MVC](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html)和 [Spring Webflow](http://www.springsource.org/spring-web-flow)。Spring 为核心 CAS 代码库以及部署者提供了一个完整且可扩展的框架；通过挂钩 CAS 和 Spring API 扩展点来自定义或扩展 CAS 行为很简单。Spring 的一般知识有助于理解一些框架组件之间的相互作用，但不是严格要求的。然而，用于配置 CAS 和 Spring 组件的基于 XML 的配置是安装、定制和扩展的核心问题。对 XML 的能力， 尤其是[Spring IOC Container](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html)，是安装 CAS 的先决条件。

## CAS协议

CAS 协议是一种简单而强大的【**基于票据**】的协议，专为 CAS 开发。可以在[此处](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html)找到完整的协议规范。

它涉及一个或多个客户端和一个服务器。客户端嵌入【**CAS应用程序**】中，而 CAS 服务器是一个独立的组件：

- **[CAS服务器](https://apereo.github.io/cas/4.2.x/installation/Configuring-Authentication-Components.html)负责【验证用户并授予访问】应用程序**
- [**CAS客户端**](https://apereo.github.io/cas/4.2.x/integration/CAS-Clients.html)**保护【CAS应用程序】和检索CAS服务器的授权用户的身份。**

关键概念是：

- **存储在 CAS TGC cookie 中的 TGT（Ticket Granting Ticket）代表用户的 SSO 会话**
- **ST（服务票证）作为 url 中的 GET 参数传输，代表 CAS 服务器授予特定用户对*CAS*应用程序的访问权限。**

## CAS流程

当前的 CAS 协议规范是`3.x`. 实际协议规范可在[CAS-Protocol-Specification 获得](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html)，特此由 Apereo CAS Server 实现，作为官方参考实现。

### 流程图

![image-20210724002014928](C:\Users\17996\AppData\Roaming\Typora\typora-user-images\image-20210724002014928.png)



### 代理网页流程图

CAS 协议最强大的功能之一是 CAS 服务能够充当另一个 CAS 服务的代理，传输用户身份。

![image-20210802215303378](C:\Users\17996\AppData\Roaming\Typora\typora-user-images\image-20210802215303378.png)



## 其他协议

即使 CAS 服务器的主要目标是实现 CAS 协议，也支持其他协议作为扩展：

- [开放ID](https://apereo.github.io/cas/4.2.x/protocol/OpenID-Protocol.html)
- [身份验证](https://apereo.github.io/cas/4.2.x/protocol/OAuth-Protocol.html)
- [安全反洗钱](https://apereo.github.io/cas/4.2.x/protocol/SAML-Protocol.html)

## 委托认证

使用 CAS 协议，CAS 服务器还可以配置为[将身份验证委托](https://apereo.github.io/cas/4.2.x/integration/Delegate-Authentication.html)给另一个 CAS 服务器



## 安装部署

### WAR覆盖安装

CAS 安装基本上是一个面向源代码的过程，我们推荐一个 [WAR 覆盖](http://maven.apache.org/plugins/maven-war-plugin/overlays.html)项目来组织自定义，例如组件配置和 UI 设计。WAR 覆盖构建的输出是一个`cas.war`可以部署在 Java servlet 容器（如 [Tomcat ）上的文件](http://tomcat.apache.org/whichversion.html)。

提供WAR覆盖项目供参考学习。

#### CAS Maven 覆盖

CAS 使用 【**Spring Webflow**】 以【模块化和可配置的方式】驱动登录过程；该`login-webflow.xml` **文件包含对流中状态和转换的直接描述**。除了 Spring XML 配置文件中的组件配置之外，定制这个文件可能是最常见的配置问题。有关各种 CAS 流的详细描述和常见配置点请参阅 【Spring Webflow 自定义指南】。

#### CAS 配置

CAS 服务器严重依赖 Spring 框架。有确切的下和特殊的XML配置文件`spring-configuration`目录CAS控制各种属性以及`cas-servlet.xml`和`deployerConfigContext.xml`其主要由中科院使用者预计后者将包括在覆盖特定环境-CAS设置。

如果需要，可以通过 Maven 覆盖过程覆盖 XML 配置文件中的 Spring bean 以更改行为。对此有两种方法：

1、**XML 文件可以从 CAS 版本的源中获取，并在 Maven 覆盖构建中以相同的确切名称放置在相同的确切路径中。如果配置正确，构建将使用本地提供的 XML 文件而不是默认文件。**

2、**CAS 服务器能够加载 XML 配置文件的模式以覆盖默认提供的内容。这些旨在否决 CAS 默认行为的配置文件可以放置在`/WEB-INF/`并且必须由以下模式命名：`cas-servlet-*.xml`. 放置在此文件中的 Bean 将覆盖其他 Bean。**

#### 自定义和第三方来源

通过开发实现 CAS API 的 Java 组件或通过 Maven 依赖项引用包含第三方源来定制或扩展 CAS 的功能是很常见的。包括第三方来源是微不足道的；只需在覆盖`pom.xml`文件中包含相关的依赖项。**为了包含自定义 Java 源代码，它应该包含在`src/java/main`覆盖项目源代码树的目录下**。

```java
├── src
│   ├── main
│   │   ├── java
│   │   │   └── edu
│   │   │       └── vt
│   │   │           └── middleware
│   │   │               └── cas
│   │   │                   ├── audit
│   │   │                   │   ├── CompactSlf4jAuditTrailManager.java
│   │   │                   │   ├── CredentialsResourceResolver.java
│   │   │                   │   ├── ServiceResourceResolver.java
│   │   │                   │   └── TicketOrCredentialPrincipalResolver.java
│   │   │                   ├── authentication
│   │   │                   │   └── principal
│   │   │                   │       ├── AbstractCredentialsToPrincipalResolver.java
│   │   │                   │       ├── PDCCredentialsToPrincipalResolver.java
│   │   │                   │       └── UsernamePasswordCredentialsToPrincipalResolver.java
│   │   │                   ├── services
│   │   │                   │   └── JsonServiceRegistryDao.java
│   │   │                   ├── util
│   │   │                   │   └── X509Helper.java
│   │   │                   └── web
│   │   │                       ├── HelpController.java
│   │   │                       ├── RegisteredServiceController.java
│   │   │                       ├── StatsController.java
│   │   │                       ├── WarnController.java
│   │   │                       ├── flow
│   │   │                       │   ├── AbstractForgottenCredentialAction.java
│   │   │                       │   ├── AbstractLdapQueryAction.java
│   │   │                       │   ├── AffiliationHandlerAction.java
│   │   │                       │   ├── CheckAccountRecoveryMaintenanceAction.java
│   │   │                       │   ├── CheckPasswordExpirationAction.java
│   │   │                       │   ├── ForgottenCredentialTypeAction.java
│   │   │                       │   ├── LookupRegisteredServiceAction.java
│   │   │                       │   ├── NoSuchFlowHandler.java
│   │   │                       │   ├── User.java
│   │   │                       │   ├── UserLookupAction.java
│   │   │                       │   └── WarnCookieHandlerAction.java
│   │   │                       └── util
│   │   │                           ├── ProtocolParameterAuthority.java
│   │   │                           ├── UriEncoder.java
│   │   │                           └── UrlBuilder.java
```

另外，请注意，对于要编译并包含在最终`cas.war`文件中的任何自定义 Java 组件，`pom.xml`Maven 覆盖中的 必须包含对 Maven 【Java 编译器】的引用，以便类可以编译。

```xml
...

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.3</version>
            <configuration>
                <warName>cas</warName>
                <overlays>
                    <overlay>
                        <groupId>org.jasig.cas</groupId>
                        <artifactId>cas-server-webapp</artifactId>
                        <excludes>
                <exclude>WEB-INF/cas.properties</exclude>
                            <exclude>WEB-INF/classes/log4j.xml</exclude>
                            <exclude>...</exclude>
                        </excludes>
                    </overlay>
                </overlays>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>${java.source.version}</source>
                <target>${java.target.version}</target>
            </configuration>
        </plugin>

    </plugins>
    <finalName>cas</finalName>
</build>

...
```

文件系统层次可视化由`tree`程序生成。

## CAS认证过程

### 验证

CAS 认证过程由几个相关组件执行：

#### PrincipalNameTransformer

将输入到【登录表单】中的【用户 id 字符串】转换为**由特定类型的身份验证处理程序验证的暂定主体名称**。

#### AuthenticationManager

身份验证子系统的入口点。它接受一个或多个凭据并将身份验证委托给配置的**`AuthenticationHandler`组件**。它收集每次尝试的结果并确定有效的安全策略。

#### AuthenticationHandler

验证单个凭证并报告三种可能结果之一：成功、失败、未尝试。

#### PrincipalResolver

将身份验证凭据中的信息转换为安全主体，该主体通常包含其他元数据属性（即用户详细信息，例如隶属关系、组成员身份、电子邮件、显示名称）。

#### AuthenticationMetaDataPopulator

用于设置有关成功身份验证事件的任意元数据的策略组件；这些通常用于设置特定于协议的数据。除非另有说明，**所有【身份验证组件】的配置都在`deployerConfigContext.xml`.**

### 身份验证管理器

CAS 附带一个单一而灵活的【身份验证管理器】，【**PolicyBasedAuthenticationManager**】应该足以满足大多数需求。它根据以下合同执行身份验证。

对于每个给定的凭据，请执行以下操作：

1. 迭代所有配置的身份验证处理程序。
2. 如果处理程序支持凭据，则尝试对其进行身份验证。
3. 成功尝试解决委托人。
4. 检查是否为验证凭据的处理程序配置了解析器。
5. 如果找到合适的解析器，请尝试解析主体。
6. 如果找不到合适的解析器，请使用由身份验证处理程序解析的主体。
7. 检查是否满足安全策略（例如任何、全部）。
8. 如果符合安全政策，立即返回。
9. 如果不满足安全策略，则继续。
10. 在尝试了所有凭据后，再次检查安全策略，【**AuthenticationException**】 如果不满意则抛出。

有一个隐式安全策略需要至少一个处理程序来成功验证凭据，但可以通过【**`#setAuthenticationPolicy(AuthenticationPolicy)`** 】使用以下策略之一进行设置来进一步控制该行为。

#### AnyAuthenticationPolicy

如果任何处理程序成功，则满意。支持`tryAll`在上面的步骤 4.1 中避免短路的标志，并尝试每个处理程序，即使之前的一个成功。此策略是默认策略，并提供与`AuthenticationManagerImpl`CAS 3.x 组件的向后兼容行为 。

#### AllAuthenticationPolicy

当且仅当所有给定的凭据都成功通过身份验证时才满意。对多个凭证的支持是 CAS 中的新功能，此处理程序仅在多因素身份验证情况下才可接受。

#### RequiredHandlerAuthenticationPolicy

只有当指定的处理程序成功验证其凭据时才满足。支持`tryAll`在上面的步骤 4.1 中避免短路的标志，并尝试每个处理程序，即使之前的一个成功。此策略可用于支持多因素身份验证情况，例如，需要用户名/密码身份验证但额外的 OTP 是可选的。

### 身份验证处理程序

CAS 支持针对许多常见类型的身份验证系统进行身份验证。以下列表提供了支持的身份验证技术的完整列表；

- [数据库](https://apereo.github.io/cas/4.2.x/installation/Database-Authentication.html)
- [JAAS](https://apereo.github.io/cas/4.2.x/installation/JAAS-Authentication.html)
- [LDAP](https://apereo.github.io/cas/4.2.x/installation/LDAP-Authentication.html)
- [OAuth 1.0/2.0，OpenID](https://apereo.github.io/cas/4.2.x/installation/OAuth-OpenId-Authentication.html)
- [半径](https://apereo.github.io/cas/4.2.x/installation/RADIUS-Authentication.html)
- [SPNEGO](https://apereo.github.io/cas/4.2.x/installation/SPNEGO-Authentication.html) (Windows)
- [受信任](https://apereo.github.io/cas/4.2.x/installation/Trusted-Authentication.html)(REMOTE_USER)
- [X.509](https://apereo.github.io/cas/4.2.x/installation/X509-Authentication.html)（客户端 SSL 证书）
- [远程地址](https://apereo.github.io/cas/4.2.x/installation/Remote-Address-Authentication.html)
- [尤比钥匙](https://apereo.github.io/cas/4.2.x/installation/YubiKey-Authentication.html)
- [阿帕奇四郎](https://apereo.github.io/cas/4.2.x/installation/Shiro-Authentication.html)
- [pac4j](https://apereo.github.io/cas/4.2.x/installation/Pac4j-Authentication.html)

对于小型部署和特殊情况，还有一些额外的处理程序：

- [列表](https://apereo.github.io/cas/4.2.x/installation/Whitelist-Authentication.html)
- [黑名单](https://apereo.github.io/cas/4.2.x/installation/Blacklist-Authentication.html)

**默认凭据**

```
要测试 CAS 中的默认身份验证方案，请分别使用casuser和Mellon作为用户名和密码。
```

### 密码编码

密码编码器负责在身份验证事件期间将凭证密码转换和编码为身份验证源可接受的形式。

#### 默认编码器

```xml
<alias name="defaultPasswordEncoder" alias="passwordEncoder" />
```

以下设置适用：

```
# cas.authn.password.encoding.char=UTF-8
# cas.authn.password.encoding.alg=SHA-256
```

#### 纯文本

```xml
<alias name="plainTextPasswordEncoder" alias="passwordEncoder" />
```

### 参数提取器

提取器负责检查收到的 http 请求，以获取描述身份验证请求（例如请求`service`等）的参数。提取器存在用于许多受支持的身份验证协议，并且每个都创建【**WebApplicationService**】包含提取结果的适当实例。

### 主要决议

有关主要决议的更多详细信息，请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-Principal-Resolution.html)。

#### 主要转化

通常处理用户名密码凭据的身份验证处理程序可以配置为在执行身份验证序列之前转换用户 ID。以下组件可用：

#### `NoOpPrincipalNameTransformer`

默认转换器，实际上不对用户 id 进行转换。

```xml
<alias name="noOpPrincipalNameTransformer" alias="principalNameTransformer" />
```

#### `PrefixSuffixPrincipalNameTransformer`

通过添加后缀或后缀来转换用户 ID。

```xml
<alias name="prefixSuffixPrincipalNameTransformer" alias="principalNameTransformer" />
```

以下设置适用cas.properties：

```
# cas.principal.transform.prefix=
# cas.principal.transform.suffix=
```

#### `ConvertCasePrincipalNameTransformer`

将表单 uid 转换为小写或大写的转换器。结果也被修剪。转换器还能够接受并处理可能修改了 uid 的前一个转换器的结果，以便可以将两者链接起来。

```xml
<alias name="convertCasePrincipalNameTransformer" alias="principalNameTransformer" />
```

以下设置适用：

```
# cas.principal.transform.upperCase=false
```

### 认证元数据

【**AuthenticationMetaDataPopulator**】组件提供了一种可插入的策略，用于将【任意元数据】注入到【身份验证子系统】中，供【其他子系统或外部组件】使用。元数据填充器的一些显着用途：

- **支持长期认证功能**
- **SAML 协议支持**
- **OAuth 和 OpenID 协议支持。**

对于大多数部署，默认的身份验证元数据填充器应该足够了。如果组件需要支持可选的 CAS 功能，则将明确标识它们并提供配置。

### 长期认证

CAS 支持长期票证授予票证，该功能也称为【**"记住我"**】以将 SSO 会话的长度扩展到典型配置之外。请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-LongTerm-Authentication.html)了解更多详情。

### 代理认证

**请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-Proxy-Authentication.html)了解更多详情。**

默认情况下启用对 CAS v1+ 协议的代理身份验证支持，因此利用代理身份验证功能完全是 CAS 客户端配置的问题。

**代理认证参考：http://www.ibloger.net/article/3129.html**

#### 什么是 CAS 代理认证

考虑这样一种场景：有两个应用App1和App2，它们都是受Cas Server保护的，即请求它们时都需要通过Cas Server的认证。现需要在App1中通过Http请求访问App2，显然该请求将会被App2配置的Cas的`AuthenticationFilter`拦截并转向Cas Server，Cas Server将引导用户进行登录认证，这样我们也就不能真正的访问到App2了。针对这种应用场景，Cas也提供了对应的支持。

#### **服务配置**

请注意，注册表中的每个注册应用程序都必须明确配置为允许代理身份验证。请参阅[本指南](https://apereo.github.io/cas/4.2.x/installation/Service-Management.html) 以了解如何在注册表中注册服务。

对于希望从策略上避免代理身份验证作为安全策略问题的部署，建议禁用代理身份验证组件。

#### 处理启用 SSL 的代理 URL

默认情况下，CAS 附带一个捆绑的 HTTP 客户端，该客户端部分负责回调 URL 以进行代理身份验证。请注意，此 URL 还需要经过 CAS 服务注册中心的授权才能进行回调。[有关](https://apereo.github.io/cas/4.2.x/installation/Service-Management.html)更多信息，[请参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Service-Management.html)。

如果回调 URL 由服务注册中心授权，并且端点在 HTTPS 下并受 SSL 证书保护，则 CAS 还将尝试验证端点证书的有效性，然后才能建立成功的连接。如果证书无效、过期、缺少链中的一个步骤、自签名或其他方式，CAS 将无法执行回调。

CAS 的 HTTP 客户端确实提供了一个类似于 Java 平台的本地信任库。建议使用这个信任库来处理所有需要导入平台的证书的管理，让CAS成功执行回调URL。虽然在默认情况下，本地信任存储到CAS是空的，CAS仍然会利用**双方**的默认和本地信任存储。本地信任库当然应该只用于与 CAS 相关的功能，信任库文件可以在 CAS 和 Java 升级之间转移，当然由应该托管所有 CAS 配置的源控制系统管理。

```
# The http client truststore file, in addition to the default's
# http.client.truststore.file=classpath:truststore.jks
#
# The http client truststore's password
# http.client.truststore.psw=changeit
```

#### 在验证响应中返回 PGT

在使用`CAS20ProxyHandler`可能不受欢迎的情况下，例如调用回调 url 来接收代理授权票证是不可行的，CAS 可以配置为直接在验证响应中返回代理授权票证 ID。为了在 CAS 服务器和应用程序之间成功建立信任，客户端应用程序生成私钥/公钥对，然后在 CAS 内部分发和配置**公钥**。CAS 将使用公钥加密代理授予票证 id，并将`<proxyGrantingTicketId>` 在验证响应中发布一个新属性，仅当服务被授权接收它时。

请注意，如果客户端应用程序向`/p3/serviceValidate`端点（或`/p3/proxyValidate`）发出请求，则代理授予票证 ID 的返回仅由 CAS 验证响应执行。其他将属性返回给 CAS 的方法，例如 SAML1，将**不**支持额外返回代理授权票证。

#### 配置

#### 注册服务

从客户端应用程序所有者那里收到公钥后，必须首先在 CAS 服务器的服务注册表中注册它。持有上述公钥的服务也必须被授权接收 PGT 作为选择的给定属性发布策略的属性。

```
{
  "@class" : "org.jasig.cas.services.RegexRegisteredService",
  "serviceId" : "^https://.+",
  "name" : "test",
  "id" : 1,
  "evaluationOrder" : 0,
  "attributeReleasePolicy" : {
    "@class" : "org.jasig.cas.services.ReturnAllowedAttributeReleasePolicy",
    "principalAttributesRepository" : {
      "@class" : "org.jasig.cas.authentication.principal.DefaultPrincipalAttributesRepository"
    },
    "authorizedToReleaseCredentialPassword" : false,
    "authorizedToReleaseProxyGrantingTicket" : true
  },
  "publicKey" : {
    "@class" : "org.jasig.cas.services.RegisteredServicePublicKeyImpl",
    "location" : "classpath:RSA1024Public.key",
    "algorithm" : "RSA"
  }
}
```

#### 解密 PGT id

一旦客户端应用程序收到`proxyGrantingTicket`CAS 验证响应中的id 属性，它就可以通过自己的私钥对其进行解密。由于属性默认是base64编码的，所以需要先解码才能解密。这是一个示例代码片段：

```java
final Map<?, ?> attributes = ...
final String encodedPgt = (String) attributes.get("proxyGrantingTicket");
final PrivateKey privateKey = ...
final Cipher cipher = Cipher.getInstance(privateKey.getAlgorithm());
final byte[] cred64 = decodeBase64ToByteArray(encodedPgt);
cipher.init(Cipher.DECRYPT_MODE, privateKey);
final byte[] cipherData = cipher.doFinal(cred64);
return new String(cipherData);
```

### 多重身份验证 (MFA)

请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-Multifactor-Authentication.html)了解更多详情。

### 登录限制

CAS 提供了一种工具来限制失败的登录尝试，以支持密码猜测和相关的滥用场景。有关登录限制的其他详细信息，请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-Authentication-Throttling.html)。

### SSO 会话 Cookie

票证授予 cookie 是 CAS 在建立单点登录会话时设置的 HTTP cookie。此 cookie 为客户端维护登录状态，并且当它有效时，客户端可以将其提供给 CAS 以代替主要凭据。有关更多详细信息，请[参阅本指南](https://apereo.github.io/cas/4.2.x/installation/Configuring-SSO-Session-Cookie.html)。



## CAS故障排查指南

https://apereo.github.io/cas/4.2.x/installation/Troubleshooting-Guide.html



从结构来看，CAS主要分为Server和Client。

Server主要负责对用户的认证工作；

Client负责处理客户端 受保护资源的访问请求，登录时，重定向到Server进行认证。

 基础模式的SSO访问流程步骤： 

1. 访问服务：客户端发送请求访问应用系统提供的服务资源。 
2. 定向认证：客户端重定向用户请求到中心认证服务器。
3. 用户认证：用户进行身份认证 
4. 发放票据：服务器会产生一个随机的 Service Ticket 。
5. 验证票据： SSO 服务器验证票据 Service Ticket 的合法性，验证通后，允许客户端访问服务。
6. 传输用户信息： SSO 服务器验证票据通过后，传输用户认证结果信息给客户端。



https://apereo.github.io/cas/4.2.x/installation/Configuring-Authentication-Components.html

https://apereo.github.io/cas/4.2.x/installation/Configuring-Multifactor-Authentication.html

https://github.com/casinthecloud/cas-pac4j-oauth-demo/tree/4.2.x

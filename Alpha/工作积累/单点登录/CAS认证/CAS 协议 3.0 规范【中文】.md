# CAS 协议 3.0 规范

# 作者版本

版本：3.0.2

发布日期：2015-01-13

版权所有 © 2005，耶鲁大学

版权所有 © 2015, Apereo, Inc.

# 1. 简介

这是 CAS 1.0、2.0 和 3.0 协议的官方规范。CAS是一种用于 Web 的【单点登录/单点注销】协议。它允许用户访问多个应用程序，同时仅向**中央 CAS 服务器应用程序**提供一次其凭据（例如用户 ID 和密码）。

## 1.1. 约定和定义

本文档中的关键词“必须”、“不得”、“要求”、“应该”、“不应”、“应该”、“不应该”、“推荐”、“可以”和“可选”是解释为 RFC 2119 [1 中所述](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#1)。

- 【客户端】是指最终用户和/或网络浏览器。
- 【CAS 客户端】是指与 Web 应用程序集成并通过 CAS 协议与 CAS 服务器交互的软件组件。
- 【服务器】是指中央身份验证服务服务器。
- 【服务】是指客户端尝试访问的应用程序。
- 【后端服务】是指服务试图代表客户端访问的应用程序。这也可以称为“目标服务”。
- 【SSO】是指单点登录。
- 【SLO】是指单点注销。
- 【<LF>】是一个空换行符（ASCII 值 0x0a）。

## 1.2 参考实现

Apereo CAS-Server [8](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#8)是 CAS 协议规范的官方参考实现。Apereo CAS Server 4.x 支持 CAS 协议 3.0 规范。

# 2. CAS URI

CAS是一个HTTP [2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#2)，[3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#3)系的协议，需要每个组件是通过特定的URI访问。本节将讨论每个 URI：

| URI                     | 描述                            |
| ----------------------- | :------------------------------ |
| **/login**              | **凭证请求者/接受者**           |
| **/logout**             | **销毁 CAS 会话（注销）**       |
| **/validate**           | **服务票证验证 [CAS1.0]**       |
| **/serviceValidate**    | **服务票证验证 [CAS 2.0]**      |
| **/proxyValidate**      | **服务/代理票证验证 [CAS 2.0]** |
| **/proxy**              | **代购票服务 [CAS 2.0]**        |
| **/p3/serviceValidate** | **服务票证验证 [CAS 3.0]**      |
| **/p3/proxyValidate**   | **服务/代理票证验证 [CAS 3.0]** |

## 2.1. /login 作为【凭证请求者】

如果客户端已经与 CAS 建立了单点登录会话，则 Web 浏览器会向 CAS 提供一个安全 cookie，其中包含标识授予票据的字符串。此 cookie 称为【票证授予 cookie】。如果票据授予 cookie 密钥为有效的票据授予票据，则 CAS 可以发布【服务票据】，前提是满足本规范中的所有其他条件。有关票证授予 cookie 的更多信息，请参阅第[3.6](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.6)节。

### 2.1.1. 参数

`/login`当它充当凭证请求者时，可能会传递以下 HTTP 请求参数。它们都区分大小写，并且它们都必须由`/login`.

- **`service`[可选]** - **客户端尝试访问的应用程序的标识符**。在几乎所有情况下，这将是应用程序的 URL。作为 HTTP 请求参数，此 URL 值必须按照 RFC 3986 [ [4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4) ] 的第 2.2 节中所述进行 **URL 编码**。如果`service`未指定并且单点登录会话尚不存在，则 CAS 应该向用户请求凭据以启动单点登录会话。如果`service`未指定并且单点登录会话已经存在，则 CAS 应该显示一条消息，通知客户端它已经登录。

> 注意：强烈建议`service`通过服务管理工具过滤所有url，这样只有经过授权和已知的客户端应用程序才能使用 CAS 服务器。保持服务管理工具开放以允许对所有应用程序的宽松访问可能会增加服务攻击和其他安全漏洞的风险。此外，建议仅`https`允许客户端应用程序使用安全协议，以进一步加强对客户端的验证。

- **`renew`[可选]** - 如果设置了此参数，**将绕过单点登录，在这种情况下，无论是否存在与 CAS 的单点登录会话，CAS 都将要求客户端提供凭据**。此参数与**`gateway`**参数不兼容。【重定向到`/login`URI 的服务】和【发布到 URI 的登录表单视图`/login` 】不应同时设置 `renew`和`gateway`请求参数。如果两者都设置，则行为未定义。建议 CAS 实现忽略`gateway`参数`renew`设置。建议将`renew`参数设置为“true”。

- **`gateway`[可选]** - 如果设置了此参数，**CAS 将不会要求客户端提供凭据**。如果客户端预先存在与 CAS 的单点登录会话，或者如果可以通过非交互方式（即信任身份验证）建立单点登录会话，则 CAS 可以将客户端重定向到`service`参数指定的 URL ，附加有效的服务票证。（CAS 也可以插入一个咨询页面，通知客户端已经进行了 CAS 身份验证。）如果客户端没有与 CAS 的单点登录会话，并且无法建立非交互式身份验证，则 CAS 必须重定向客户端到`service`参数指定的 URL ，没有在 URL 后附加“ticket”参数。如果`service`未指定参数并且`gateway`设置，CAS 的行为是未定义的。建议在这种情况下，CAS 请求凭据就好像没有指定任何参数一样。此参数与`renew`参数不兼容。如果两者都设置，则行为未定义。建议将`gateway`参数设置为“true”。
- **`method`[可选，CAS 3.0]** -`method`发送响应时使用。虽然本机 HTTP 重定向 (GET) 可以用作默认方法，但需要 POST 响应的应用程序可以使用此参数来指示方法类型。由 CAS 服务器实现来确定是否支持 POST 响应。

### 2.1.2. /login 的 URL 示例

简单登录示例：

```
https://cas.example.org/cas/login?service=http%3A%2F%2Fwww.example.org%2Fservice
```

不提示输入用户名/密码：

```
https://cas.example.org/cas/login?service=http%3A%2F%2Fwww.example.org%2Fservice&gateway=true
```

总是提示输入用户名/密码：

```
https://cas.example.org/cas/login?service=http%3A%2F%2Fwww.example.org%2Fservice&renew=true
```

使用 POST 响应而不是重定向：

```
https://cas.example.org/cas/login?method=POST&service=http%3A%2F%2Fwww.example.org%2Fservice
```

### 2.1.3. 用户名/密码认证响应

当`/login`作为凭据请求者时，响应将根据它请求的凭据类型而有所不同。在大多数情况下，CAS 将通过显示一个要求用户名和密码的登录屏幕来响应。该页面必须包含一个带有参数【用户名】、【密码】和【lt】的表单。该表单还可以包含参数【warn】。如果`service`指定为`/login`，则 `service`还必须是表单的参数，包含最初传递给 的值`/login`。这些参数在第[2.2.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.1)节中详细讨论。表单必须通过 HTTP POST 方法提交`/login`，然后将充当凭证接受器，在第[2.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2)节中讨论。

### 2.1.4. 信任认证响应

信任身份验证考虑到请求的任意方面作为身份验证的基础。考虑到本地策略和实施的特定身份验证机制的后勤，信任身份验证的适当用户体验将是高度特定于部署者的。

当`/login`作为信任身份验证的凭证请求者时，其行为将取决于它将接收的凭证类型。如果凭据有效，CAS 可以透明地将用户重定向到服务。或者，CAS 可能会显示一个警告，提示已提供凭据，并允许客户端确认它想要使用这些凭据。建议 CAS 实现允许部署者选择首选行为。如果凭据无效或不存在，建议 CAS 向客户端显示身份验证失败的原因，并可能向用户提供其他身份验证方式（例如用户名/密码身份验证）。

### 2.1.5. 单点登录身份验证的响应

如果客户端已经与 CAS 建立了单点登录会话，则客户端将向其提供其 HTTP 会话 cookie，`/login`并且将按照第[2.2.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.4)节中的方式处理行为。但是，如果`renew`设置了该参数，则将按照第[2.1.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.1.3)或 [2.1.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.1.4)节中的方式处理行为。

## 2.2. /login 作为【凭证接受者】

当一组接受的凭证被传递给 `/login`时，`/login`充当凭证接受者，其行为在本节中定义。

### 2.2.1. 所有类型的身份验证通用的参数

`/login`当它充当凭证接受器时，可以传递以下 HTTP 请求参数。它们都区分大小写，并且都必须由`/login`.

`service`[可选] - 客户端尝试访问的应用程序的 URL。作为 HTTP 请求参数，此 URL 值必须按照 RFC 1738 [ [4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4) ] 的第 2.2 节中的描述进行 URL 编码。验证成功后，CAS 必须将客户端重定向到此 URL。这在第[2.2.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.4)节中详细讨论。如果 CAS 服务器在非开放模式下运行（允许使用 CAS 服务器的服务 URL 在 CAS 服务器中注册），则 CAS 服务器必须拒绝操作并在出现非授权服务 URL 时打印出有意义的消息。

### 2.2.2. 用户名/密码认证参数

除了第[2.2.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.1)节中指定的 OPTIONAL 参数外，以下 HTTP 请求参数必须`/login`在它充当用户名/密码身份验证的凭据接受器时传递给。它们都区分大小写。

- `username` [REQUIRED] - 尝试登录的客户端的用户名
- `password` [必需] - 尝试登录的客户端的密码
- `lt`[可选] - 登录票。这是作为第[2.1.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.1.3)节中讨论的登录表单的一部分提供的。登录票本身在第[3.5](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.5)节中讨论。
- `rememberMe`[可选，CAS 3.0] - 如果设置了此参数，则 CAS 服务器可能会创建长期票证授予票证（称为“记住我”支持）。是否支持 Long-Term Ticket Granting Tickets 取决于 CAS 服务器配置。

> 注意：当 CAS 服务器支持长期票证授予票证（记住我）时，必须考虑安全因素。例如，这包括共享计算机的使用。在 CAS 客户端系统上，可能需要处理不同的记住我登录。有关详细信息，请参阅第[4.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head4.1)节 。

### **2.2.3. 信任认证参数**

没有用于信任身份验证的必需的 HTTP 请求参数。信任认证可以基于 HTTP 请求的任何方面。

### **2.2.4. 响应**

`/login`当它作为凭证接受者运行时，必须提供以下响应之一。

- 成功登录：`service` 以不会导致客户端凭据被转发到`service`. 此重定向必须导致客户端向`service`. 请求必须包含有效的服务票据，作为 HTTP 请求参数“票据”传递。有关详细信息，请参阅[附录 B。](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head_appdx_b)如果`service`未指定，CAS 必须显示一条消息，通知客户端它已成功启动单点登录会话。
- 登录失败：`/login`作为凭证请求者返回。在这种情况下，建议 CAS 服务器向用户显示一条错误消息，说明登录失败的原因（例如密码错误、帐户锁定等），并在适当的情况下为用户提供尝试再次登录的机会。

## **2.3. /logout**

`/logout`破坏客户端的单点登录 CAS 会话。票证授予 cookie（第[3.6](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.6)节）被销毁，后续请求`/login`将不会获得服务票证，直到用户再次出示主要凭据（从而建立新的单点登录会话）。

### **2.3.1. 参数**

以下 HTTP 请求参数可以指定为`/logout`. 它区分大小写，应该由`/logout`.

- `service`[可选，CAS 3.0] - 如果`service`指定了参数，则`service` 在 CAS 服务器执行注销后，浏览器可能会自动重定向到 指定的 URL 。CAS Server 是否实际执行重定向取决于服务器配置。作为 HTTP 请求参数，该`service`值必须是 URL 编码的，如 RFC 1738 [ [4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4) ] 的第 2.2 节所述。

  > 注意：强烈建议`service`通过服务管理工具过滤所有url，这样只有经过授权和已知的客户端应用程序才能使用 CAS 服务器。保持服务管理工具开放以允许对所有应用程序的宽松访问可能会增加服务攻击和其他安全漏洞的风险。此外，建议仅`https`允许客户端应用程序使用安全协议，以进一步加强对客户端的验证。

  > 注意：`url`以前 CAS 2.0 规范中定义的参数不再是 CAS 3.0 中的有效参数。CAS 服务器必须忽略给定的 `url`参数。CAS 客户端可以提供`service`如上所述的参数，因为这可以确保在非开放模式下操作时，根据注册的服务 URL 验证参数。详见[2.3.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.3.2)。

### 2.3.2. 响应

[CAS 1.0, CAS 2.0]`/logout`必须显示一个页面，说明用户已注销。如果实现了“url”请求参数，则`/logout`还应该提供指向所提供 URL 的链接，如第[2.3.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.3.1)节所述。

[CAS 3.0]`/logout`如果没有`service`提供参数，则必须显示一个页面，说明用户已注销。如果`service`提供了带有编码 URL 值的请求参数，则 CAS 服务器会在成功注销后重定向到给定的服务 URL。

> 注意：当 CAS 服务器在非开放模式下运行时（在 CAS 服务器中注册了允许的服务 URL），CAS 服务器必须确保只接受注册的 [service] 参数服务 URL 进行重定向。该`url`前CAS 2.0规范定义的参数是不是在CAS 3.0有效的参数了。CAS 服务器必须忽略给定的 `url`参数。

### **2.3.3 单点退出**

CAS 服务器可以支持单点注销 (SLO)。SLO 意味着用户不仅会从 CAS 服务器注销，还会从所有访问过的 CAS 客户端应用程序注销。**如果 CAS 服务器支持 SLO，则每当用户明确的票据授予票据过期时，CAS 服务器必须向在此 CAS 会话期间提供给 CAS 的所有服务 URL发送包含注销 XML 文档（参见[附录 C](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head_appdx_c)）的 HTTP POST 请求（例如在注销期间）**。不支持 SLO POST 请求的 CAS 客户端必须忽略这些请求。**SLO 请求也可以在 TGT 空闲超时时由 CAS 服务器发起**。

#### **2.3.3.1 服务器行为**

CAS 服务器应忽略对 CAS 客户端应用程序服务 URL 的 Single Logout POST 请求可能发生的所有错误。这确保发送 POST 请求时的任何错误都不会干扰 CAS 服务器的性能和可用性（“即发即弃”）。

#### **2.3.3.2 客户端行为**

处理注销 POST 请求数据取决于 CAS 客户端。建议从 SLO POST 请求中发送的服务票证 ID 标识的应用程序中注销用户。如果客户端支持 SLO POST 请求处理，则客户端应返回 HTTP 成功状态代码。

## **2.4. /validate [CAS 1.0]**

`/validate`检查服务票证的有效性。`/validate`是 CAS 1.0 协议的一部分，因此不处理代理身份验证。当代理票被传递给CAS时，CAS 必须以票证验证失败响应进行响应 `/validate`。

### **2.4.1. 参数**

以下 HTTP 请求参数可以指定为`/validate`. 它们区分大小写并且必须全部由`/validate`.

- `service`[REQUIRED] - 为其签发票证的服务的标识符，如第 2.2.1 节所述。作为 HTTP 请求参数，该`service`值必须是 URL 编码的，如 RFC 1738 [ [4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4) ] 的第 2.2 节所述。

  > 注意：强烈建议`service`通过服务管理工具过滤所有url，这样只有经过授权和已知的客户端应用程序才能使用 CAS 服务器。保持服务管理工具开放以允许对所有应用程序的宽松访问可能会增加服务攻击和其他安全漏洞的风险。此外，建议仅`https`允许客户端应用程序使用安全协议，以进一步加强对客户端的验证。

- `ticket`[REQUIRED] - 由`/login`. 服务票据在 3.1 节中描述。

- `renew`[可选] - 如果设置了此参数，则仅当服务票证是从用户的主要凭据的呈现中发出时，票证验证才会成功。如果票证是从单点登录会话发出的，它将失败。

### **2.4.2. 响应**

`/validate` 将返回以下两个响应之一：

票证验证成功：

<LF>

票证验证失败时：

没有<LF>



### **2.4.3. /validate 的 URL 示例**

简单的验证尝试：

```
https://cas.example.org/cas/validate?service=http%3A%2F%2Fwww.example.org%2Fservice&ticket=ST-1856339-aA5Yuvrxzpv8Tau1cYQ7
```

确保通过提供主要凭据来发出服务票证：

```
https://cas.example.org/cas/validate?service=http%3A%2F%2Fwww.example.org%2Fservice&ticket=ST-1856339-aA5Yuvrxzpv8Tau1cYQ7&renew=true
```



## **2.5. /serviceValidate [CAS 2.0]**

`/serviceValidate`检查服务票证的有效性并返回 XML 片段响应。 `/serviceValidate`还必须在请求时生成和发出代理授权票。 `/serviceValidate`如果收到代理票证，则不得返回成功的身份验证。建议如果`/serviceValidate`收到代理票证，XML 响应中的错误消息应该解释验证失败，因为代理票证已传递给`/serviceValidate`。



### **2.5.1. 参数**

以下 HTTP 请求参数可以指定为`/serviceValidate`. 它们区分大小写并且必须全部由`/serviceValidate`.

- **`service`[必需]** - 为其签发票证的服务的标识符，如第[2.2.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.1)节所述。作为 HTTP 请求参数，该`service`值必须是 URL 编码的，如 RFC 1738 [ [4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4) ] 的第 2.2 节所述。

  > 注意：强烈建议`service`通过服务管理工具过滤所有url，这样只有经过授权和已知的客户端应用程序才能使用 CAS 服务器。保持服务管理工具开放以允许对所有应用程序的宽松访问可能会增加服务攻击和其他安全漏洞的风险。此外，建议仅`https`允许客户端应用程序使用安全协议，以进一步加强对客户端的验证。

- **`ticket`[REQUIRED]** - 由`/login`. 服务票据在第[3.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.1)节中描述。

- `pgtUrl`[可选] - 代理回调的 URL。在第[2.5.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.4)节中讨论 。作为 HTTP 请求参数，“pgtUrl”值必须是 URL 编码的，如 RFC 1738 [ [4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4) ] 的第 2.2 节所述。

- `renew`[可选] - 如果设置了此参数，则仅当服务票证是从用户的主要凭据的呈现中发出时，票证验证才会成功。如果票证是从单点登录会话发出的，它将失败。

- `format`[可选] - 如果设置了此参数，则必须根据参数值生成票证验证响应。支持的值为`XML` 和`JSON`。如果未设置此参数，`XML`将使用默认格式。如果 CAS 服务器不支持该参数值，则必须返回错误代码，如第[2.5.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.3)节所述。



### **2.5.2. 回复**

`/serviceValidate` 将返回一个 XML 格式的 CAS 服务响应，如附录 A 中的 XML 模式所述。以下是示例响应：

**票证验证成功：**

```xml
<cas:serviceResponse xmlns:cas="http://www.yale.edu/tp/cas">
 <cas:authenticationSuccess>
  <cas:user>username</cas:user>
  <cas:proxyGrantingTicket>PGTIOU-84678-8a9d...</cas:proxyGrantingTicket>
 </cas:authenticationSuccess>
</cas:serviceResponse>
```

```json
{
  "serviceResponse" : {
    "authenticationSuccess" : {
      "user" : "username",
      "proxyGrantingTicket" : "PGTIOU-84678-8a9d..."
    }
  }
}
```

**票证验证失败时：**

```xml
<cas:serviceResponse xmlns:cas="http://www.yale.edu/tp/cas">
 <cas:authenticationFailure code="INVALID_TICKET">
    Ticket ST-1856339-aA5Yuvrxzpv8Tau1cYQ7 not recognized
  </cas:authenticationFailure>
</cas:serviceResponse>
```

```json
{
  "serviceResponse" : {
    "authenticationFailure" : {
      "code" : "INVALID_TICKET",
      "description" : "Ticket ST-1856339-aA5Yuvrxzpv8Tau1cYQ7 not recognized"
    }
  }
}
```

对于代理响应，请参阅第[2.6.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.6.2)节。



### **2.5.3. 错误代码**

以下值可以用作身份验证失败响应的“代码”属性。以下是所有 CAS 服务器必须实现的最小错误代码集。实现可以包括其他。

- `INVALID_REQUEST` - 并非所有必需的请求参数都存在
- `INVALID_TICKET_SPEC` - 未能满足验证规范的要求
- `UNAUTHORIZED_SERVICE_PROXY` - 该服务无权执行代理身份验证
- `INVALID_PROXY_CALLBACK`- 指定的代理回调无效。为代理身份验证指定的凭据不符合安全要求
- `INVALID_TICKET`- 提供的票证无效，或者票证不是来自初始登录并且`renew`是在验证时设置的。`\<cas:authenticationFailure\>`XML 响应块的主体应该描述确切的细节。
- `INVALID_SERVICE`- 提供的票证有效，但指定的服务与与票证关联的服务不匹配。CAS 必须使票证无效并禁止将来验证同一张票证。
- `INTERNAL_ERROR` - 票证验证期间发生内部错误

对于所有错误代码，建议 CAS 提供更详细的消息作为`\<cas:authenticationFailure\>`XML 响应块的正文。



### **2.5.4. 代理回调**

**如果服务希望将客户端的身份验证代理到后端服务**，**则它必须获取代理授权票证 (PGT)**。此票证的获取是通过代理回调 URL 处理的。此 URL 将唯一且安全地标识代理客户端身份验证的服务。然后，后端服务可以根据代理服务识别回调 URL 来决定是否接受凭证。

代理回调机制的工作原理如下：

1. **请求代理授权票证 (PGT)** 的服务在初始服务票证或代理票证验证时指定 HTTP 请求参数“pgtUrl” `/serviceValidate`（或`/proxyValidate`）。这是 CAS 将连接到的服务的回调 URL，以验证服务的身份。这个 URL 必须是 HTTPS 并且 CAS 必须评估端点以建立对等信任。建立信任至少涉及使用 PKIX 和使用容器信任来验证回调 url 证书的签名、链和到期窗口。proxy-granting-ticket 或相应的proxy-granting-ticket IOU 的生成可能会因为代理回调url 无法满足最低安全要求，例如无法在peer 之间建立信任或端点无响应等。失败，将不会发出代理授权票，并且第[2.5.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.2)节中描述的 CAS 服务响应不得包含 `\<proxyGrantingTicket\>`堵塞。此时，代理授权票证的颁发将停止，服务票证验证将失败。否则，该过程将正常进行到步骤 2。

2. CAS使用HTTP GET请求中传递的HTTP请求参数`pgtId`和 `pgtIou`到pgtUrl端点。这些实体分别在第[3.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.3)和 [3.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.4)节中讨论。如果代理回调 url 指定了任何参数，则必须保留这些参数。CAS 还必须通过验证来自 GET 请求的响应 HTTP 状态代码来确保端点是可访问的，如步骤 #3 中所述。如果代理服务验证失败或端点以不可接受的状态代码响应，代理验证必须失败并且 CAS 必须以适当的错误代码响应，如第[2.5.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.3)节所述。

3. 如果 HTTP GET 返回 HTTP 状态代码 200（OK），则 CAS 必须使用包含块内代理授予票证 IOU（第[3.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.4)节）的服务响应（第[2.5.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.2)节）来响应（`/serviceValidate`或`/proxyValidate`）请求。如果 HTTP GET 返回任何其他状态代码，除了 HTTP 3xx 重定向，CAS 必须 用一个不能包含块的服务响应来响应（或）请求。CAS 可以遵循由. 但是，在块中验证时提供的标识回调 URL必须与最初作为 参数传递给(或) 的URL 相同。

   `\<cas:proxyGrantingTicket\>``/serviceValidate``/proxyValidate``\<cas:proxyGrantingTicket\>``pgtUrl``\<proxy\>``/serviceValidate``/proxyValidate``pgtUrl`

4. 该服务在 CAS 响应中收到一个代理授权票证 IOU，以及来自代理回调的代理授权票证和代理授权票证 IOU，将使用代理授权票证 IOU 关联代理授权票证与验证响应。然后，该服务将使用代理授权票证获取代理票证，如第[2.7](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.7)节所述。



### **2.5.5. 属性 [CAS 3.0]**

[CAS 3.0] 响应文档可以包含一个可选的 <cas:attributes> 元素，用于额外的认证和/或用户属性。有关详细信息，请参阅[附录 A。](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head_appdx_b)



### **2.5.6. /serviceValidate 的 URL 示例**

简单的验证尝试：

```
https://cas.example.org/cas/serviceValidate?service=http%3A%2F%2Fwww.example.org%2Fservice&ticket=ST-1856339-aA5Yuvrxzpv8Tau1cYQ7
```

确保通过提供主要凭据来发出服务票证：

```
https://cas.example.org/cas/serviceValidate?service=http%3A%2F%2Fwww.example.org%2Fservice&ticket=ST-1856339-aA5Yuvrxzpv8Tau1cYQ7&renew=true
```

传入一个回调 URL 进行代理：

```
https://cas.example.org/cas/serviceValidate?service=http%3A%2F%2Fwww.example.org%2Fservice&ticket=ST-1856339-aA5Yuvrxzpv8Tau1cYQ7&pgtUrl=https://www.example.org%2Fservice%2FproxyCallback
```



### **2.5.7 具有自定义属性的示例响应**

```xml
  <cas:serviceResponse xmlns:cas="http://www.yale.edu/tp/cas">
    <cas:authenticationSuccess>
      <cas:user>username</cas:user>
      <cas:attributes>
        <cas:firstname>John</cas:firstname>
        <cas:lastname>Doe</cas:lastname>
        <cas:title>Mr.</cas:title>
        <cas:email>jdoe@example.org</cas:email>
        <cas:affiliation>staff</cas:affiliation>
        <cas:affiliation>faculty</cas:affiliation>
      </cas:attributes>
      <cas:proxyGrantingTicket>PGTIOU-84678-8a9d...</cas:proxyGrantingTicket>
    </cas:authenticationSuccess>
  </cas:serviceResponse>
```



```json
{
  "serviceResponse" : {
    "authenticationSuccess" : {
      "user" : "username",
      "proxyGrantingTicket" : "PGTIOU-84678-8a9d...",
      "proxies" : [ "https://proxy1/pgtUrl", "https://proxy2/pgtUrl" ],
      "attributes" : {
        "firstName" : "John",
        "affiliation" : [ "staff", "faculty" ],
        "title" : "Mr.",
        "email" : "jdoe@example.orgmailto:jdoe@example.org",
        "lastname" : "Doe"
      }
    }
  }
}
```



## **2.6. /proxyValidate [CAS 2.0]**

`/proxyValidate`必须执行与`/serviceValidate`代理票证相同的验证任务，并另外验证代理票证。`/proxyValidate`必须能够验证服务票证和代理票证。有关详细信息，请参阅第[2.5.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.4)节 。



### **2.6.1. 参数**

`/proxyValidate`具有与`/serviceValidate` 相同的参数要求。见第[2.5.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.1)节。



### **2.6.2. 响应**

`/proxyValidate` 将返回一个 XML 格式的 CAS 服务响应，如附录 A 中的 XML 模式所述。以下是示例响应：

票证验证成功的响应：

```xml
  <cas:serviceResponse xmlns:cas="http://www.yale.edu/tp/cas"> 
    <cas:authenticationSuccess>
      <cas:user>username</cas:user>
      <cas:proxyGrantingTicket>PGTIOU-84678-8a9d...</cas:proxyGrantingTicket>
      <cas:proxies>
        <cas:proxy>https://proxy2/pgtUrl</cas:proxy>
        <cas:proxy>https://proxy1/pgtUrl</cas:proxy>
      </cas:proxies>
    </cas:authenticationSuccess> 
  </cas:serviceResponse>
```

```json
{
  "serviceResponse" : {
    "authenticationSuccess" : {
      "user" : "username",
      "proxyGrantingTicket" : "PGTIOU-84678-8a9d...",
      "proxies" : [ "https://proxy1/pgtUrl", "https://proxy2/pgtUrl" ]
    }
  }
}
```

> 注意：当身份验证通过多个代理进行时，代理遍历的顺序必须反映在 <cas:proxies> 块中。最近访问的代理必须是列出的第一个代理，并且所有其他代理必须在添加新代理时下移。在上面的示例中，首先访问了由 <https://proxy1/pgtUrl> 标识的服务，并且该服务代理对由 <https://proxy2/pgtUrl> 标识的服务进行身份验证。

票证验证失败的响应：

```
  <cas:serviceResponse xmlns:cas='http://www.yale.edu/tp/cas'>
      <cas:authenticationFailure code="INVALID_TICKET">
         ticket PT-1856376-1HMgO86Z2ZKeByc5XdYD not recognized
      </cas:authenticationFailure>
  </cas:serviceResponse>
```



### **2.6.3 错误代码**

见第[2.5.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.3)节



### **2.6.4 /proxyValidate 的 URL 示例**

`/proxyValidate`接受与`/serviceValidate`相同的参数。有关使用示例，请参见第[2.5.5](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.5)节 ，将“serviceValidate”替换为“proxyValidate”。



## **2.7. /proxy [CAS 2.0]**

`/proxy` 为已获得代理授权票证的服务提供代理票证，并将代理身份验证到后端服务。



### **2.7.1. 参数**

必须为 指定以下 HTTP 请求参数`/proxy`。它们都区分大小写。

- `pgt` [必需] - 服务在服务票证或代理票证验证期间获取的代理授权票证。
- `targetService`[REQUIRED] - 后端服务的服务标识符。请注意，并非所有后端服务都是 Web 服务，因此此服务标识符并不总是一个 URL。但是，此处指定的服务标识符必须与代理票证验证时`service`指定的参数匹配`/proxyValidate`。



### **2.7.2. 响应**

`/proxy`将返回一个 XML 格式的 CAS serviceResponse 文档，如[附录 A](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head_appdx_a)中的 XML 模式所述。以下是示例响应：

请求成功的响应：

```xml
  <cas:serviceResponse xmlns:cas="http://www.yale.edu/tp/cas">
      <cas:proxySuccess>
          <cas:proxyTicket>PT-1856392-b98xZrQN4p90ASrw96c8</cas:proxyTicket>
      </cas:proxySuccess>
  </cas:serviceResponse>
```

请求失败的响应：

```xml
<cas:serviceResponse xmlns:cas="http://www.yale.edu/tp/cas">
      <cas:proxyFailure code="INVALID_REQUEST">
          'pgt' and 'targetService' parameters are both required
      </cas:proxyFailure>
  </cas:serviceResponse>
```



### **2.7.3. 错误代码**

以下值可以用作`code`身份验证失败响应的属性。以下是所有 CAS 服务器必须实现的最小错误代码集。实现可以包括其他。

- `INVALID_REQUEST` - 并非所有必需的请求参数都存在
- `UNAUTHORIZED_SERVICE` - 服务未被授权执行代理请求
- `INTERNAL_ERROR` - 票证验证期间发生内部错误

对于所有错误代码，建议 CAS 提供更详细的消息作为 XML 响应的 <cas:authenticationFailure> 块的正文。



### **2.7.4. /proxy 的 URL 示例**

简单的代理请求：

```
https://server/cas/proxy?targetService=http%3A%2F%2Fwww.service.com&pgt=PGT-490649-W81Y9Sa2vTM7hda7xNTkezTbVge4CUsybAr
```



### **2.7.5 服务票据生命周期影响**

CAS 服务器实现可以在生成代理票证时更新父服务票证 (ST) 生命周期。

## **2.8. /p3/serviceValidate [CAS 3.0]**

`/p3/serviceValidate`必须执行与`/serviceValidate`CAS 响应相同的验证任务，**并在 CAS 响应中额外返回用户属性**。有关详细信息，请参阅第[2.5](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5)节和第[2.5.7](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.7)节。



### **2.8.1. 参数**

`/p3/serviceValidate`具有与 `/serviceValidate`相同的参数要求。见第[2.5.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.1)节。

## **2.9. /p3/proxyValidate [CAS 3.0]**

`/p3/proxyValidate`必须执行与`/p3/serviceValidate`代理票证相同的验证任务，并另外验证代理票证。见第[2.8](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5)节。



### **2.9.1. 参数**

`/p3/proxyValidate`具有与 `/p3/serviceValidate`相同的参数要求。见第[2.8.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.8.1)节。



# **3. CAS 实体**



## **3.1. 服务票据**

**服务票据是一个不透明的字符串，客户端将其用作获取服务访问权限的凭据**。如`/login`第[2.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2)节所述，根据【客户端的凭据】和【服务标识符】从 CAS 获取服务票据。



### **3.1.1. 服务票属性**

- 服务票据仅对`/login`生成时指定的服务标识符有效。服务标识符不应该是服务票据的一部分。
- **服务票据必须仅对一次票据验证尝试有效。无论验证是否成功，CAS 都必须使票无效，从而导致该票的所有未来验证尝试都失败。**
- **CAS 应该在未验证的服务票据发出后的合理时间内使它们过期**。如果一个服务提供了一个过期的服务票来验证，CAS 必须用一个验证失败响应来响应。
- 建议验证响应包含一条说明性消息，解释验证失败的原因。
- 建议服务票在过期前的有效时间不超过五分钟。本地安全和 CAS 使用考虑可能会决定未验证服务票的最佳寿命。
- 服务票据必须包含足够的安全随机数据，以便票据是不可猜测的。
- **服务票证必须以字符`ST-`.开头。**
- 服务必须能够接受长达 32 个字符的服务票证。建议服务支持最长 256 个字符的服务票证。



## **3.2. 代理票**

**代理票证是一个不透明的字符串，服务将其用作凭证来代表客户端获取对后端服务的访问权限。代理票是在服务提供有效的代理授权票（第[3.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.3)节）以及它所连接的后端服务的服务标识符后从 CAS 获得的 。**



### **3.2.1. 代理票属性**

- 代理票仅对`/proxy` 生成时指定的服务标识符有效。服务标识符不应该是代理票的一部分。
- 代理票必须仅对一次票验证尝试有效。无论验证是否成功，CAS 都必须使票无效，从而导致该票的所有未来验证尝试都失败。
- CAS 应该在未验证的代理票发出后的合理时间内到期。如果一个服务提供一个过期的代理票证以供验证，CAS 必须以一个验证失败响应来响应。
- 建议验证响应包含一条说明性消息，解释验证失败的原因。
- 建议代理票在过期前的有效时间不超过五分钟。本地安全和 CAS 使用考虑可能会决定未验证代理票的最佳寿命。
- 代理票必须包含足够的安全随机数据，以便票是不可猜测的。
- 代理票应该以字符开头，`PT-`。
- 后端服务必须能够接受长达 32 个字符的代理票证。建议后端服务支持最长 256 个字符的代理票证。



## **3.3. 代理授权票**

代理授权票证 (PGT) 是一个不透明的字符串，服务使用它来获取代理票证以代表客户端获取对后端服务的访问权。在验证服务票证或代理票证后，从 CAS 获得代理授权票证。第[2.5.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.4)节完整描述了代理授予票证的发行 。



### **3.3.1. 代理授予票证属性**

- 服务可以使用代理授权票证来获取多个代理票证。代理授权票不是一次性使用票。
- 当其身份验证被代理的客户端退出 CAS 时，代理授予票必须过期。
- 代理授权票必须包含足够的安全随机数据，以便在合理的时间内无法通过蛮力攻击猜出票。
- 代理授权票应该以字符开头`PGT-`。
- 服务必须能够处理长达 64 个字符的代理授权票证。
- 建议服务支持最长 256 个字符的代理授予票证。



## **3.4. 代理授权票借据**

代理授权票据 IOU 是一个不透明的字符串，放置在由其提供的响应中`/serviceValidate`，`/proxyValidate`用于将服务票据或代理票据验证与特定的代理授权票据相关联。有关此过程的完整说明，请参阅第[2.5.4](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.4)节。



### **3.4.1. 代理授予票据 IOU 属性**

- 代理授予票据 IOU 不应包含对其相关代理授予票据的任何引用。给定一个特定的 PGTIOU，一定不可能在合理的时间段内通过算法方法推导出其对应的 PGT。
- 代理授予票据 IOU 必须包含足够的安全随机数据，以便在合理的时间内无法通过蛮力攻击猜出票据。
- 授予代理权的票据 IOU 应该以字符`PGTIOU-`.
- 服务必须能够处理长达 64 个字符的 PGTIOU。建议服务支持最多 256 个字符的 PGTIOU。



## **3.5. 登录票**

登录票是一个**可选**字符串，由`/login`【凭据请求者】提供，并`/login`作为凭据接受者传递给用户名/密码身份验证。其目的是防止由于 Web 浏览器中的错误而重放凭据。



### **3.5.1. 登录票属性**

- 由发行的登录票`/login`必须在概率上是唯一的。
- 登录票必须仅对一次身份验证尝试有效。无论身份验证是否成功，CAS 都必须使登录票证无效，从而导致所有未来对该登录票证实例的身份验证尝试失败。
- **登录票应该以字符开头`LT-`。**



## **3.6. 票证cookie**

**授予票证的 cookie 是CAS 在建立单点登录会话时设置的 HTTP cookie[ [5](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#5) ]。此 cookie 为客户端维护登录状态，并且当它有效时，客户端可以将其提供给 CAS 以代替主要凭据。服务可以选择单点登录通过出`renew` 在章节参数说明[2.1.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.1.1)，[2.4.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.4.1)和[2.5.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5.1)。**



### **3.6.1. 授予票证的 cookie 属性**

- 如果相应 TGT的长期支持未处于活动状态（[4.1.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head4.1.1)），则授予票证的 cookie 应设置为在客户端浏览器会话结束时过期。
- **CAS 应尽可能限制 cookie 路径。例如，如果 CAS 服务器设置在路径 /cas 下，则 cookie 路径应设置为 /cas。**
- 票证 cookie 的值应包含足够的安全随机数据，以便票证 cookie 在合理的时间段内是不可猜测的。
- **授予票证的 cookie 的名称应该以字符开头`TGC-`。**
- **票据授予 cookie 的值应该遵循与票据授予票据相同的规则。通常，票据授予 cookie 的值可以包含票据授予票据本身作为经过身份验证的单点登录会话的表示。**



## **3.7. 票证和票证授予 cookie 字符集**

**除上述要求外，所有 CAS 票证和票证授予 cookie 的值必须仅包含集合中`{A-Z, a-z, 0-9}`的字符 和连字符`-`。**



## **3.8. 出票票**

票证授予票证 (TGT) 是由 CAS 服务器生成的不透明字符串，在`/login`. **此票证可能与代表单点登录会话状态的票证授予 cookie 相关联，具有有效期，并充当颁发服务票证、代理授予票证等的基础和基线。**



### **3.8.1. 授予票证的票证属性**

- 服务可以使用授予票证的票证来获取多个服务票证。授予门票的门票不是一次性使用的门票，并且与有效期和到期政策相关联。
- 当其身份验证被管理的客户端退出 CAS 时，授予票证的票证必须过期。
- 授予票证的票证必须包含足够的安全随机数据，以便在合理的时间内无法通过蛮力攻击猜出票证。
- 授予门票的门票应该以字符开头`TGT-`。
- 建议在与其他外部资源共享时对授予票据的票据进行加密，以最大限度地减少安全漏洞，因为它们与票据授予 cookie 相关联并代表身份验证会话。



# **4. 可选功能**



## **4.1 长期票 - 记住我 [CAS 3.0]**

CAS 服务器可以支持长期票证授予票证（称为“记住我”功能）。如果 CAS Server 支持此功能，只要 CAS Server 中的 Long-Term Ticket Granting Ticket 未过期且浏览器的 TGC Cookie 有效，就可以对 CAS Server 执行重复性、非交互式重新登录.



### **4.1.1 启用记住我（登录页面）**

- CAS 服务器必须在登录页面上提供一个复选框以允许记住我的功能。
- 默认情况下，必须取消选中该复选框。
- 必须由用户选择是否为登录启用“记住我”。见第[2.2.2](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.2)节。



### **4.1.2 安全影响**

启用记住我可能有安全隐患。由于 CAS 身份验证绑定到浏览器，并且当存在有效的长期 TGT 票证且浏览器提供的 CAS cookie 有效时，用户不会以交互方式登录，因此必须在 CAS 客户端特别注意正确处理记住我登录。必须由 CAS 客户端负责决定是否以及何时可以特殊处理 Remember-Me CAS 登录。见[4.1.3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head4.1.3)。



### **4.1.3 CAS 验证响应属性**

由于只有 CAS 客户端必须决定如何处理记住我登录（见 [4.2.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head4.2.1)），CAS 服务器必须向 CAS 客户端提供关于记住我登录的信息。必须由CAS服务器（见第支持的所有验票方法来提供这个信息 [2.5](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.5)，[2.6](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.6)和[2.8](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.8)在这种情况下）。

- 在 serviceValidate XML 响应中（见[附录 A](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head_appdx_a)），Remember-Me 登录必须由`longTermAuthenticationRequestTokenUsed`属性指示 。此外，该 `isFromNewLogin`属性可以用于决定这是否具有安全隐患。
- 在 SAML 验证响应中，Remember-Me 必须由`longTermAuthenticationRequestTokenUsed`属性指示 。



### **4.1.4 CAS 客户端要求**

如果 CAS 客户端需要处理特殊的记住我登录（例如，在记住登录时拒绝访问 CAS 客户端应用程序的敏感区域），则 CAS 客户端`/validate`不得使用CAS 验证 URL，因为此 URL 不支持 CAS 属性验证响应文件。

### **4.1.5 长期票据授予 cookie 属性**

当 CAS 服务器创建长期 TGT 时，授予票据的 cookie 不得在[3.6.1 中](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head3.6.1)定义的客户端浏览器会话结束时过期。相反，票证授予 cookie 应在定义的长期 TGT 票证生存期到期。

Long-Term Ticket Granting Tickets 的生命周期价值定义由 CAS Server 实现者决定。长期票证有效期不得超过 3 个月。



## **4.2 /samlValidate [CAS 3.0]**

`/samlValidate`通过 HTTP POST 提供的 SAML 1.1 请求文档检查服务票证的有效性。必须返回SAML（安全访问标记语言）[7](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#7) 1.1 响应文档。这允许发布经过身份验证的 NetID 的附加信息（属性）。安全断言标记语言 (SAML) 描述了一个文档和协议框架，通过该框架可以交换安全断言（例如关于先前身份验证行为的断言）。



### **4.2.1 参数**

必须为 指定以下 HTTP 请求参数`/samlValidate`。它们都区分大小写。

- `TARGET`[REQUIRED] - 后端服务的 URL 编码服务标识符。请注意，作为 HTTP 请求参数，此 URL 值必须是 URL 编码的，如 RFC 1738 [[4\] 的](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#4)第 2.2 节所述。此处指定的服务标识符必须与`service`提供给的参数匹配`/login`。见第[2.1.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.1.1)节。该`TARGET`服务应使用 HTTPS。SAML 属性不得发布到非 SSL 站点。



### **4.2.2 HTTP 请求方法和正文**

对 /samlValidate 的请求必须是 HTTP POST 请求。请求正文必须是文档类型为“text/xml”的有效 SAML 1.0 或 1.1 请求 XML 文档。



### **4.2.3 SAML 请求值**

- `RequestID` [REQUIRED] - 请求的唯一标识符
- `IssueInstant` [必需] - 请求的时间戳
- `samlp:AssertionArtifact`[必需] - 在登录时作为响应参数获得的有效 CAS 服务票证。见第[2.2.4 节](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head2.2.4)。



### **4.2.4 /samlValidate POST 请求示例**

```
POST /cas/samlValidate?TARGET=
Host: cas.example.com
Content-Length: 491
Content-Type: text/xml 
```



```
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
	<SOAP-ENV:Header/>
	<SOAP-ENV:Body>
		<samlp:Request xmlns:samlp="urn:oasis:names:tc:SAML:1.0:protocol" MajorVersion="1" MinorVersion="1" RequestID="_192.168.16.51.1024506224022" IssueInstant="2002-06-19T17:03:44.022Z">
			<samlp:AssertionArtifact>ST-1-u4hrm3td92cLxpCvrjylcas.example.com</samlp:AssertionArtifact>
		</samlp:Request>
	</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

### **4.2.5 SAML 响应**

CAS 服务器对`/samlValidate`请求的响应。必须是 SAML 1.1 响应。

SAML 1.1 验证响应示例：

```xml
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <SOAP-ENV:Header />
  <SOAP-ENV:Body>
    <Response xmlns="urn:oasis:names:tc:SAML:1.0:protocol" xmlns:saml="urn:oasis:names:tc:SAML:1.0:assertion"
    xmlns:samlp="urn:oasis:names:tc:SAML:1.0:protocol" xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" IssueInstant="2008-12-10T14:12:14.817Z"
    MajorVersion="1" MinorVersion="1" Recipient="https://eiger.iad.vt.edu/dat/home.do"
    ResponseID="_5c94b5431c540365e5a70b2874b75996">
      <Status>
        <StatusCode Value="samlp:Success">
        </StatusCode>
      </Status>
      <Assertion xmlns="urn:oasis:names:tc:SAML:1.0:assertion" AssertionID="_e5c23ff7a3889e12fa01802a47331653"
      IssueInstant="2008-12-10T14:12:14.817Z" Issuer="localhost" MajorVersion="1"
      MinorVersion="1">
        <Conditions NotBefore="2008-12-10T14:12:14.817Z" NotOnOrAfter="2008-12-10T14:12:44.817Z">
          <AudienceRestrictionCondition>
            <Audience>
              https://some-service.example.com/app/
            </Audience>
          </AudienceRestrictionCondition>
        </Conditions>
        <AttributeStatement>
          <Subject>
            <NameIdentifier>johnq</NameIdentifier>
            <SubjectConfirmation>
              <ConfirmationMethod>
                urn:oasis:names:tc:SAML:1.0:cm:artifact
              </ConfirmationMethod>
            </SubjectConfirmation>
          </Subject>
          <Attribute AttributeName="uid" AttributeNamespace="http://www.ja-sig.org/products/cas/">
            <AttributeValue>12345</AttributeValue>
          </Attribute>
          <Attribute AttributeName="groupMembership" AttributeNamespace="http://www.ja-sig.org/products/cas/">
            <AttributeValue>
              uugid=middleware.staff,ou=Groups,dc=vt,dc=edu
            </AttributeValue>
          </Attribute>
          <Attribute AttributeName="eduPersonAffiliation" AttributeNamespace="http://www.ja-sig.org/products/cas/">
            <AttributeValue>staff</AttributeValue>
          </Attribute>
          <Attribute AttributeName="accountState" AttributeNamespace="http://www.ja-sig.org/products/cas/">
            <AttributeValue>ACTIVE</AttributeValue>
          </Attribute>
        </AttributeStatement>
        <AuthenticationStatement AuthenticationInstant="2008-12-10T14:12:14.741Z"
        AuthenticationMethod="urn:oasis:names:tc:SAML:1.0:am:password">
          <Subject>
            <NameIdentifier>johnq</NameIdentifier>
            <SubjectConfirmation>
              <ConfirmationMethod>
                urn:oasis:names:tc:SAML:1.0:cm:artifact
              </ConfirmationMethod>
            </SubjectConfirmation>
          </Subject>
        </AuthenticationStatement>
      </Assertion>
    </Response>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```



### **4.2.5.1 SAML CAS 响应属性**

SAML 响应中可能会提供以下附加属性：

- `longTermAuthenticationRequestTokenUsed`- 如果 CAS 服务器支持长期票证授予票证（记住我）（请参阅第[4.1](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#head4.1)节），则 SAML 响应必须包含此属性以指示对 CAS 客户端的记住登录。



# **附录 A：CAS 响应 XML 模式**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:cas="http://www.yale.edu/tp/cas" targetNamespace="http://www.yale.edu/tp/cas" elementFormDefault="qualified" attributeFormDefault="unqualified">
    <xs:annotation>
        <xs:documentation>The following is the schema for the Central Authentication Service (CAS) version 3.0 protocol response. This covers the responses for the following servlets: /serviceValidate, /proxyValidate, /p3/serviceValidate, /p3/proxyValidate, /proxy This specification is subject to change.</xs:documentation>
    </xs:annotation>
    <xs:element name="serviceResponse" type="cas:ServiceResponseType"></xs:element>
    <xs:complexType name="ServiceResponseType">
        <xs:choice>
            <xs:element name="authenticationSuccess" type="cas:AuthenticationSuccessType"></xs:element>
            <xs:element name="authenticationFailure" type="cas:AuthenticationFailureType"></xs:element>
            <xs:element name="proxySuccess" type="cas:ProxySuccessType"></xs:element>
            <xs:element name="proxyFailure" type="cas:ProxyFailureType"></xs:element>
        </xs:choice>
    </xs:complexType>
    <xs:complexType name="AuthenticationSuccessType">
        <xs:sequence>
            <xs:element name="user" type="xs:string"></xs:element>
            <xs:element name="attributes" type="cas:AttributesType" minOccurs="0"></xs:element>
            <xs:element name="proxyGrantingTicket" type="xs:string" minOccurs="0"></xs:element>
            <xs:element name="proxies" type="cas:ProxiesType" minOccurs="0"></xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="ProxiesType">
        <xs:sequence>
            <xs:element name="proxy" type="xs:string" maxOccurs="unbounded"></xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="AuthenticationFailureType">
        <xs:simpleContent>
            <xs:extension base="xs:string">
                <xs:attribute name="code" type="xs:string" use="required"></xs:attribute>
            </xs:extension>
        </xs:simpleContent>
    </xs:complexType>
    <xs:complexType name="ProxySuccessType">
        <xs:sequence>
            <xs:element name="proxyTicket" type="xs:string"></xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="ProxyFailureType">
        <xs:simpleContent>
            <xs:extension base="xs:string">
                <xs:attribute name="code" type="xs:string" use="required"></xs:attribute>
            </xs:extension>
        </xs:simpleContent>
    </xs:complexType>
    <xs:complexType name="AttributesType">
        <xs:sequence>
            <xs:element name="authenticationDate" type="xs:dateTime" minOccurs="1" maxOccurs="1"></xs:element>
            <xs:element name="longTermAuthenticationRequestTokenUsed" type="xs:boolean" minOccurs="1" maxOccurs="1">
                <xs:annotation>
                    <xs:documentation>true if a long-term (Remember-Me) token was used</xs:documentation>
                </xs:annotation>
            </xs:element>
            <xs:element name="isFromNewLogin" type="xs:boolean" minOccurs="1" maxOccurs="1">
                <xs:annotation>
                    <xs:documentation>true if this was from a new, interactive login. If login was from a non-interactive login (e.g. Remember-Me), this value is false or might be omitted.</xs:documentation>
                </xs:annotation>
            </xs:element>
            <xs:element name="memberOf" type="xs:string" minOccurs="0" maxOccurs="unbounded">
                <xs:annotation>
                    <xs:documentation>One or many elements describing the units the user is member in. E.g. LDAP format values.</xs:documentation>
                </xs:annotation>
            </xs:element>
            <xs:any minOccurs="0" maxOccurs="unbounded" processContents="lax">
                <xs:annotation>
                    <xs:documentation>Any user specific attribute elements.</xs:documentation>
                </xs:annotation>
            </xs:any>
        </xs:sequence>
    </xs:complexType>
</xs:schema>
```

> 注意：由于 CAS 服务器实现者可以扩展用户属性（请参阅 <xs:any> 架构定义），建议使用以下格式形成自定义属性：

```xml
<cas:attributes>
    ...
    <cas:[attribute-name]>VALUE</cas:[attribute-name]>
</cas:attributes>
```

> 具有自定义属性的示例响应：

```xml
<cas:attributes>
    <cas:authenticationDate>2015-11-12T09:30:10Z</cas:authenticationDate>
    <cas:longTermAuthenticationRequestTokenUsed>true</cas:longTermAuthenticationRequestTokenUsed>
    <cas:isFromNewLogin>true</cas:isFromNewLogin>
    <cas:myAttribute>myValue</cas:myAttribute>
</cas:attributes>
```



# **附录 B：安全重定向**

成功登录后，必须小心处理将客户端从 CAS 安全重定向到其最终目的地。在大多数情况下，客户端已通过 POST 请求将凭据发送到 CAS 服务器。根据此规范，CAS 服务器必须将用户转发到具有 GET 请求的应用程序。

HTTP/1.1 RFC[ [3](https://apereo.github.io/cas/4.2.x/protocol/CAS-Protocol-Specification.html#3) ] 提供了响应代码 303: See Other，它提供了所需的行为：通过 POST 请求接收数据的脚本可以通过 303 重定向，通过 GET 将浏览器转发到另一个 URL要求。但是，并非所有浏览器都正确实现了此行为。

因此，推荐的重定向方法是 JavaScript。包含`window.location.href`以下方式的页面 可以充分执行：

```xml
 <html>
    <head>
        <title>Yale Central Authentication Service</title>
        <script>
            window.location.href="https://portal.yale.edu/Login?ticket=ST-..."
mce_href="https://portal.yale.edu/Login?ticket=ST-...";`
       </script>
    </head>
    <body>
        <noscript>
            <p>CAS login successful.</p>
            <p>  Click <a xhref="https://portal.yale.edu/Login?ticket=ST-..."
mce_href="https://portal.yale.edu/Login?ticket=ST-...">here</a>
            to access the service you requested.<br />  </p>
        </noscript>
    </body>
 </html>
```

**此外，CAS 应该通过设置所有与缓存相关的标头来禁用浏览器缓存：**

- 编译指示：无缓存
- 缓存控制：无存储
- 到期：[RFC 1123[6] 日期等于或早于现在]

**登录票证的引入消除了 CAS 接受由浏览器缓存和重放的凭据的可能性**。但是，Apple 的 Safari 浏览器的早期版本包含一个错误，即通过使用“后退”按钮，Safari 可能会被迫向其尝试访问的服务提供客户端凭据。如果 CAS 检测到远程浏览器是这些早期版本的 Safari 之一，则 CAS 可以通过不自动重定向来防止这种行为。相反，CAS 应该显示一个页面，说明登录成功，并提供指向所请求服务的链接。然后客户端必须手动单击以继续。



# **附录 C：注销 XML 文档**

当 CAS 服务器支持 SLO 时，它将回调到每个在系统中注册的服务，并发送带有以下 SAML 注销请求 XML 文档的 POST 请求：

```xml
  <samlp:LogoutRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
     ID="[RANDOM ID]" Version="2.0" IssueInstant="[CURRENT DATE/TIME]">
    <saml:NameID xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
      @NOT_USED@
    </saml:NameID>
    <samlp:SessionIndex>[SESSION IDENTIFIER]</samlp:SessionIndex>
  </samlp:LogoutRequest>`
```



# **附录 D：参考文献**

[1] Bradner, S.，“在 RFC 中用于指示需求级别的关键词”，[RFC 2119](http://www.ietf.org/rfc/rfc2119.txt)，哈佛大学，1997 年 3 月。

[2] Berners-Lee, T.、Fielding, R.、Frystyk, H.，“超文本传输协议 - HTTP/1.0”，[RFC 1945](http://www.ietf.org/rfc/rfc1945.txt)，MIT/LCS，加州大学欧文分校，MIT/LCS，1996 年 5 月。

[3] Fielding, R., Gettys, J., Mogul, J., Frystyk, H., Masinter, L., Leach, P., Berners-Lee, T.，“超文本传输协议 - HTTP/1.1”，[RFC 2068](http://www.ietf.org/rfc/rfc2068.txt)、UC Irvine、Compaq/W3C、Compaq、W3C/MIT、Xerox、Microsoft、W3C/MIT，1999 年 6 月。

[4] Berners-Lee, T.、Masinter, L. 和 MaCahill, M.，“统一资源定位器 (URL)”，[RFC 1738](http://www.ietf.org/rfc/rfc1738.txt)，CERN，施乐公司，明尼苏达大学，1994 年 12 月。

[5] Kristol, D., Montulli, L.，“HTTP 状态管理机制”， [RFC 2965](http://www.ietf.org/rfc/rfc2965.txt)，贝尔实验室/朗讯科技，Epinions.com, Inc.，2000 年 10 月。

[6] Braden, R.，“互联网主机的要求 - 应用和支持”，[RFC 1123](http://www.ietf.org/rfc/rfc1123.txt)，互联网工程任务组，1989 年 10 月。

[7] OASIS SAML 1.1 标准，saml.xml.org，2009 年 12 月。

[8] Apereo [CAS Server](http://www.apereo.org/cas)参考实现



# **附录 E：CAS 许可证**

根据一项或多项贡献者许可协议授权给 Apereo。有关版权所有权的更多信息，请参阅随本作品分发的 NOTICE 文件。Apereo 根据 Apache 许可，版本 2.0（“许可”）向您许可此文件；除非遵守许可，否则您不得使用此文件。您可以在以下位置获得许可证的副本：

http://www.apache.org/licenses/LICENSE-2.0

除非适用法律要求或书面同意，否则根据许可分发的软件是按“原样”分发的，没有任何类型的明示或暗示的保证或条件。请参阅许可证以了解管理许可证下的许可和限制的特定语言。

# **附录 F：耶鲁执照**

版权所有 (c) 2000-2005 耶鲁大学。

本软件“按原样”提供，明确否认任何明示或暗示的保证，包括但不限于适销性和特定用途适用性的暗示保证。在任何情况下，耶鲁大学或其员工均不对任何直接的、间接的、偶然的、特殊的、惩戒性的或后果性的损害负责（包括但不限于替代服务、资源或数据使用费的采购成本； ; 或业务中断），无论是基于任何责任理论，无论是在合同、严格责任或侵权（包括疏忽或其他原因）中因使用本软件的 Avid这样的损害。

如果满足以下条件，则允许以源代码或二进制形式重新分发和使用本软件，无论是否进行修改：

1. 任何重新分发都必须在任何相关文档中以及（如果可行）在重新分发的软件中包含上述版权声明和免责声明以及此条件列表。
2. 任何重新分发都必须在任何相关文档中以及（如果可行）重新分发的软件中包括确认“本产品包括耶鲁大学开发的软件”。
3. “耶鲁”和“耶鲁大学”的名称不得用于认可或推广源自该软件的产品。



# **附录 F：对本文档的更改**

2005 年 5 月 4 日：v1.0 - 初始版本

2012 年 3 月 2 日：v1.0.1 - 修正了“noscropt”错字。apetro per amazurek 感谢 ASU 的 Faraz Khan 发现了错别字。

2013 年 4 月：v2.0 - CAS 3.0 协议，Apereo 版权，Apache License 2.0


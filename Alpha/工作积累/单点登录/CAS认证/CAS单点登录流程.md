# CAS单点登录loginflow登录流程

如何通过loginflow进入登录页面，如何通过loginFlowRegistry登录流程，进入并展示登录页面。解析AbstractAction
由于CAS单点登录采用的是Spring web flow框架进行设计开发的，我们先大致看webflow的架构代码。

## 1. AbstractAction【webflow基类】

首先是登录入口处理类**initialFlowSetupAction**，从这个类可以看出是继承于AbstractAction。

```java
@Component("initialFlowSetupAction")
public final class InitialFlowSetupAction extends AbstractAction {
```

下面我们来看一下AbstractAction.java这个类到底做了一些什么工作。

```java
/**org.springframework.webflow.action.AbstractAction.java*/
public abstract class AbstractAction implements Action, InitializingBean
public final Event execute(RequestContext context) throws Exception {
	Event result = doPreExecute(context);
	if (result == null) {
        //处理启动执行
		result = doExecute(context);
		doPostExecute(context);
	} else {
		if (logger.isInfoEnabled()) {
			logger.info("Action execution disallowed; pre-execution result is '" + result.getId() + "'");
		}
	}
	return result;
}
```

在此类中主要方法为**execute()，doPreExecute，doexecute，doPostExecute**这三个方法都可被子类重写掉，作用就是钩子hook。也就是说，程序先执行AbstractAction类中的execute方法后，会执行AbstractAction实现类中（子类中的实现方法）的doExecute(context)方法，后面都会调用该方法。

## 2. login-webflow.xml【登录流配置文件】

主要展示login-webflow.xml中相关的action-state的配置，省略了其他的一些跳转和判断的配置。

```xml
<var name="credential" class="org.jasig.cas.authentication.UsernamePasswordCredential"/>
<!-- <var name="credential" class="org.jasig.cas.authentication.RememberMeUsernamePasswordCredential" /> -->
<!--启动流程-->
<on-start>
    <evaluate expression="initialFlowSetupAction"/>
</on-start>

<!--登录信息绑定的对象-->
<view-state id="viewLoginForm" view="casLoginView" model="credential">
    <binder>
        <binding property="username" required="true"/>
        <binding property="password" required="true"/>
        <!--
        <binding property="rememberMe" />
        -->
    </binder>
    <on-entry>
        <set name="viewScope.commandName" value="'credential'"/>
        <!--
        <evaluate expression="samlMetadataUIParserAction" />
        -->
    </on-entry>
    <transition on="submit" bind="true" validate="true" to="realSubmit"/>
</view-state>

<action-state id="sendTicketGrantingTicket">
    <evaluate expression="sendTicketGrantingTicketAction"/>
    <transition to="serviceCheck"/>
</action-state>
```

webflow调用的相关组件配置文件中定义了认证的对象为credential以及定义了初始化执行表达式initialFlowSetupAction。
上面的配置文件会根据属性进行执行流程，但有on-start属性时，会进入此属性的对应代码中执行。然后会默认执行按顺序的第一个action-state属性中对应的代码。

## 3. initialFlowSetupAction【登录初始化组件】

InitialFlowSetupAction此类在cas-server-webapp-actions模块下的org.jasig.cas.web.flow包下。

```java
/** org.jasig.cas.web.flow.InitialFlowSetupAction.java */
@Component("initialFlowSetupAction")
public final class InitialFlowSetupAction extends AbstractAction {
	private final transient Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
	protected Event doExecute(final RequestContext context) throws Exception {
    	final HttpServletRequest request = WebUtils.getHttpServletRequest(context);
		final String contextPath = context.getExternalContext().getContextPath();
		final String cookiePath = StringUtils.isNotBlank(contextPath) ? contextPath + '/' : "/";

		if (StringUtils.isBlank(warnCookieGenerator.getCookiePath())) {
    		logger.info("Setting path for cookies for warn cookie generator to: {} ", cookiePath);
    		//设置Warningcookie的路径
    		this.warnCookieGenerator.setCookiePath(cookiePath);
		} else {
    		logger.debug("Warning cookie path is set to {} and path {}", warnCookieGenerator.getCookieDomain(),
            		warnCookieGenerator.getCookiePath());
		}
		
        if (StringUtils.isBlank(ticketGrantingTicketCookieGenerator.getCookiePath())) {
    		logger.info("Setting path for cookies for TGC cookie generator to: {} ", cookiePath);
   			//设置TGCcookie的路径
            this.ticketGrantingTicketCookieGenerator.setCookiePath(cookiePath);
		} else {
    		logger.debug("TGC cookie path is set to {} and path {}", ticketGrantingTicketCookieGenerator.getCookieDomain(),
            ticketGrantingTicketCookieGenerator.getCookiePath());
		}

		WebUtils.putTicketGrantingTicketInScopes(context,
        		this.ticketGrantingTicketCookieGenerator.retrieveCookieValue(request));

		WebUtils.putWarningCookie(context,
        		Boolean.valueOf(this.warnCookieGenerator.retrieveCookieValue(request)));
		// 由于我们是直接访问CAS单点登录服务端，因此这里获取到的service为空
		final Service service = WebUtils.getService(this.argumentExtractors, context);
        if (service != null) {
    		logger.debug("Placing service in context scope: [{}]", service.getId());
            
            // 根据请求中的service来获取CAS单点登录服务端配置的service，看是否符合//配置的要求。默认采用的配置文件在cas-server-webapp模块下的//\src\main\resources\services文件夹下。
        final RegisteredService registeredService = this.servicesManager.findServiceBy(service);

		//判断这个获取到的service是否被允许访问到CAS单点登录服务端
        if (registeredService != null && registeredService.getAccessStrategy().isServiceAccessAllowed()) {
            logger.debug("Placing registered service [{}] with id [{}] in context scope",
                    registeredService.getServiceId(),
                    registeredService.getId());
            WebUtils.putRegisteredService(context, registeredService);
            final RegisteredServiceAccessStrategy accessStrategy = registeredService.getAccessStrategy();
        if (accessStrategy.getUnauthorizedRedirectUrl() != null) {
            logger.debug("Placing registered service's unauthorized redirect url [{}] with id [{}] in context scope",
                    accessStrategy.getUnauthorizedRedirectUrl(),
                    registeredService.getServiceId());
            WebUtils.putUnauthorizedRedirectUrl(context, accessStrategy.getUnauthorizedRedirectUrl());
        }
    }
} else if (!this.enableFlowOnAbsentServiceRequest) {
    //此enableFlowOnAbsentServiceRequest开关要求一定带有service参数才//可以正常访问到cas服务端登录页面，否则报错
    logger.warn("No service authentication request is available at [{}]. CAS is configured to disable the flow.",
            WebUtils.getHttpServletRequest(context).getRequestURL());
    throw new NoSuchFlowExecutionException(context.getFlowExecutionContext().getKey(),
            new UnauthorizedServiceException("screen.service.required.message", "Service is required"));
}
	//flowScope中设置service值
	WebUtils.putService(context, service);
	return result("success");
}
```


主要功能包括设置两个cookie路径、设置两个cookie到flowScope中、校验service、将service放进上下文等。初始化完成后应执行start-state，因为没有配置start-state，所以默认执行第一个action-state。

## 4. ticketGrantingTicketCheck【执行第一个action-state】

ticketGrantingTicketCheck

```xml
<action-state id="ticketGrantingTicketCheck">
    <evaluate expression="ticketGrantingTicketCheckAction"/>
    <transition on="notExists" to="gatewayRequestCheck"/>
    <transition on="invalid" to="terminateSession"/>
    <transition on="valid" to="hasServiceCheck"/>
</action-state>
```

首先执行表达式expression=“ticketGrantingTicketCheckAction”，检查票据tgt是否存在。此时evaluate expression对应的java类就是TicketGrantingTicketCheckAction.java。

```java
/** org.jasig.cas.web.flow.TicketGrantingTicketCheckAction.java */
@Component("ticketGrantingTicketCheckAction")
public class TicketGrantingTicketCheckAction extends AbstractAction {

/**
 * Determines whether the TGT in the flow request context is valid.
 *
 * @param requestContext Flow request context.
 *
 * @throws Exception in case ticket cannot be retrieved from the service layer
 * @return {@link #NOT_EXISTS}, {@link #INVALID}, or {@link #VALID}.
 */
@Override
protected Event doExecute(final RequestContext requestContext) throws Exception {
	//通过请求获取到tgtId
    final String tgtId = WebUtils.getTicketGrantingTicketId(requestContext);
    if (!StringUtils.hasText(tgtId)) {
        return new Event(this, NOT_EXISTS);
    }

    String eventId = INVALID;
    try {
        //如果tgtId不为空，通过中心认证服务获取到这个tgtId对应的ticket，校验是//否有效
        final Ticket ticket = this.centralAuthenticationService.getTicket(tgtId, Ticket.class);
        if (ticket != null && !ticket.isExpired()) {
            eventId = VALID;
        }
    } catch (final AbstractTicketException e) {
        logger.trace("Could not retrieve ticket id {} from registry.", e);
    }
    return new Event(this,  eventId);
}

```

当不存在tgtId时返回NOT_EXISTS，否则需要通过tgtId获取ticket，再判断这个ticket是否有效，有效就返回VALID，无效返回INVALID。由于是第一次请求访问CAS服务端登录页面，本例没有TGT，所以返回NOT_EXISTS。
由于不存在TGT，那么下一步就要走transition on=“notExists” to=“gatewayRequestCheck”/>，跳转到了gatewayRequestCheck。

## 5. gatewayRequestCheck【决策状态】

通过上一节我们知道不存在TGT的时候，会跳转到gatewayRequestCheck，因此我们还是回到登录流程的xml配置中

```xml
<decision-state id="gatewayRequestCheck">
    <if test="requestParameters.gateway != '' and requestParameters.gateway != null and flowScope.service != null"
        then="gatewayServicesManagementCheck" else="serviceAuthorizationCheck"/>
</decision-state>
```


本例中我们直接请求CAS单点登录的服务端，而且通过上文的代码分析已经知道service为空，而且gateway在这样的请求中也为空，所以下一步将进入serviceAuthorizationCheck。

## 6. serviceAuthorizationCheck【行为状态】

```xml
login-webflow.xml：
<action-state id="serviceAuthorizationCheck">
	<evaluate expression="serviceAuthorizationCheck"/>
 	<transition to="initializeLogin"/>
</action-state>
```

这里要执行表达式serviceAuthorizationCheck，那么我们来看一下serviceAuthorizationCheck代码是如何实现的。ServiceAuthorizationCheck.java此类在cas-server-webapp-actions模块下的org.jasig.cas.web.flow包中。

```java
/** org.jasig.cas.web.flow.ServiceAuthorizationCheck.java */
@Component("serviceAuthorizationCheck")
public final class ServiceAuthorizationCheck extends AbstractAction {
@Override
protected Event doExecute(final RequestContext context) throws Exception {
	//获取请求的service
    final Service service = WebUtils.getService(context);
    //No service == plain /login request. Return success indicating transition to the login form
    if (service == null) {
        return success();
    }
    //判断cas单点登录服务端的services文件（一般为HTTPSandIMAPS-10000001.json）下是否配置了相关的service
    if (this.servicesManager.getAllServices().isEmpty()) {
        final String msg = String.format("No service definitions are found in the service manager. "
                + "Service [%s] will not be automatically authorized to request authentication.", service.getId());
        logger.warn(msg);
        throw new UnauthorizedServiceException(UnauthorizedServiceException.CODE_EMPTY_SVC_MGMR);
    }
    //根据请求的service的获取到HTTPSandIMAPS-10000001.json配置文件中配置的，一般都会以正则的方式进行配置，表示准入到cas服务端的service。
    final RegisteredService registeredService = this.servicesManager.findServiceBy(service);

    //判断这个获取到的service是否允许请求进入到cas单点登录服务端。
    if (registeredService == null) {
        final String msg = String.format("Service Management: Unauthorized Service Access. "
                + "Service [%s] is not found in service registry.", service.getId());
        logger.warn(msg);
        throw new UnauthorizedServiceException(UnauthorizedServiceException.CODE_UNAUTHZ_SERVICE, msg);
    }
    if (!registeredService.getAccessStrategy().isServiceAccessAllowed()) {
        final String msg = String.format("Service Management: Unauthorized Service Access. "
                + "Service [%s] is not allowed access via the service registry.", service.getId());
        
        logger.warn(msg);

        WebUtils.putUnauthorizedRedirectUrlIntoFlowScope(context,
                registeredService.getAccessStrategy().getUnauthorizedRedirectUrl());
        throw new UnauthorizedServiceException(UnauthorizedServiceException.CODE_UNAUTHZ_SERVICE, msg);
    }
    return success();
}

```

**该类的主要功能是用于检查service是否正确合法，如果没有权限，会重定向到无法认证的错误提示页面。在本例中由于是直接访问cas单点登录服务端，因此service为空，所以直接返回success，下一步执行transition to=“initializeLogin”/>。**

## 7.  initializeLogin【行为状态】

```xml
<action-state id="initializeLogin">
    <evaluate expression="'success'"/>
    <transition on="success" to="viewLoginForm"/>
</action-state>
```

**这个执行状态是非常简单，没有对应执行类。表达式的结果直接为值“success”，因此直接执行下一步transition on=“success” to=“viewLoginForm”/>**

## 8. viewLoginForm【视图状态】

接下来我们根据执行的顺序可以再看一下viewLoginForm进行了哪些操作。

```xml
<view-state id="viewLoginForm" view="casLoginView" model="credential">
    <binder>
        <binding property="username" required="true"/>
        <binding property="password" required="true"/>
        <!--
        <binding property="rememberMe" />
        -->
    </binder>
    <on-entry>
        <set name="viewScope.commandName" value="'credential'"/>
        <!--
        <evaluate expression="samlMetadataUIParserAction" />
        -->
    </on-entry>
    <transition on="submit" bind="true" validate="true" to="realSubmit"/>
</view-state>
```
**视图view=“casLoginView” 表示对应视图文件为casLoginView.jsp，model="credential"表示对于模型类为org.jasig.cas.authentication.UsernamePasswordCredential.java。下一步：渲染视图casLoginView.jsp，展现给请求用户。**

## 9. casLoginView.jsp【登录页面】

```jsp
<form id="fm1" action="/cas/login" method="post">
   <input id="username" name="username" class="required" tabindex="1" accesskey="n" type="text" value="" size="25" autocomplete="off"/>  
	<input id="password" name="password" class="required" tabindex="2" accesskey="p" type="password" value="" size="25" autocomplete="off"/>	   
	<input type="hidden" name="execution" value="..." />
	<input type="hidden" name="_eventId" value="submit" />
	<input class="btn-submit" name="submit" accesskey="l" value="LOGIN" tabindex="6" type="submit" />
	<input class="btn-reset" name="reset" accesskey="c" value="CLEAR" tabindex="7" type="reset" />
</form>
```

**输入用户名和密码，点击提交后，传递参数_eventId的值为submit，根据webflow规范，在视图状态viewLoginForm里：transition on=“submit” bind=“true” validate=“true” to=“realSubmit”/>,所以提交后进入行为状态realSubmit。至此，开始了CAS单点登录流程图的第3步：**

![image-20210727000542623](C:\Users\17996\AppData\Roaming\Typora\typora-user-images\image-20210727000542623.png)

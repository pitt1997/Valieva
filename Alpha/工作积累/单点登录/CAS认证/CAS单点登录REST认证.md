# CAS 4.2.x 版本REST认证实现

## 什么是REST认证？ 

通过数据接口对用户进行认证

## cas又是怎么做的？ 

通过请求接口，返回固定格式，进行对密码匹配，判断用户是否合法 

## 什么场景下用REST认证？

用户数据存在远端、不允许cas直接访问数据库、cas不希望你知道帐号数据的表结构

## 一、准备工作

数据库沿用上一篇jdbc文章中的sql脚本

| 项目名                                          | 访问地址                        | 说明                                   |
| ----------------------------------------------- | ------------------------------- | -------------------------------------- |
| cas-overlay-template-master（springboot+meven） | https://cas.server.com:8443/cas | Cas服务器                              |
| cas_db（springboot+Gradle）                     | https://localhost:8080/cas_db   | Cas客户端，提供用户登录授权验证api接口 |

### 二、Cas 服务配置

#### 1、pom.xml 配置

由于Rest方式并不需要直接通过JDBC链接数据库，所以在上一节文章中介绍的JDBC和mysql驱动依赖项就可以删掉了，保留一个就可以了

Markup

```markup
<dependency>
    <groupId>org.apereo.cas</groupId>
    <artifactId>cas-server-support-rest-authentication</artifactId>
    <version>${cas.version}</version>
</dependency>
```



#### 2、application.properties配置 

同样的，把上一篇文章介绍的jdbc配置可以删掉了，配置如下即可

Markdown

```markdown
##
# CAS Authentication Credentials
#
# cas.authn.accept.users=tingfeng::tingfeng
        
##
# REST 认证开始, 请求远程调用接口
#
cas.authn.rest.uri=http://localhost:8080/cas_db/user/login
cas.authn.rest.passwordEncoder.type=DEFAULT
cas.authn.rest.passwordEncoder.characterEncoding=UTF-8
cas.authn.rest.passwordEncoder.encodingAlgorithm=MD5
```

#### 3、流程介绍

当用户点击登录后，cas会发送**post请求**到 `http://localhost:8080/cas_db/login `并且把用户信息以”`用户名:密码`”进行`Base64`编码放在`authorization`**请求头**中

若输入用户名密码为：`admin/123`；那么请求头包括：`authorization=Basic Base64(admin+MD5(123))`



那么发送后客户端必须响应一下数据，cas明确规定如下：

- cas 服务端会通过post请求，并且把用户信息以”用户名:密码”进行Base64编码放在authorization请求头中
- 200状态码：并且格式为`{“@class”:”org.apereo.cas.authentication.principal.SimplePrincipal”,”id”:”casuser”,”attributes”:{}}`是成功的；
- 403状态码：用户不可用；
- 404状态码：账号不存在；
- 423状态码：账户被锁定；
- 428状态码：过期；
- 其他登录失败



### 三、cas_db客户端介绍

使用SpringBoot+Gradle+MyBatis构建

#### SysUser.java

Java

```java
package com.tingfeng.domain;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonProperty;

import javax.validation.constraints.NotNull;
import java.util.HashMap;
import java.util.Map;


/**
 * 机能概要:用于传递给CAS服务器验证数据
 */
public class SysUser {

    @JsonProperty("id")
    private String id;

    @NotNull
    private String username;

    @JsonProperty("@class")
    //需要返回实现org.apereo.cas.authentication.principal.Principal的类名接口
    private String clazz = "org.apereo.cas.authentication.principal.SimplePrincipal";

    @JsonProperty("attributes")
    private Map<String, Object> attributes = new HashMap<>();

    @JsonIgnore
    private String password;

    @JsonIgnore
    //用户是否不可用
    private boolean disabled = false;

    @JsonIgnore
    //用户是否过期
    private boolean expired = false;

    @JsonIgnore
    //是否锁定
    private boolean locked = false;

    /** 省略getter和setter 自行补充**/
}
```

#### SysUserMapper.java

Java

```java
package com.tingfeng.mapper;

import com.tingfeng.domain.SysUser;
import org.apache.ibatis.annotations.Select;

public interface SysUserMapper {

    @Select("select * from sys_user where username=#{username}")
    SysUser getSysUser(String username);

}
```

#### SysUserController.java

Java

```java
package com.tingfeng.controller;

import com.tingfeng.domain.SysUser;
import com.tingfeng.mapper.SysUserMapper;
import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.util.Base64Utils;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.*;

import java.io.UnsupportedEncodingException;


@RestController
@RequestMapping("/user")
public class SysUserController {
    private Logger logger = LogManager.getLogger(SysUserController.class);

    @Autowired
    private SysUserMapper sysUserMapper;

    @PostMapping("/login")
    public Object login(@RequestHeader HttpHeaders httpHeaders) {
        logger.info("Rest api login.");
        logger.debug("request headers: " + httpHeaders);
        SysUser user = null;
        try {
            UserTemp userTemp = obtainUserFormHeader(httpHeaders);

            //当没有 传递 参数的情况
            if(userTemp == null){
                return new ResponseEntity<SysUser>(HttpStatus.NOT_FOUND);
            }

            //尝试查找用户库是否存在
            user = sysUserMapper.getSysUser(userTemp.username);
            if (user != null) {
                if (!user.getPassword().equals(userTemp.password)) {
                    //密码不匹配
                    return new ResponseEntity(HttpStatus.BAD_REQUEST);
                }
                if (user.isDisabled()) {
                    //禁用 403
                    return new ResponseEntity(HttpStatus.FORBIDDEN);
                }
                if (user.isLocked()) {
                    //锁定 423
                    return new ResponseEntity(HttpStatus.LOCKED);
                }
                if (user.isExpired()) {
                    //过期 428
                    return new ResponseEntity(HttpStatus.PRECONDITION_REQUIRED);
                }
            } else {
                //不存在 404
                return new ResponseEntity(HttpStatus.NOT_FOUND);
            }
        } catch (UnsupportedEncodingException e) {
            logger.error("", e);
            new ResponseEntity(HttpStatus.BAD_REQUEST);
        }
        logger.info("[{" + user.getUsername() + "}] login is ok");
        //成功返回json
        return user;
    }


    /**
     * 根据请求头获取用户名及密码
     *
     * @param httpHeaders
     * @return
     * @throws UnsupportedEncodingException
     */
    private UserTemp obtainUserFormHeader(HttpHeaders httpHeaders) throws UnsupportedEncodingException {
        /**
         *
         * This allows the CAS server to reach to a remote REST endpoint via a POST for verification of credentials.
         * Credentials are passed via an Authorization header whose value is Basic XYZ where XYZ is a Base64 encoded version of the credentials.
         */
        //根据官方文档，当请求过来时，会通过把用户信息放在请求头authorization中，并且通过Basic认证方式加密
        String authorization = httpHeaders.getFirst("authorization");//将得到 Basic Base64(用户名:密码)
        if(StringUtils.isEmpty(authorization)){
            return null;
        }
        String baseCredentials = authorization.split(" ")[1];
        String usernamePassword = new String(Base64Utils.decodeFromString(baseCredentials), "UTF-8");//用户名:密码
        logger.debug("login user: " + usernamePassword);
        String credentials[] = usernamePassword.split(":");
        return new UserTemp(credentials[0], credentials[1]);
    }


    /**
     * 解析请求过来的用户
     */
    private class UserTemp {
        private String username;
        private String password;

        public UserTemp(String username, String password) {
            this.username = username;
            this.password = password;
        }
    }

}
```

### 四、测试





### 五、源码下载

https://github.com/X-rapido/CAS_SSO_Record

### 六、参看文档

https://apereo.github.io/cas/5.2.x/installation/Rest-Authentication.html

https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#rest-authentication

https://blog.csdn.net/u010475041/article/details/77972605




# CAS委托认证

## 这个项目是什么

已创建此*cas-pac4j-oauth-demo*项目以测试 CAS 服务器中的身份验证委托。

### 构建和测试 Build & test

构建项目：

```
cd cas-pac4j-oauth-demo
mvn 清洁包
```

并将构建的 WAR( `cas.war`) 作为 JAR（嵌入式 Tomcat）运行：它将在`http://localhost:8080/cas`.

使用`jleleu`/`jleleu`或`leleuj`/`leleuj`登录。

授权的应用程序匹配以下模式：`^http://localhost:.*`.
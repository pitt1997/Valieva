# CAS单点登录集群部署实践

当系统访问量越来越来多时，我们需要对CAS服务的性能进行提升，而通过集群的方式提高CAS的服务性能是比较直接的。在部署的服务中，通过使用Nginx来实现负载均衡分发到CAS具体的服务上，但是我们知道前端每次访问时是随机访问分配的，所以就会出现session共享问题，同时在不同CAS中的TGT分配的ticket也不同，因为默认ticket是保存在内存中的，所以我们需要统一管理session和ticket来解决问题，目前比较流行使用redis作为共享存储，下面是具体实践步骤。

## 一、TICKET的持久化

### 简介

根据CAS[官方文档](https://apereo.github.io/cas/5.3.x/installation/Configuration-Properties.html#ticket-registry)描述，可以发现提供多种不同Ticket的持久化方式，包括：

- JPA
- CouchDb
- Couchbase
- Hazelcast
- MongoDb
- Redis

下面介绍使用Redis的实践。

### 具体步骤

#### 1、添加具体依赖

```xml
	<dependency>
		<groupId>redis.clients</groupId>
		<artifactId>jedis</artifactId>
		<version>2.9.0</version>
	</dependency>
```

#### 2、修改配置信息 **deployerConfigContext.xml**，注释掉jpa注册信息行defaultTicketRegistry

```xml
	<!-- <alias name="defaultTicketRegistry" alias="ticketRegistry" /> -->
```

#### 3、修改配置文件 **cas.properties** 中集群配置 webflow、tgc 的 key

首先通过 com.yggdrasill.cas.DistributedKey 类中的 main 方法生成 key

```java
/**
 * 生成 cas 集群配置时的密钥
 */
public class DistributedKey {
    public static void main(String[] args) {
        tgc();
        webflow();
    }

    /**
     * 生成并打印 webflow 密钥
     */
    public static void webflow() {
        String webflowEncryptionKey = RandomStringUtils.randomAlphabetic(16);

        final OctetSequenceJsonWebKey octetKey = OctJwkGenerator.generateJwk(512);
        final Map<String, Object> params = octetKey.toParams(JsonWebKey.OutputControlLevel.INCLUDE_SYMMETRIC);
        String webflowSigningKey = params.get("k").toString();
        System.out.println("webflow.encryption.key:"+webflowEncryptionKey);
        System.out.println("webflow.signing.key:" + webflowSigningKey);
    }

    /**
     * 生成并打印 tgc 密钥
     */
    public static void tgc() {
        final OctetSequenceJsonWebKey octetKey = OctJwkGenerator.generateJwk(256);
        final Map<String, Object> params = octetKey.toParams(JsonWebKey.OutputControlLevel.INCLUDE_SYMMETRIC);
        String tgcEncryptionKey = params.get("k").toString();

        final OctetSequenceJsonWebKey octetKey2 = OctJwkGenerator.generateJwk(512);
        final Map<String, Object> params2 = octetKey2.toParams(JsonWebKey.OutputControlLevel.INCLUDE_SYMMETRIC);
        String tgcSigningKey = params2.get("k").toString();

        System.out.println("tgc.encryption.key:" + tgcEncryptionKey);
        System.out.println("tgc.signing.key:" + tgcSigningKey);
    }
}

```

eg.

```properties
tgc.encryption.key=pkeBmjr8gLbfBAPOHqorjBkFEDk9f5LYYAW4m4Ct6bk
tgc.signing.key=r47BYPu75RLCzSEV6Eznv57ZKqwknzynuc3oKjlTcXa8THyM4JqpFFfD4y7LwNPypnrz8yNhrXLFcP1EIxZY9w
webflow.encryption.key=JAecVJYHpwmjZTTu
webflow.signing.key=mOBrINzWwxs9GoQcLFpUtRz3q4cm6WVYI3rWzG_V49shITrr56bwc8HnyX-eKjvfI1-gkdJB3TvZhc4bcuVJrw
```

根据以上获取的配置修改 cas.properties 中的 webflow.encryption.key、webflow.signing.key、tgc.encryption.key 和 tgc.signing.key。

#### 4、修改配置文件 **spring-cas-config.xml** 添加自定义bean配置 ticketRegistry 和 redisTemplate

```xml
	<bean id="ticketRegistry" class="com.yggdrasill.cas.custom.sxdx.redisTicketRegistry.RedisTicketRegistry">
		<constructor-arg index="0" ref="redisTemplate" />
		<constructor-arg index="1" value="1800" />
		<constructor-arg index="2" value="10" />
	</bean>
	<bean id="jedisConnFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
		p:hostName="172.168.174.3"
		p:port="6379"
		p:password="123456"
		p:database="0" 
		p:usePool="true"
		p:pool-config-ref="poolConfig" />
	<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxIdle" value="200" />
		<property name="maxTotal" value="100"/>
        <property name="maxWaitMillis" value="2000"/>
	</bean>
	<bean id="redisTemplate" class="com.yggdrasill.cas.demo.redisTicketRegistry.TicketRedisTemplate"
		p:connection-factory-ref="jedisConnFactory">
	</bean>
```

RedisTicketRegistry：

```java
public class RedisTicketRegistry extends AbstractDistributedTicketRegistry implements DisposableBean {
	
	/** redis client. */
	@NotNull
	private final TicketRedisTemplate redisTemplate ;
	
	/**
	 * TGT cache entry timeout in seconds.
	 */
	@Min(0)
	private final int tgtTimeout;
	
	/**
	 * ST cache entry timeout in seconds.
	 */
	@Min(0)
	private final int stTimeout;
	
	private final String PREFIX_CAS = "CAS:TICKET:";

	public RedisTicketRegistry(TicketRedisTemplate redisTemplate, int tgtTimeout, int stTimeout) {
		this.redisTemplate = redisTemplate;
		this.tgtTimeout = tgtTimeout;
		this.stTimeout = stTimeout;
	}

	@Override
	public void addTicket(Ticket ticket) {
		logger.info("Add ticket {}", ticket);
		try {
			this.redisTemplate.boundValueOps(PREFIX_CAS + ticket.getId())
								.set(ticket, getTimeout(ticket),TimeUnit.SECONDS);
		} catch (Exception e) {
			logger.info(e.getMessage(),e);
		}
	}

	@Override
	public Ticket getTicket(String ticketId) {
		logger.info("Get ticket {}", ticketId);
		try {
			Ticket t = (Ticket) this.redisTemplate.boundValueOps(PREFIX_CAS + ticketId).get();
			if (t != null) {
				return getProxiedTicketInstance(t);
			}
		} catch (final Exception e) {
			logger.error("Failed fetching {} ", ticketId, e);
		}
		return null;
	}

	@Override
	public Collection<Ticket> getTickets() {
		Set<Ticket> tickets = new HashSet<Ticket>();
		Set<String> keys = this.redisTemplate.keys(PREFIX_CAS + "*");
		for (String key : keys) {
			Ticket ticket = (Ticket) this.redisTemplate.boundValueOps(key).get();
			if (ticket == null)
				this.redisTemplate.delete(key);
			else {
				tickets.add(ticket);
			}
		}
		return tickets;
	}

	@Override
	protected boolean needsCallback() {
		return true;
	}

	@Override
	protected void updateTicket(Ticket ticket) {
		logger.info("Updating ticket {}", ticket);
		try {
			this.redisTemplate.boundValueOps(PREFIX_CAS + ticket.getId())
								.set(ticket, getTimeout(ticket),TimeUnit.SECONDS);
		} catch (final Exception e) {
			logger.error("Failed updating {}", ticket, e);
		}
	}

	@Override
	public boolean deleteSingleTicket(String ticketId) {
		logger.info("Deleting Single Ticket {}", ticketId);
		try {
			this.redisTemplate.delete(PREFIX_CAS + ticketId);
			return true;
		} catch (final Exception e) {
			logger.error("Failed deleting {}", ticketId, e);
		}
		return false;
	}

	@Override
	public void destroy() throws Exception {
	}

	/**
	 * Gets the timeout value for the ticket.
	 * @param t the Ticket
	 * @return the timeout
	 */
	private int getTimeout(final Ticket t) {
		if (t instanceof TicketGrantingTicket) {
			return this.tgtTimeout;
		} else if (t instanceof ServiceTicket) {
			return this.stTimeout;
		}
		throw new IllegalArgumentException("class RedisTicketRegistry Invalid ticket type");
	}
	
}
```

TicketRedisTemplate：

```java
public class TicketRedisTemplate extends RedisTemplate<String, Ticket> {
	
	@SuppressWarnings("rawtypes")
	public TicketRedisTemplate() {
		RedisSerializer string = new StringRedisSerializer();
		JdkSerializationRedisSerializer jdk = new JdkSerializationRedisSerializer();
		setKeySerializer(string);
		setValueSerializer(jdk);
		setHashKeySerializer(string);
		setHashValueSerializer(jdk);
	}

	public TicketRedisTemplate(RedisConnectionFactory connectionFactory) {
		setConnectionFactory(connectionFactory);
		afterPropertiesSet();
	}
}

```

## 二、SESSION的持久化

### 简介

[Session的持久化](https://apereo.github.io/cas/5.3.x/installation/Webflow-Customization-Sessions.html)和Ticket类似，官方同时也为我们提供了多种方式可选。如下：

- Client-side
- Server-side
- Hazelcast
- Redis
- MongoDb

### 具体步骤

#### 1、这里我们选择Redis方式进行配置，同样添加依赖。

```xml
<dependency>
  <groupId>org.apereo.cas</groupId>
  <artifactId>cas-server-webapp-session-redis</artifactId>
  <version>${cas.version}</version>
</dependency>
```

#### 2、修改 tomcat 的conf目录下的配置文件 **context.xml** ，添加以下配置

redis单机：

```xml
	<Valve className="com.seejoke.tomcat.redissessions.RedisSessionHandlerValve"/> 
	<Manager className="com.seejoke.tomcat.redissessions.RedisSessionManager" 
		host="172.168.174.3"
		port="6379"
		password="123456"
		database="0" 
		maxInactiveInterval="1800" />
```

redis集群:

```xml
	<Valve className="com.seejoke.tomcat.redissessions.RedisSessionHandlerValve"/>
	<Manager className="com.seejoke.tomcat.redissessions.RedisSessionManager"
		sentinelMaster="mymaster"
		sentinels="172.190.80.3:26379,172.190.80.4:26379,172.190.80.5:26379"
		database="2"
		password="123456"
		maxInactiveInterval="3600" />
```
#### 3、复制以下jar包到Tomcat/lib 目录下，通过上面maven坐标可获取

```
jedis-2.8.2.jar
commons-pool2-2.2.jar
tomcat-redis-session-manager.jar
```

jedis-2.8.2.jar和commons-pool2-2.2.jar可在maven官方坐标获取，tomcat-redis-session-manager.jar从github可获取，[github链接在此](https://github.com/jcoleman/tomcat-redis-session-manager)。

#### 4、配置完成重启tomcat

## 三、配置NGINX实现负载均衡

### Nginx中的upstream轮询机制介绍

Nginx中upstream有以下几种方式：

**1、轮询**
默认选项，当weight不指定时，各服务器weight相同，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

```json
upstream bakend {
    server 192.168.1.10;
    server 192.168.1.11;
}
```

**2、weight**

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。如果后端服务器down掉，能自动剔除。比如下面配置，则192.168.1.11服务器的访问量为192.168.1.10服务器的两倍。

```json
upstream bakend {
    server 192.168.1.10 weight=1;
    server 192.168.1.11 weight=2;
}
```

**3、ip_hash**
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session不能跨服务器的问题。如果后端服务器down掉，要手工down掉。

```json
upstream resinserver{
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

**4、fair（第三方插件）**
按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```json
upstream resinserver{
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    fair;
}
```

**5、url_hash（第三方插件）**
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存服务器时比较有效。在upstream中加入hash语句，hash_method是使用的hash算法。

```
upstream resinserver{
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    hash $request_uri;
    hash_method crc32;
}
```

设备的状态有:

- down 表示单前的server暂时不参与负载
- weight 权重,默认为1。 weight越大，负载的权重就越大。
- max_fails 允许请求失败的次数默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误
- fail_timeout max_fails次失败后，暂停的时间。
- backup 备用服务器, 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

### Nginx部署下CAS集群访问情况

**在Nngix中主要配置upstream参数，默认按照轮询调度的策略选择组内服务器处理请求。按时间顺序逐一分配到不同的后端服务器。我们按照该默认方式进行Nginx部署下的CAS集群访问情况。**

#### 1、一个主机部署两个tomcat

在http模块下的upstream配置如下：

```json
    upstream cas {
      	server localhost:8081; 
      	server localhost:8080; 
    }
```

然后再在server模块中配置如下：

```json
    server {
        listen       81; 
        # server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://cas; 
        }
        # .....
    }
```

然后我们启动Nginx服务，访问`http://localhost:81/`，进行轮询访问CAS并且登录。

这里感兴趣可以使用ip_hash的方式进行测试，我们知道ip_hash是对每个请求按访问的ip的hash结果分配，这样每次客户端ip固定访问一个后端服务器，也可以进行负载，但是这样并没有实现session共享。

```json
    upstream cas {
      	server localhost:8081; 
      	server localhost:8080; 
      	ip_hash;
    }
```

重启Nginx登录成功。

#### 2、cas分别部署在不同的主机上

模拟项目真实部署情况，使用https协议，各自分别在两台主机上部署cas，cas两台服务器分别在172.168.98.107和172.168.98.115，nginx部署在主机172.16.174.81上。upstream模块配置443端口，并且server模块需要配置ssl等相关配置项。

upstream模块配置如下：

```json
	upstream cas_server {   
    	server 172.168.98.107:443;  
        server 172.168.98.115:443;  
	}
```

server模块配置如下：

```
    server {
            listen       443 ssl;
            server_name  localhost;
            ssl_certificate /usr/cert/server/server.crt;
            ssl_certificate_key /usr/cert/server/server_key.pem;
            ssl_session_timeout 5m;
            ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_prefer_server_ciphers on;
            location / {
                root   html;
                index  index.html index.htm;
                proxy_pass https://cas_server;
            }
            ...
    }
```

配置完成之后重启nginx，访问https://172.16.174.81/cas/login发现登录成功，观察后台ip先到172.168.98.107完成登录，后续浏览器继续访问地址https://172.16.174.81/cas/login发现请求落到172.168.98.115上，浏览器显示cas是已经登录状态，即访问另外一台主机的CAS也是登录状态，那么到此CAS集群负载部署实现完成。

## 四、相关参考

[关于 tomcat 集群中 session 共享的三种方法](https://blog.csdn.net/QH_JAVA/article/details/45955923?spm=1001.2101.3001.6650.8&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-8.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-8.pc_relevant_default&utm_relevant_index=13)

[nginx集群tomcat下session共享问题](https://blog.csdn.net/tuesdayma/article/details/81387862?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1.pc_relevant_default&utm_relevant_index=2)


>以下分析来自SpringBoot的2.1.6.RELEASE

# 一、Session共享

**Session会话管理及带来的问题？**

* 在Web项目开发中，Session会话管理是一个很重要的部分，用于存储与记录用户的状态或相关的数据。
* 通常情况下session交由容器（tomcat）来负责存储和管理，但是如果项目部署在多台tomcat中，则session管理存在很大的问题
* 多台tomcat之间无法共享session，比如用户在tomcat A服务器上已经登录了，但当负载均衡跳转到tomcat B时，由于tomcat B服务器并没有用户的登录信息，session就失效了，用户就退出了登录
* 一旦tomcat容器关闭或重启也会导致session会话失效
* 因此如果项目部署在多台tomcat中，就需要解决session共享的问题



**分布式 Session 的解决方案：**

* **Session 复制**
  通过对应用服务器的配置开启服务器的 Session 复制功能，在集群中的几台服务器之间同步 Session 对象，使得每台服务器上都保存所有的 Session 信息，这样任何一台宕机都不会导致 Session 的数据丢失，服务器使用 Session 时，直接从本地获取。这种方式的缺点也比较明显。因为 Session 需要时时同步，并且同步过程是有应用服务器来完成，由此对服务器的性能损耗也比较大。

* **Session 绑定**
  利用 hash 算法，比如 **nginx 的 ip_hash,使得同一个 Ip 的请求分发到同一台服务器上**。 这种方式不符合对系统的高可用要求，因为一旦某台服务器宕机，那么该机器上的 Session 也就不复存在了，用户请求切换到其他机器后么有 Session，无法完成业务处理。

* **Session 服务器**
  Session 服务器可以解决上面的所有的问题，利用独立部署的 Session 服务器统一管理 Session，服务器每次读写 Session 时，都访问 Session 服务器。 对于 Session 服务器，我们可以使用 Redis 或者 MongoDB 等内存数据库来保存 Session 中的数据，以此替换掉服务中的 HttpSession。达到 Session 共享的效果。

# 二、SpringSession的使用

**1、引入依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- 引入springboot&redis整合场景 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- 引入springboot&springsession整合场景 -->
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <!--Lettuce是 一 个 基 于 Netty的 NIO方 式 处 理 Redis的 技 术 -->
    <dependency>
        <groupId>io.lettuce</groupId>
        <artifactId>lettuce-core</artifactId>
    </dependency>
</dependencies>
```

**2、编写配置**

```
spring.redis.host=192.168.2.4

spring.session.store-type=redis

server.port=8181
# 设置springsession的session生命周期为30m，表示30分钟
server.servlet.session.timeout=30m
# 用于指定Cookie的存放路径为根路径，用于实现同域名不同项目的session共享
server.servlet.session.cookie.path=/
# 指定Cookie的存域名，用于实现同根域名不同二级子域名的session共享
# server.servlet.session.cookie.domain=zb.com
```

**3、controller的编写**

```java
@RequestMapping("/test/spring/session/save")
public String testSession(HttpSession session,HttpServletRequest request) {
	//如果配置了redis来存储session，则该该session数据将放到redis中
	session.setAttribute("king", "hello-king");
	return "数据存入Session域！";
}
```



# 三、SpringSession Redis存储结构

运行项目后，请求一次接口，进入redis中可以看到，多出来**3组数据**

![image-20211001114617536](https://gitee.com/jobim/blogimage/raw/master/img/20211001114617.png)

session在存储时分为:  

* **hash类型**，session本身的一些属性存储 
  * field=sessionAttr:key2,value=… //往session 设置的属性根据实际情况可能会有多个
  * field=creationTime,value= //创建时间
  * field=maxInactiveInterval，value= //最大生存时间
  * field=lastAccessedTime，value= //最后访问时间
* **string类型**，专门负责用于过期的key存储 
* **set类型**，以时间为key存储在该时间点需要过期的sessionId列表



**string类型**一般设置成session的过期时间如30分钟或者15分钟，同时session的客户端会注册一个redis的key过期事件的监听，一旦有key过期客户端有会事件响应和处理。  在处理事件时可能会需要该session的信息，这时候hash存储就有用了，因此第hash存储的过期时间会比string存储过期时间多1-3min，这就是为什么需要把属性存储和过期分开的原因。   

那set类型的用处呢？因为`Redis`的key过期方式是定期随机测试是否过期和获取时测试是否过期（也称懒删除），由于定期随机测试Task的优先级是比较低的，所以即便这个key已经过期但是没有测试到所以不会触发key过期的事件。所以，第三个存储的意义在于，存储了什么时间点会过期的session，这样可以去主动请求来触发懒删除，以此触发过期事件。





# 四、源码分析

> 好的博客：[SpringSession的源码解析（生成session，保存session，写入cookie全流程分析）](https://blog.csdn.net/u014534808/article/details/105479385)

**主要类的说明：**

| 类名                          | 作用                                           |
| ----------------------------- | ---------------------------------------------- |
| RedisHttpSessionConfiguration | 定义RedisOperationsSessionRepository等类的对象 |
| **SessionRepositoryFilter**   | 过滤器，操作session的入口类                    |
| **SessionRepositoryRequestWrapper** | 是SessionRepositoryFilter内部类，包装HttpRequest请求，调用RedisOperationsSessionRepository类相关的方法都是通过其完成 |
| CookieHttpSessionIdResolver | 这个类主要是调用DefaultCookieSerializer类的方法将sessionid存入cookie中，或者从cookie中读取sessionid，并返回给他的上一层 |
| DefaultCookieSerializer | 这个类是真正的操作cookie的类，设置cookie的相关属性，只需要重新实例化这个类即可 |
| **RedisOperationsSessionRepository** | 这个类的作用是生成session，并将session保存到redis中，另外就是根据sessionid查找session |
| RedisSession | 这个类就是Spring Session的真正的实例对象,这是原始的session |



## RedisHttpSessionConfiguration

* RedisHttpSessionConfiguration 本身是一个 Spring 配置类, 会向 Spring 容器注册 RedisOperationsSessionRepository, redisMessageListenerContainer 等实例;

* RedisMessageListenerContainer, 并将 RedisIndexedSessionRepository 作为 Redis 消息订阅的监听器, 因为它实现了 MessageListener 接口。当 Redis 中 key 过期或销毁时, 会通知将 RedisIndexedSessionRepository 调用其onMessage() 方法来处理消息;

```java
	@Bean
	public RedisOperationsSessionRepository sessionRepository() {
		RedisTemplate<Object, Object> redisTemplate = createRedisTemplate();
		RedisOperationsSessionRepository sessionRepository = new RedisOperationsSessionRepository(
				redisTemplate);
		sessionRepository.setApplicationEventPublisher(this.applicationEventPublisher);
		if (this.defaultRedisSerializer != null) {
			sessionRepository.setDefaultSerializer(this.defaultRedisSerializer);
		}
		sessionRepository
				.setDefaultMaxInactiveInterval(this.maxInactiveIntervalInSeconds);
		if (StringUtils.hasText(this.redisNamespace)) {
			sessionRepository.setRedisKeyNamespace(this.redisNamespace);
		}
		sessionRepository.setRedisFlushMode(this.redisFlushMode);
		int database = resolveDatabase();
		sessionRepository.setDatabase(database);
		return sessionRepository;
	}

	@Bean
	public RedisMessageListenerContainer redisMessageListenerContainer() {
		RedisMessageListenerContainer container = new RedisMessageListenerContainer();
		container.setConnectionFactory(this.redisConnectionFactory);
		if (this.redisTaskExecutor != null) {
			container.setTaskExecutor(this.redisTaskExecutor);
		}
		if (this.redisSubscriptionExecutor != null) {
			container.setSubscriptionExecutor(this.redisSubscriptionExecutor);
		}
		container.addMessageListener(sessionRepository(), Arrays.asList(
				new ChannelTopic(sessionRepository().getSessionDeletedChannel()),
				new ChannelTopic(sessionRepository().getSessionExpiredChannel())));
		container.addMessageListener(sessionRepository(),
				Collections.singletonList(new PatternTopic(
						sessionRepository().getSessionCreatedChannelPrefix() + "*")));
		return container;
	}
```



## SpringHttpSessionConfiguration

* SpringHttpSessionConfiguration 会初始化一个最核心的组件 SessionRepositoryFilter, 该过滤器会拦截所有的 http 请求, 解析并处理 session。

```java
	@Bean
	public <S extends Session> SessionRepositoryFilter<? extends Session> springSessionRepositoryFilter(
			SessionRepository<S> sessionRepository) {
		SessionRepositoryFilter<S> sessionRepositoryFilter = new SessionRepositoryFilter<>(
				sessionRepository);
		sessionRepositoryFilter.setServletContext(this.servletContext);
		sessionRepositoryFilter.setHttpSessionIdResolver(this.httpSessionIdResolver);
		return sessionRepositoryFilter;
	}
```



## SessionRepositoryFilter

* **请求之前会执行SessionRepositoryFilter的doFilterInternal方法：**

* 它会过滤请求时，主要将请求 HttpServletRequest 对象包装成 `SessionRepositoryRequestWrapper` 对象

  ```java
  @Override
  protected void doFilterInternal(HttpServletRequest request,
                                  HttpServletResponse response, FilterChain filterChain)
      throws ServletException, IOException {
      request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);
  
      SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(
          request, response, this.servletContext);
      SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(
          wrappedRequest, response);
  
      try {
          //执行其他过滤器
          filterChain.doFilter(wrappedRequest, wrappedResponse);
      }
      finally {
          //wrappedRequest是SessionRepositoryRequestWrapper类的一个实例
          wrappedRequest.commitSession();
      }
  }
  ```

  

* **SessionRepositoryRequestWrapper类的getSession(true)方法**

* 经过断点调试，并查看调用栈，发现调用这个filterChain.doFilter(wrappedRequest, wrappedResponse);方法之后，**最终会调用到SessionRepositoryRequestWrapper类的getSession(true)方法**。

  ```java
  public HttpSessionWrapper getSession(boolean create) {
  	//1. 获取HttpSessionWrapper实例，如果可以获取到，则说明session已经生成了。就直接返回
  	HttpSessionWrapper currentSession = getCurrentSession();
  	if (currentSession != null) {
  		return currentSession;
  	}
  	//根据request中cookie携带的session id信息，看服务器中有没有存储
  	S requestedSession = getRequestedSession();
  	//不为空，代表不是第一次请求次系统，构建一个session对象放到request对象里面，供后续使用
  	if (requestedSession != null) {
  		if (getAttribute(INVALID_SESSION_ID_ATTR) == null) {
  			requestedSession.setLastAccessedTime(Instant.now());
  			this.requestedSessionIdValid = true;
  			currentSession = new HttpSessionWrapper(requestedSession, getServletContext());
  			currentSession.setNew(false);
  			setCurrentSession(currentSession);
  			return currentSession;
  		}
  	}
  	//如果获取不到session，则进入下面分支，创建session
  	else {
  		//省略部分代码
  		//如果create为false，直接返回null
  		if (!create) {
  			return null;
  		}
  		//省略部分代码
  		//如果create为true，则调用RedisOperationsSessionRepository类的createSession方法创建session实例
  		S session = SessionRepositoryFilter.this.sessionRepository.createSession();
  		session.setLastAccessedTime(Instant.now());
  		currentSession = new HttpSessionWrapper(session, getServletContext());
  		setCurrentSession(currentSession);
  		return currentSession;
  	}
  }
  ```

* **RedisOperationsSessionRepository类的createSession()方法**

* 从前面的代码分析我们可以知道如果获取不到session实例，则会调用createSession()方法进行创建。这个方法是在RedisOperationsSessionRepository类中，该方法比较简单，主要就是实例化RedisSession对象。其中RedisSession对象中包括了sessionid，creationTime，maxInactiveInterval和lastAccessedTime等属性。其中原始的sessionid是一段唯一的UUID字符串。

  ```java
  public class RedisOperationsSessionRepository implements ....{
  	@Override
  	public RedisSession createSession() {
  		Duration maxInactiveInterval = Duration
  				.ofSeconds((this.defaultMaxInactiveInterval != null)
  						? this.defaultMaxInactiveInterval
  						: MapSession.DEFAULT_MAX_INACTIVE_INTERVAL_SECONDS);
  		RedisSession session = new RedisSession(maxInactiveInterval);
  		session.flushImmediateIfNecessary();
  		return session;
  	}
  
  
  	final class RedisSession implements Session {
  		private final MapSession cached;
  		private Instant originalLastAccessTime;
  		private Map<String, Object> delta = new HashMap<>();
  		private boolean isNew;
  		private String originalPrincipalName;
  		private String originalSessionId;
  
  		RedisSession(Duration maxInactiveInterval) {
  			this(new MapSession());
  			this.cached.setMaxInactiveInterval(maxInactiveInterval);
              //创建时间
  			this.delta.put(CREATION_TIME_ATTR, getCreationTime().toEpochMilli());
              //最大生存时间
  			this.delta.put(MAX_INACTIVE_ATTR, (int) getMaxInactiveInterval().getSeconds());
              //最后访问时间
  			this.delta.put(LAST_ACCESSED_ATTR, getLastAccessedTime().toEpochMilli());
  			this.isNew = true;
  		}
      }
  }
  ```



* **doFilterInternal方法finally代码块中会调用SessionRepositoryRequestWrapper类内部的commitSession()方法**

* commitSession()方法会保存session信息到Redis中，并将sessionid写到cookie中。

  ```java
  private void commitSession() {
      //当前请求会话中获取HttpSessionWrapper对象的实例
      HttpSessionWrapper wrappedSession = getCurrentSession();
      //如果wrappedSession为空则调用expireSession写入一个空值的cookie
      if (wrappedSession == null) {
          if (isInvalidateClientSession()) {
              SessionRepositoryFilter.this.httpSessionIdResolver.expireSession(this,
                                                                               this.response);
          }
      }
      else {
          //获取session
          S session = wrappedSession.getSession();
          clearRequestedSessionCache();
          //调用RedisOperationsSessionRepository类的save(session)方法将session信息保存到Redis中
          SessionRepositoryFilter.this.sessionRepository.save(session);
          //获取sessionid
          String sessionId = session.getId();
          if (!isRequestedSessionIdValid()
              || !sessionId.equals(getRequestedSessionId())) {
              //调用CookieHttpSessionIdResolver类的setSessionId方法将sessionid设置到Cookie中
              SessionRepositoryFilter.this.httpSessionIdResolver.setSessionId(this,
                                                                              this.response, sessionId);
          }
      }
  }
  ```

## CookieHttpSessionIdResolver

* CookieHttpSessionIdResolver类的`setSessionId`方法

* `setSessionId`方法主要就是将生成的sessionid设置到请求会话中，然后调用DefaultCookieSerializer类的`writeCookieValue`方法将sessionid设置到cookie中。

```java
@Override
public void setSessionId(HttpServletRequest request, HttpServletResponse response,
                         String sessionId) {
    //如果sessionid等于请求头中的sessionid，则直接返回
    if (sessionId.equals(request.getAttribute(WRITTEN_SESSION_ID_ATTR))) {
        return;
    }
    //将sessionid设置到请求头中
    request.setAttribute(WRITTEN_SESSION_ID_ATTR, sessionId);
    //将sessionid写入cookie中
    this.cookieSerializer
        .writeCookieValue(new CookieValue(request, response, sessionId));
}

```



## DefaultCookieSerializer

* DefaultCookieSerializer类的`writeCookieValue`方法

```java
@Override
public void writeCookieValue(CookieValue cookieValue) {
    HttpServletRequest request = cookieValue.getRequest();
    HttpServletResponse response = cookieValue.getResponse();

    StringBuilder sb = new StringBuilder();
    //设置cookie的名称，默认是SESSION
    sb.append(this.cookieName).append('=');
    //设置cookie的值，就是传入的sessionid
    String value = getValue(cookieValue);
    if (value != null && value.length() > 0) {
        validateValue(value);
        sb.append(value);
    }
    //设置cookie的失效时间
    int maxAge = getMaxAge(cookieValue);
    if (maxAge > -1) {
        sb.append("; Max-Age=").append(cookieValue.getCookieMaxAge());
        OffsetDateTime expires = (maxAge != 0)
            ? OffsetDateTime.now().plusSeconds(maxAge)
            : Instant.EPOCH.atOffset(ZoneOffset.UTC);
        sb.append("; Expires=")
            .append(expires.format(DateTimeFormatter.RFC_1123_DATE_TIME));
    }
    String domain = getDomainName(request);
    //设置Domain属性，默认就是当前请求的域名，或者ip
    if (domain != null && domain.length() > 0) {
        validateDomain(domain);
        sb.append("; Domain=").append(domain);
    }
    //设置Path属性，默认是当前项目名（例如：/spring-boot-session），可重设
    String path = getCookiePath(request);
    if (path != null && path.length() > 0) {
        validatePath(path);
        sb.append("; Path=").append(path);
    }
    if (isSecureCookie(request)) {
        sb.append("; Secure");
    }
    //设置在HttpOnly是否只读属性。
    if (this.useHttpOnlyCookie) {
        sb.append("; HttpOnly");
    }
    if (this.sameSite != null) {
        sb.append("; SameSite=").append(this.sameSite);
    }
    //将设置好的cookie放入响应头中
    response.addHeader("Set-Cookie", sb.toString());
}

```

> 分析可得：如果要实现同域名下不同项目的项目之间session共享，我们只需要改变Path属性即可
>
> 如果要指定域名的话，我们只需要设置DomainName属性即可





## 总结

* 当请求进来的时候，SessionRepositoryFilter 会先拦截到请求，将 request 和 response 对象转换成 SessionRepositoryRequestWrapper 和 SessionRepositoryResponseWrapper。
* 后续当第一次调用 request 的 getSession() 方法时，会调用到 SessionRepositoryRequestWrapper 的 getSession() 方法。这个方法的逻辑是先从 request 的属性中查找，如果找不到；再查找一个 key 值是 "SESSION" 的cookie，通过这个 cookie 拿到 sessionId 去 redis 中查找，如果查不到，就直接创建一个 **RedisSession 对象**。
* SessionRepositoryFilter中doFilterInternal方法的finally代码块中会调用SessionRepositoryRequestWrapper类内部的commitSession()方法**保存session信息到Redis中，并将sessionid写到cookie中**。

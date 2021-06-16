---
title: SpringSecurity实战
date: 2021-05-30 14:56:44
tags:
---

# 2. 表单认证

## 默认表单认证

​	在WebSecurity类上加上@EnableWebSecurity，就会自动被Spring发现并注册。

接着 查看 WebSecurityConfigurerAdapter 类 对 configure（ HttpSecurity http） 的 定义。

```java
protected void configure(HttpSecurity http) throws Exception {
		this.logger.debug("Using default configure(HttpSecurity). "
				+ "If subclassed this will potentially override subclass configure(HttpSecurity).");
		http.authorizeRequests((requests) -> requests.anyRequest().authenticated());
		http.formLogin();
		http.httpBasic();
	}
```

可以看到WebSecurityConfigurerAdapter已经默认声明了一些安全特性：

* 验证所有请求。
* 允许用户使用表单登录进行身份验证（Spring Security 提供了一个简单的表单登录页面）。
* 允许用户使用HTTP 基本认证。

![image-20210530153826450](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20210530153826450.png)

## 自定义表单登录页

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login.html")
                .permitAll()
                .and()
                .csrf().disable();
    }
}
```

​	在自定义表单登录页之后，处理登录请求的URL也会相应改变。如何自定义URL呢？很简单，Spring Security在表单定制里提供了相应的支持。

```
.loginProcessingUrl("/login")
```

 可以指定登录成功时的逻辑。

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login.html")
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        // xxx
                    }
                })
                .permitAll()
                .and()
                .csrf().disable();
    }
}
```

​	表单登录配置模块提供了 successHandler（）和 failureHandler（）两个方法，分别处理登录成功和登录失败的逻辑。其中，successHandler（）方法带有一个Authentication参数，携带当前登录用户名及其角色等信息；而failureHandler（）方法携带一个AuthenticationException异常参数。具体处理方式需按照系统的情况自定义。

# 3. 认证与授权

建立两个Controller

## 默认数据库模型的认证与授权

### 资源准备

```java
@RestController("/user")
public class UserController {
    @GetMapping("/wow")
    public String wow() {
        return "wow user";
    }
}

@RestController("/hello")
public class HelloController {
    @RequestMapping("/index")
    public String index() {
        return "login success";
    }
    @GetMapping("/hello")
    public String hello() {
        return "hello spring security";
    }
}
```

### 资源授权

​	antMatchers（）是一个采用ANT模式的URL匹配器。ANT模式使用？匹配任意单个字符，使用*匹配0或任意数量的字符，使用**匹配0或者更多的目录。

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/user/**").hasRole("admin")
                .antMatchers("/hello/**").hasRole("user")
                .and()
                .formLogin()
                .loginPage("/login.html")
                .permitAll()
                .and()
                .csrf().disable();
    }
}
```

​	页面显示403错误，表示该用户授权失败（401代表该用户认证失败）。也就是说，本次访问已经通过了认证环节，只是在授权的时候被驳回了。认证环节是没有问题的，因为Spring Security默认的用户角色正是user。

### 基于内存的多用户支持

​	到目前为止，我们仍然只有一个可登录的用户，怎样引入多用户呢？非常简单，我们只需实现一个自定义的UserDetailsService即可。

```java
	@Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("user").password("123").roles("user").build());
        manager.createUser(User.withUsername("admin").password("123").roles("user", "admin").build());
        return manager;
    }
```

​	为其添加一个@bean注解，便可被Spring Security发现并使用。Spring Security支持各种来源的用户数据，包括内存、数据库、LDAP等。它们被抽象为一个UserDetailsService接口，任何实现了 UserDetailsService 接口的对象都可以作为认证数据源。在这种设计模式下，Spring Security显得尤为灵活。
InMemoryUserDetailsManager是UserDetailsService接口中的一个实现类，它将用户数据源寄存在内存里，在一些不需要引入数据库这种重数据源的系统中很有帮助。
这里仅仅调用createUser（）生成两个用户，并赋予相应的角色。它会工作得很好，多次重启服务也不会出现问题。为什么要强调多次重启服务呢？稍后揭晓答案。

### 基于默认数据库模型的认证与授权

**数据库准备**

```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```

前面介绍过，JdbcUserDetailsManager设定了一个默认的数据库模型，Spring Security将该模型定义在/org/springframework/security/core/userdetails/jdbc/users.ddl内。

```sql
create table users(username varchar_ignorecase(50) not null primary key,password varchar_ignorecase(500) not null,enabled boolean not null);
create table authorities (username varchar_ignorecase(50) not null,authority varchar_ignorecase(50) not null,constraint fk_authorities_users foreign key(username) references users(username));
create unique index ix_auth_username on authorities (username,authority);
```

​	将其复制到MySQL命令窗口执行时，会报错，因为该语句是用hsqldb创建的，而MySQL不支持varchar_ignorecase这种类型。怎么办呢？很简单，将varchar_ignorecase改为MySQL支持的varchar即可。

```mysql
CREATE TABLE users (
	username varchar(50) NOT NULL PRIMARY KEY,
	PASSWORD varchar(500) NOT NULL,
	enabled boolean NOT NULL
);

CREATE TABLE authorities (
	username varchar(50) NOT NULL,
	authority varchar(50) NOT NULL,
	CONSTRAINT fk_authorities_users FOREIGN KEY (username) REFERENCES users (username)
);

CREATE UNIQUE INDEX ix_auth_username ON authorities (username, authority);
```

**编码**

```java
@Autowired
    private DataSource dataSource;
    @Bean
    public UserDetailsService userDetailsService() {
        JdbcUserDetailsManager manager = new JdbcUserDetailsManager();
        manager.setDataSource(dataSource);
        manager.createUser(User.withUsername("user").password("123").roles("user").build());
        manager.createUser(User.withUsername("admin").password("123").roles("user", "admin").build());
        return manager;
    }
```

**判断是否存在**

```java
@Bean
    public UserDetailsService userDetailsService() {
        JdbcUserDetailsManager manager = new JdbcUserDetailsManager();
        manager.setDataSource(dataSource);
        if (!manager.userExists("user")) {
            manager.createUser(User.withUsername("user").password("123").roles("user").build());
        }
        if (!manager.userExists("admin")) {
            manager.createUser(User.withUsername("admin").password("123").roles("user", "admin").build());
        }
        return manager;
    }
```

configure方法也可以创建user

![](https://pic.imgdb.cn/item/60b39e5839f6859bc2904f30.jpg)

## 自定义数据库模型的认证与授权

### 实现UserDetails

​	UserDetailsService仅定义了一个loadUserByUsername方法，用于获取一个UserDetails对象。UserDetails对象包含了一系列在验证时会用到的信息，包括用户名、密码、权限以及其他信息，Spring Security会根据这些信息判定验证是否成功。

**数据库准备**

```mysql
CREATE TABLE users2 (
	id bigint(20) not null auto_increment,
	username varchar(50) NOT NULL,
	PASSWORD varchar(500) NOT NULL,
	enabled boolean NOT NULL,
    roles varchar(500),
	PRIMARY KEY(id),
	KEY username (username)
);
insert into users2(username, password, enabled, roles) values('user', '123', true, 'user');
insert into users2(username, password, enabled, roles) values('admin', '123', true, 'admin,user');
```

**编码实现**

编写实体

```java
public class Users2 implements UserDetails {
    private Long id;
    private String username;
    private String password;
    private String roles;
    private boolean enabled;
    private List<GrantedAuthority> authorities;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getRoles() {
        return roles;
    }

    public void setRoles(String roles) {
        this.roles = roles;
    }

    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }
}
```

实现UserDetails定义的几个方法：

* isAccountNonExpired、isAccountNonLocked 和 isCredentialsNonExpired 暂且用不到，统一返回true，否则Spring Security会认为账号异常。
*  isEnabled对应enable字段，将其代入即可。
*  getAuthorities方法本身对应的是roles字段，但由于结构不一致，所以此处新建一个，并在后续进行填充。

### 实现UserDetailsService

**持久层准备**

```xml
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>
```

```java
@SpringBootApplication
@MapperScan("top.sorie.mapper")
public class SecuritydemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecuritydemoApplication.class, args);
    }

}
```

```java
@Component
public interface Users2Mapper {
    @Select("SELECT * FROM users2 WHERE username=#{username}")
    Users2 findByUserName(@Param("username") String username);
}
```

**编码实现**

​	实现UserDetailService

```java
@Service
public class MyUserDetailService implements UserDetailsService {
    @Autowired
    private Users2Mapper users2Mapper;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if (username == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        Users2 users2 = users2Mapper.findByUserName(username);
        if (users2 == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        users2.setAuthorities(AuthorityUtils.commaSeparatedStringToAuthorityList(users2.getRoles()));
        return users2;
    }
}

```

修改配置

```java
    @Autowired
    private MyUserDetailService myUserDetailService;
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/user/**").hasAuthority("admin")
                .antMatchers("/hello/**").hasAuthority("user")
                .and()
                .userDetailsService(myUserDetailService)
                .formLogin()
                .loginPage("/login.html")
                .permitAll()
                .and()
                .csrf().disable();
    }
```

遇到这个问题

https://stackoverflow.com/questions/49654143/spring-security-5-there-is-no-passwordencoder-mapped-for-the-id-null

# 4. 实现图形验证码

## 使用过滤器实现图形验证码

### 自定义过滤器

```java
	@Override
	public HttpSecurity addFilterAfter(Filter filter, Class<? extends Filter> afterFilter) {
		return addFilterAtOffsetOf(filter, 1, afterFilter);
	}

	@Override
	public HttpSecurity addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter) {
		return addFilterAtOffsetOf(filter, -1, beforeFilter);
	}

	private HttpSecurity addFilterAtOffsetOf(Filter filter, int offset, Class<? extends Filter> registeredFilter) {
		int order = this.filterOrders.getOrder(registeredFilter) + offset;
		this.filters.add(new OrderedFilter(filter, order));
		return this;
	}

	@Override
	public HttpSecurity addFilter(Filter filter) {
		Integer order = this.filterOrders.getOrder(filter.getClass());
		if (order == null) {
			throw new IllegalArgumentException("The Filter class " + filter.getClass().getName()
					+ " does not have a registered order and cannot be added without a specified order. Consider using addFilterBefore or addFilterAfter instead.");
		}
		this.filters.add(new OrderedFilter(filter, order));
		return this;
	}

	public HttpSecurity addFilterAt(Filter filter, Class<? extends Filter> atFilter) {
		return addFilterAtOffsetOf(filter, 0, atFilter);
	}
```

### 图形验证码过滤器

​	制作图形验证码的方法很多，例如kaptcha。

```xml
	<dependency>
       <groupId>com.github.penggle</groupId>
       <artifactId>kaptcha</artifactId>
       <version>2.3.2</version>
    </dependency>
```

​	配置一个kaptcha实例

```java
@Bean
    public Producer captcha() {
        // 配置图形验证码的基本参数
        Properties properties = new Properties();
        // 图片宽度
        properties.setProperty("kapacha.image.width", "150");
        // 高度
        properties.setProperty("kapacha.image.height", "50");
        // 字符集
        properties.setProperty("kaptacha.textproducer.char.string", "0123456789");
        // 字符长度
        properties.setProperty("kaptacha.textproducer.char.length", "4");
        Config config = new Config(properties);
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
```

创建CaptchaController用于获取图形验证码。

```java
@Controller
public class CaptchaController {
    @Autowired
    private Producer captchaProducer;

    @GetMapping("/captcha.jpg")
    public void getCaptcha(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 设置内容类型
        response.setContentType("image/jpeg");
        // 创建验证码文本
        String capText = captchaProducer.createText();
        // 将验证码文本设置到Session
        request.getSession().setAttribute("captcha", capText);
        // 创建验证码图片
        BufferedImage bufferedImage = captchaProducer.createImage(capText);
        // 获取响应输出流
        ServletOutputStream out = response.getOutputStream();
        // 输出
        ImageIO.write(bufferedImage, "jpg", out);
        // 推送并关闭响应流
        try {
            out.flush();
        } finally {
            out.close();
        }
    }
}

```

​	有了图形验证码的API之后，就可以自定义验证码校验过滤器了。虽然Spring Security的过滤器链对过滤器没有特殊要求，只要继承了 Filter 即可，但是在 Spring 体系中，推荐使用OncePerRequestFilter来实现，它可以确保一次请求只会通过一次该过滤器（Filter实际上并不能保证这一点）。

```java
public class VerificationCodeException extends AuthenticationException {
    public VerificationCodeException() {
        super("图形验证码校验失败");
    }
}

public class VerificationCodeFilter extends OncePerRequestFilter {
    private AuthenticationFailureHandler failureHandler = new AuthenticationFailureHandler() {
        @Override
        public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
            response.setContentType("application/json;charset=utf-8");
            PrintWriter out = response.getWriter();
            out.write(exception.getMessage());
            out.flush();
            out.close();

        }
    };
    @Override
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, FilterChain filterChain) throws ServletException, IOException {
        // 非登录请求不校验验证码
        if (!"/auth/form".equals(httpServletRequest.getRequestURI())) {
            filterChain.doFilter(httpServletRequest, httpServletResponse);
        } else {
            try {
                verificationCode(httpServletRequest);
            } catch (VerificationCodeException e) {
                failureHandler.onAuthenticationFailure(httpServletRequest, httpServletResponse, e);
            }
        }
    }
    public void verificationCode(HttpServletRequest request) {
        String requestCode = request.getParameter("captcha");
        HttpSession session = request.getSession();
        String saveCode = (String) session.getAttribute("captcha");
        if (!StringUtils.isEmpty(saveCode)) {
            // 清除
            session.removeAttribute("captcha");
        }
        // 校验不通过，抛出异常
        if (StringUtils.isEmpty(request) || StringUtils.isEmpty(saveCode) || !requestCode.equals(saveCode)) {
            throw new VerificationCodeException();
        }
    }
}
```

然后过滤器需要配置

![](https://pic.imgdb.cn/item/60b783bf39f6859bc2b5b81d.jpg)

## 使用自定义认证实现图形验证码

​	提供了一种更优雅的实现图形码的方式

### 认识AuthenticationProvider

​	用户，在Spring Security中被称为主体（principal）。

```java
public interface Authentication extends Principal, Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();

	Object getCredentials();

	Object getDetails();

	Object getPrincipal();

	boolean isAuthenticated();

	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;

}
```

​	由于大部分场景下身份验证都是基于用户名和密码进行的，所以Spring Security提供了一个UsernamePasswordAuthenticationToken用于代指这一类证明（例如，用SSH KEY也可以登录，但它不属于用户名和密码登录这个范畴，如有必要，也可以自定义提供）。在前面使用的表单登录中，每一个登录用户都被包装为一个 UsernamePasswordAuthenticationToken，从而在Spring Security的各个AuthenticationProvider中流动。

```java
public interface AuthenticationProvider {


	Authentication authenticate(Authentication authentication) throws AuthenticationException;

	boolean supports(Class<?> authentication);

}

```

​	一次完整的认证可以包含多个AuthenticationProvider，一般由ProviderManager管理。

### 自定义AuthenticationProvider

Spring Security提供了多种常见的认证技术，包括但不限于以下几种：

* HTTP层面的认证技术，包括HTTP基本认证和HTTP摘要认证两种。
* 基于LDAP的认证技术（Lightweight Directory Access Protocol，轻量目录访问协议）。
* 聚焦于证明用户身份的OpenID认证技术。
* 聚焦于授权的OAuth认证技术。
* 系统内维护的用户名和密码认证技术。

在 AbstractUserDetailsAuthenticationProvider 中实现了基本的认证流程，通过继承AbstractUserDetailsAuthenticationProvider，并实现retrieveUser和additionalAuthenticationChecks两个抽象方法即可自定义核心认证过程，灵活性非常高。

​	Spring Security 同样提供一个继承自 AbstractUserDetailsAuthenticationProvider 的 Authenti cationProvider。

​	DaoAuthenticationProvider的用户信息来源于UserDetailsService，并且整合了密码编码的实现，在前面章节中学习的表单认证就是由DaoAuthenticationProvider提供的。

### 实现图形验证码的AuthenticationProvider

![image-20210602210509043](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20210602210509043.png)

​	验证流程中添加新的逻辑后似乎有些问题。在additionalAuthenticationChecks中，我们可以得到的参数是来自UserDetailsService的UserDetails，以及根据用户提交的账号信息封装而来的UsernamePasswordAuthenticationToken，而图形验证码的校验必须要有HttpServletRequest对象，因为用户提交的验证码和session存储的验证码都需要从用户的请求中获取，这是否意味着这种实现方式不可行呢？并非如此，Authentication实际上还可以携带账号信息之外的数据。

​	前面提到过，一次完整的认证可以包含多个AuthenticationProvider，这些AuthenticationProvider都是由ProviderManager管理的，而 ProviderManager 是由 UsernamePasswordAuthenticationFilter 调用的。也就是说，所有的AuthenticationProvider包含的Authentication都来源于UsernamePasswordAuthenticationFilter。

​	AbstractAuthenticationProcessingFilter本身并没有设置用户详细信息的流程，而且是通过标准接口AuthenticationDetailsSource构建的，这意味着它是一个允许定制的特性。

![](https://pic.imgdb.cn/item/60b782cf39f6859bc2a1460f.jpg)

接下来实现我们自定义的AuthenticationProvider。

![](https://pic.imgdb.cn/item/60b782dc39f6859bc2a25f02.jpg)

![](https://pic.imgdb.cn/item/60b782f439f6859bc2a48ffd.jpg)

​	想要应用自定义的 AuthenticationProvider 和 AuthenticationDetailsSource，还需在WebSecurityConfig中完成剩余的配置。

![](https://pic.imgdb.cn/item/60b7849439f6859bc2c7f2d3.jpg)

![](https://pic.imgdb.cn/item/60b784a239f6859bc2c93632.jpg)

# 5. 自动登录和注销登录

自动登录是将用户的登录信息保存在用户浏览器的cookie中，当用户下次访问时，自动实现校验并建立登录态的一种机制。
Spring Security提供了两种非常好的令牌：

* 用散列算法加密用户必要的登录信息并生成令牌。
* 数据库等持久性数据存储机制用的持久化令牌。

![](https://pic.imgdb.cn/item/60bafa778355f7f718418b7f.jpg)

​	其中，expirationTime指本次自动登录的有效期，key为指定的一个散列盐值，用于防止令牌被修改。通过这种方式生成cookie后，在下次登录时，Spring Security首先用Base64简单解码得到用户名、过期时间和加密散列值；然后使用用户名得到密码；接着重新以该散列算法正向计算，并将计算结果与旧的加密散列值进行对比，从而确认该令牌是否有效。

## 实现自动登录

**散列加密方案**

 ```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .userDetailsService(myUserDetailService)
                .formLogin()
                .loginPage("/login.html")
                .loginProcessingUrl("/doLogin")
                .defaultSuccessUrl("/hello/index")
                .failureUrl("/login.html")
                .usernameParameter("uname")
                .passwordParameter("passwd")
                .permitAll()
                .and()
                .csrf().disable()
                .rememberMe();
    }
 ```

​	前提是已经实现了一个 UserDetailsService。重启服务后访问受限 API，这次在表单登录页中多了一个可选框，如图5-1所示。

![](https://pic.imgdb.cn/item/60bafb888355f7f71851da30.jpg)

![](https://pic.imgdb.cn/item/60bafe0a8355f7f718776fb7.jpg)

```java
/**
	 * Calculates the digital signature to be put in the cookie. Default value is MD5
	 * ("username:tokenExpiryTime:password:key")
	 */
	protected String makeTokenSignature(long tokenExpiryTime, String username, String password) {
		String data = username + ":" + tokenExpiryTime + ":" + password + ":" + getKey();
		try {
			MessageDigest digest = MessageDigest.getInstance("MD5");
			return new String(Hex.encode(digest.digest(data.getBytes())));
		}
		catch (NoSuchAlgorithmException ex) {
			throw new IllegalStateException("No MD5 algorithm available!");
		}
	}

	@Override
	public void onLoginSuccess(HttpServletRequest request, HttpServletResponse response,
			Authentication successfulAuthentication) {
		String username = retrieveUserName(successfulAuthentication);
		String password = retrievePassword(successfulAuthentication);
		// If unable to find a username and password, just abort as
		// TokenBasedRememberMeServices is
		// unable to construct a valid token in this case.
		if (!StringUtils.hasLength(username)) {
			this.logger.debug("Unable to retrieve username");
			return;
		}
		if (!StringUtils.hasLength(password)) {
			UserDetails user = getUserDetailsService().loadUserByUsername(username);
			password = user.getPassword();
			if (!StringUtils.hasLength(password)) {
				this.logger.debug("Unable to obtain password for user: " + username);
				return;
			}
		}
		int tokenLifetime = calculateLoginLifetime(request, successfulAuthentication);
		long expiryTime = System.currentTimeMillis();
		// SEC-949
		expiryTime += 1000L * ((tokenLifetime < 0) ? TWO_WEEKS_S : tokenLifetime);
		String signatureValue = makeTokenSignature(expiryTime, username, password);
		setCookie(new String[] { username, Long.toString(expiryTime), signatureValue }, tokenLifetime, request,
				response);
		if (this.logger.isDebugEnabled()) {
			this.logger.debug(
					"Added remember-me cookie for user '" + username + "', expiry: '" + new Date(expiryTime) + "'");
		}
	}
```

​	其中，在没有指定时，key是一个UUID字符串。这将导致每次重启服务后，key都会重新生成，使得重启之前的所有自动登录cookie失效。除此之外，在多实例部署的情况下，由于实例间的 key并不相同，所以当用户访问系统的另一个实例时，自动登录策略就会失效。合理的用法是指定key。

![](https://pic.imgdb.cn/item/60bafeb18355f7f7187f51f3.jpg)

**持久化令牌方案**

​	持久化令牌方案在交互上与散列加密方案一致，都是在用户勾选Remember-me之后，将生成的令牌发送到用户浏览器，并在用户下次访问系统时读取该令牌进行认证。不同的是，它采用了更加严谨的安全性设计。

​	在持久化令牌方案中，最核心的是series和token两个值，它们都是用MD5散列过的随机字符串。不同的是，series仅在用户使用密码重新登录时更新，而token会在每一个新的session中都重新生成。
​	这样设计有什么好处呢？
​	首先，解决了散列加密方案中一个令牌可以同时在多端登录的问题。每个会话都会引发token的更新，即每个token仅支持单实例登录。
​	其次，自动登录不会导致series变更，而每次自动登录都需要同时验证series和token两个值，当该令牌还未使用过自动登录就被盗取时，系统会在非法用户验证通过后刷新 token 值，此时在合法用户的浏览器中，该token值已经失效。当合法用户使用自动登录时，由于该series对应的 token 不同，系统可以推断该令牌可能已被盗用，从而做一些处理。例如，清理该用户的所有自动登录令牌，并通知该用户可能已被盗号等。
在实现上，Spring Security使用PersistentRememberMeToken来表明一个验证实体。

​	![](https://pic.imgdb.cn/item/60bb02ec8355f7f718a5f0e1.jpg)

![](https://pic.imgdb.cn/item/60bb03118355f7f718a6f2c8.jpg)

```java
public interface PersistentTokenRepository {

   void createNewToken(PersistentRememberMeToken token);

   void updateToken(String series, String tokenValue, Date lastUsed);

   PersistentRememberMeToken getTokenForSeries(String seriesId);

   void removeUserTokens(String username);

}
```

​	与散列加密方案一样，持久化令牌方案也需要实现UserDetailService。如果一切顺利那么重启服务后我们就已经引入持久化令牌的自动登录方案了。与散列加密方案相比，持久化令牌方案在用户体验层面没有任何变化。登录之后，可以看到cookie经过Base64解码之后不再是散列加密的形式了。

![](https://pic.imgdb.cn/item/60bb04f78355f7f718af8497.jpg)

```java
@Override
	protected UserDetails processAutoLoginCookie(String[] cookieTokens, HttpServletRequest request,
			HttpServletResponse response) {
		if (cookieTokens.length != 2) {
			throw new InvalidCookieException("Cookie token did not contain " + 2 + " tokens, but contained '"
					+ Arrays.asList(cookieTokens) + "'");
		}
		String presentedSeries = cookieTokens[0];
		String presentedToken = cookieTokens[1];
		PersistentRememberMeToken token = this.tokenRepository.getTokenForSeries(presentedSeries);
		if (token == null) {
			// No series match, so we can't authenticate using this cookie
			throw new RememberMeAuthenticationException("No persistent token found for series id: " + presentedSeries);
		}
		// We have a match for this user/series combination
		if (!presentedToken.equals(token.getTokenValue())) {
			// Token doesn't match series value. Delete all logins for this user and throw
			// an exception to warn them.
			this.tokenRepository.removeUserTokens(token.getUsername());
			throw new CookieTheftException(this.messages.getMessage(
					"PersistentTokenBasedRememberMeServices.cookieStolen",
					"Invalid remember-me token (Series/token) mismatch. Implies previous cookie theft attack."));
		}
		if (token.getDate().getTime() + getTokenValiditySeconds() * 1000L < System.currentTimeMillis()) {
			throw new RememberMeAuthenticationException("Remember-me login has expired");
		}
		// Token also matches, so login is valid. Update the token value, keeping the
		// *same* series number.
		this.logger.debug(LogMessage.format("Refreshing persistent login token for user '%s', series '%s'",
				token.getUsername(), token.getSeries()));
		PersistentRememberMeToken newToken = new PersistentRememberMeToken(token.getUsername(), token.getSeries(),
				generateTokenData(), new Date());
		try {
			this.tokenRepository.updateToken(newToken.getSeries(), newToken.getTokenValue(), newToken.getDate());
			addCookie(newToken, request, response);
		}
		catch (Exception ex) {
			this.logger.error("Failed to update token: ", ex);
			throw new RememberMeAuthenticationException("Autologin failed due to data access problem");
		}
		return getUserDetailsService().loadUserByUsername(token.getUsername());
	}
```

​	显然，两种方案都存在cookie被盗取导致身份被暂时利用的可能，如果有更高的安全性需求，建议使用Spring Security提供的令牌持久化方案。当然，最安全的方式还是尽量不使用自动登录，但很多时候，在实际开发中，优质体验比不可预期的安全风险要更为优先。

​	如果决定提供自动登录功能，就应当限制cookie登录时的部分执行权限。例如，修改密码、修改邮箱（防止找回密码）、查看隐私信息（如完整的手机号码、银行卡号等）等，校验登录密码或设置独立密码来做二次校验也是不错的方案。

## 注销登录

​	HttpSecurity内的logout()方法以一个LogoutConfigurer作为配置基础，创建一个用于注销登录的过滤器。

```java
	@Override
	public void configure(H http) throws Exception {
		LogoutFilter logoutFilter = createLogoutFilter(http);
		http.addFilter(logoutFilter);
	}
/**
	 * Creates the {@link LogoutFilter} using the {@link LogoutHandler} instances, the
	 * {@link #logoutSuccessHandler(LogoutSuccessHandler)} and the
	 * {@link #logoutUrl(String)}.
	 * @param http the builder to use
	 * @return the {@link LogoutFilter} to use.
	 */
	private LogoutFilter createLogoutFilter(H http) {
		this.logoutHandlers.add(this.contextLogoutHandler);
		this.logoutHandlers.add(postProcess(new LogoutSuccessEventPublishingLogoutHandler()));
		LogoutHandler[] handlers = this.logoutHandlers.toArray(new LogoutHandler[0]);
		LogoutFilter result = new LogoutFilter(getLogoutSuccessHandler(), handlers);
		result.setLogoutRequestMatcher(getLogoutRequestMatcher(http));
		result = postProcess(result);
		return result;
	}
```

​	它默认注册了一个/logout路由，用户通过访问该路由可以安全地注销其登录状态，包括使HttpSession失效、清空已配置的Remember-me验证，以及清空SecurityContextHolder，并在注销成功之后重定向到/login?logout页面。
​	如有必要，还可以重新配置。	

![](https://pic.imgdb.cn/item/60bb09c58355f7f718d4a3e3.jpg)

![](https://pic.imgdb.cn/item/60bb09eb8355f7f718d5921c.jpg)

# 6. 会话管理

## 理解会话

​	会话（session）就是无状态的 HTTP 实现用户状态可维持的一种解决方案。

​	尽管cookie非常有用，但有时用户会在浏览器中禁用它，可能是出于安全考虑，也可能是为了保护个人隐私。在这种情况下，基于cookie实现的sessionId自然就无法正常使用了。因此，有些服务还支持用URL重写的方式来实现类似的体验，例如：

```
http://blurooo.com;jessionid=xxx
```

​	URL重写原本是为了兼容禁用cookie的浏览器而设计的，但也容易被黑客利用。黑客只需访问一次系统，将系统生成的sessionId提取并拼凑在URL上，然后将该URL发给一些取得信任的用户。只要用户在session有效期内通过此URL进行登录，该sessionId就会绑定到用户的身份，黑客便可以轻松享有同样的会话状态，完全不需要用户名和密码，这就是典型的会话固定攻击。

## 防御会话固定攻击

​	防御会话固定攻击的方法非常简单，只需在用户登录之后重新生成新的session即可。在继承WebSecurityConfigurerAdapter时，Spring Security已经启用了该配置。

sessionManagement是一个会话管理的配置器，其中，防御会话固定攻击的策略有四种：

* none：不做任何变动，登录之后沿用旧的session。
* newSession：登录之后创建一个新的session。
* migrateSession：登录之后创建一个新的session，并将旧的session中的数据复制过来。
* changeSessionId：不创建新的会话，而是使用由Servlet容器提供的会话固定保护。
  默认已经启用migrateSession策略，如有必要，可以做出修改。

```java
http.sessionManagement()
	.sessionFixation().none();
```

​	在 Spring Security 中，即便没有配置，也大可不必担心会话固定攻击。这是因为Spring Security的HTTP防火墙会帮助我们拦截不合法的URL，当我们试图访问带session的URL时，实际上会被重定向到类似如图6-1所示的错误页。

​	具体细节可以翻看Spring Security源码，该部分内容在StrictHttpFirewall类中实现。

## 会话过期

​	除防御会话固定攻击外，还可以通过Spring Security配置一些会话过期策略。例如，会话过期时跳转到某个URL。

```java
.sessionManagement()
    invalidSessionUrl("/session/invalid")
```

或者完全自定义过期策略。

![](https://pic.imgdb.cn/item/60bc2ef88355f7f7186beec4.jpg)

​	默认情况下，只要该会话在30分钟内没有活动便会失效，失效后再尝试发起访问，将会得到应答。

​	当然，我们可以手动修改会话的过期时间。

```properties
# 单位为秒
server.session.timeout=60
```

​	会话的过期时间最少为1分钟，所以即便设置小于60秒也会被修正为1分钟，这属于Spring Boot的配置策略。

## 会话并发控制

```java
.sessionManagement()
    .maximumSessions(1)
```

​	maximumSessions 用于设置单个用户允许同时在线的最大会话数，如果没有额外配置，那么新登录的会话会踢掉旧的会话。

​	具体的实现细节在ConcurrentSessionControlAuthenticationStrategy类中可以看到。

​	如果我们需要在会话数达到最大数时，阻止新会话建立，而不是踢掉旧的会话，则可以像下面这样配置。

```java
.sessionManagement()
    .maximumSessions(1)
    .maxSessionPreventLogin(true);
```

​	首先尝试将已登录的旧会话注销（通常是访问/logout），理论上应该可以继续登录了，但很遗憾，Spring Security依然提示我们超过了最大会话数。事实上，除非重启服务，否则该用户将很难再次登录系统。这是因为Spring Security是通过监听session的销毁事件来触发会话信息表相关清理工作的，但我们并没有注册过相关的监听器，导致Spring Security无法正常清理过期或已注销的会话。

​	在Servlet中，监听session相关事件的方法是实现HttpSessionListener接口，并在系统中注册该监听器。Spring Security在HttpSessionEventPublisher类中实现HttpSessionEventPublisher接口，并转化成Spring的事件机制。

​	在Spring事件机制中，事件的发布、订阅都交由Spring容器来托管，我们可以很方便地通过注册bean的方式来订阅关心的事件。
​	显然，前面的配置无法正常工作是因为缺少了事件源（会话清理订阅的Spring事件，而非原生的HttpSessionEvent），另外，还需要把HttpSessionEventPublisher注册到IoC容器中，才能将Java事件转化成Spring事件。

![](https://pic.imgdb.cn/item/60bc328a8355f7f718965e21.jpg)

​	我们已经将验证的数据源切换到了数据库。登录功能正常，但会话并发控制似乎并不起作用，这是怎么回事呢？要想理解这个问题，还需要深入了解Spring Security关于会话管理的设计方式。
​	Spring Security为了实现会话并发控制，采用会话信息表来管理用户的会话状态，具体实现见SessionRegistryImpl类。

​	值得一提的是，principals采用了以用户信息为key的设计。我们知道，在hashMap中，以对象为key必须覆写hashCode和equals两个方法（具体原因可以查阅hashMap的设计），但我们实现UserDetails时并没有这么做，这导致同一个用户每次登录注销时计算得到的key都不相同，所以每次登录都会向principals中添加一个用户，而注销时却从来不能有效移除。在这种情况下，不仅达不到会话并发控制的效果，还会引发内存泄露。
​	

  

理解了缘由之后，要避开陷阱自然不难，为自定义的用户类覆写hashCode和equals两个方法即可。

## 集群会话的缺陷

​	从容量上来说，服务器内存有限，除了系统正常运行的消耗，留给session的空间不多，当访问量增大时，内存就会捉襟见肘。从稳定性上来说，session依赖于内存，而内存并非持久性存储容器，就算服务器本身是可靠的，但当部署在上面的服务停止或重启时，也会导致所有会话状态丢失。当然，这两个缺点还只是体验性缺陷，并不足以影响可用性，在单机部署时为了节省精力忽略这两个问题也是可以的。但当我们的系统采用集群部署时，就会有更多关于可用性的问题需要考虑。

​	大部分的集群部署会采用类似图6-6所示的网络结构。

![](https://pic.imgdb.cn/item/60bc52c48355f7f718371621.jpg)

​	正常来说，在集群环境下，同个用户的请求可能会被分发到不同的服务器上，假如登录操作是在SERVER1完成的，即SERVER1缓存了用户的登录状态，但SERVER2和SERVER3并不知情，如果该用户的后续操作被分配到了SERVER2或SERVER3上，这时就会要求该用户重新登录，这就是典型的会话状态集群不同步问题。

## 集群会话的解决方案

解决集群会话的常见方案有三种：

* session保持
* session复制
* session共享



​	session保持也叫粘滞会话（Sticky Sessions），通常采用IP哈希负载策略将来自相同客户端的请求转发至相同的服务器上进行处理。session保持虽然避开了集群会话，但也存在一些缺陷。例如，某个营业部的网络使用同个IP出口，那么使用该营业部网络的所有员工实际的源IP其实是同一个，在IP哈希负载策略下，这些员工的请求都将被转发到相同的服务器上，存在一定程度的负载失衡。

​	session复制是指在集群服务器之间同步session数据，以达到各个实例之间会话状态一致的做法。但毫无疑问，在集群服务器之间进行数据同步的做法非常不可取，尤其是在服务器实例很多的情况下，任何变动都需要其他所有实例同步，不仅消耗数据带宽，还会占用大量的资源。

​	相较于前两种方案，session 共享则要实用得多。session 共享是指将 session 从服务器内存抽离出来，集中存储到独立的数据容器，并由各个服务器共享，如图6-7所示。

​	由于所有的服务器实例单点存取session，所以集群不同步的问题自然也就不存在了，而且独立的数据容器容量相较于服务器内存要大得多。另外，与服务本身分离、可持久化等特性使得会话状态不会因为服务停止而丢失。当然，session共享并非没有缺点，独立的数据容器增加了网络交互，数据容器的读/写性能、稳定性以及网络I/O速度都成为性能的瓶颈。基于这些问题，尽管在理论上使用任何存储介质都可以实现session共享，但在内网环境下，高可用部署的Redis服务器无疑为最优选择。Redis基于内存的特性让它拥有极高的读/写性能，高可用部署不仅降低了网络I/O损耗，还提高了稳定性。

![](https://pic.imgdb.cn/item/60bc53508355f7f7183e056a.jpg)

## 整合Spring Session解决集群会话问题

​	Spring Session支持多种类型的存储容器，包括Redis、MongoDB等。

**依赖**

![](https://pic.imgdb.cn/item/60bc55388355f7f71852cdfa.jpg)

![](https://pic.imgdb.cn/item/60bc55738355f7f71855679c.jpg)

​	![](https://pic.imgdb.cn/item/60bc55988355f7f71856a3e8.jpg)

​	Spring Session与Spring Security的整合非常简单，如果要验证是否生效，则可以在不同端口上启动多个实例，并在各自端口实例上登录同个账号。若出现挤占，则代表配置生效。登录成功之后，Redis会存储用户session相关的数据，默认命名为spring:session。

# 7. 密码加密

## 密码加密演进

* MD5
  * 反查表。
  * 彩虹表。
    * 彩虹表没有直接存储摘要值与原文的一一对应关系，而是存储了一个时间—空间平衡的散列链集。
    * 尽管在彩虹表面前散列加密是脆弱的，但也并非束手无策，因为彩虹表也有弱点，加盐是最简单有效地防御彩虹表的手段。



​	所谓加盐加密，是指在计算摘要值之前，为原文附上额外的随机值，以达到扰乱目的的加密方式。具体实现方法并不固定。例如，以用户名作为盐值。

![](https://pic.imgdb.cn/item/60bc58148355f7f718690eb7.jpg)

​	或者将随机盐值直接挂在加密后的摘要值上，省去额外的存储字段。

![](https://pic.imgdb.cn/item/60bc58df8355f7f7186de587.jpg)

​	反查表和彩虹表都是通过对散列值或散列值的加工从而逆向推导出密码的，加盐等同于阻断了散列值与密码的直接对应关系，使得逆向推导的破解方式不再具有威慑力，但最经典的穷举法依然会带来威胁。随机盐值通常在实际存储密码时会同步存储，即当发生“拖库”时，盐值也很难幸免。在正向穷举时，盐值的阻碍非常小。例如，穷举1至8位的密码，只需相应附上盐值即可，不用在乎该盐值有多长，而且其计算耗时在当前来说是完全可以接受的，尤其在算力强的计算平台上（计算能力随着时间的推移会越来越强，意味着暴力破解当前的散列密码会越来越容易）。

​	我们需要一种慢的加密手段，慢到暴力破解无法忍受，慢到超级计算机也无可奈何，BCrypt正是这样一种算法。

![](https://pic.imgdb.cn/item/60bc59558355f7f7186f4f0b.jpg)

​	其中，2a表明了算法的版本。2a版本加入了对非ASCII字符以及空终止符的处理，类似的版本号还有2x、2y、2b 等，都是在前面版本的基础上修复了某些缺陷或新增了某些特性。12是一个成本参数，它表明该密文需要迭代的次数。12是指2的12次方，即4096次。BCrypt依靠此参数来限制算法的速度。成本参数的理想取值是既让暴力破解无法忍受，又不会显著影响用户的实际体验（未来，计算能力会越来越强，成本参数也应相应调整）。2a$12$之后的前22位是该密文的随机盐值，最后31位为真正的散列值。

## Spring Security的密码加密机制

​	Spring Security内置了密码加密机制，只需使用一个PasswordEncoder接口即可。

![](https://pic.imgdb.cn/item/60bc5b848355f7f71879e54a.jpg)

​	PasswordEncoder接口定义了encode和matches两个方法，当用数据库存储用户密码时，加密过程用encode方法，matches方法用于判断用户登录时输入的密码是否正确。
此外，Spring Security 还内置了几种常用的 PasswordEncoder 接口，例如，StandardPasswordEncoder中的常规摘要算法（SHA-256等）、BCryptPasswordEncoder加密，以及类似BCrypt的慢散列加密Pbkdf2PasswordEncoder等，官方推荐使用BCryptPasswordEncoder。

![](https://pic.imgdb.cn/item/60bc5c0f8355f7f7187ce6d5.jpg)

​	在声明一个PasswordEncoder的bean之后，Spring Security会自动应用。重启服务，发现user这个用户已经无法通过密码123来登录了。实际上需要什么密码没有人知道，但admin可以使用“blurooo”来登录，验证了我们的配置是有效的。

​	通常情况下，在新系统中使用BCrypt加密不需要考量太多，但老系统由于存在大量旧数据，草率接入会导致老用户无法登录，这种情况该怎么解决呢？很简单，我们自己实现一个PasswordEncoder并继承BCryptPasswordEncoder即可。

![](https://pic.imgdb.cn/item/60bc5c3c8355f7f7187d9a8e.jpg)

​	在这个PasswordEncoder中，只有当密码不是BCrypt密文时，才启用自定义的匹配逻辑，其余还是沿用原来的方案，即可轻松达到兼容的目的。
​	再进一步，如果我们不仅想要兼容，还想将不安全的旧密码无缝修改成BCrypt密文，该如何操作呢？这是个很好的问题。如果旧密码都是未经任何加密的明文，也许“跑库”修改是非常好的一种选择，但并非所有系统都有这么理想的状态。假如旧密码都是被散列加密过的，那么可以用下面两种方法解决这个问题：

* 使用增量更新的方法。当用户输入的密码正确时，判断数据库中的密码是否为BCrypt密文，如果不是，则尝试使用用户输入的密码重新生成BCrypt密文并写回数据库。
* 以旧的加密方案作为基础接入BCrypt加密。例如，旧的方案是MD5加密，即数据库中的所有密码都是 MD5形式的密码，那么直接把这些密码当作明文，先“跑库”生成 BCrypt 密文，再使用 encode 和 matches 两个方法在执行 BCrypt 加密之前都先用MD5运算一遍即可。



​	密码安全对系统来说，从来都是一个不小的挑战，即便实现了安全存储，仍然有很多的途径泄露用户密码。用户密码在提交到系统之前就被盗取的案例数不胜数，如挂木马或是抓包等。很多时候，用户往往无法准确识别陷阱，而我们能做的，就是尽可能地做好安全措施。例如，采用HTTPS安全信道，甚至是开发银行级别的安全控件，帮助用户检测系统环境是否正常。

# 8. 跨域与CORS

## 认识跨域

​	不仅不同站点间的访问存在跨域问题，同站点间的访问可能也会遇到跨域问题，只要请求的URL与所在页面的URL首部不同即产生跨域，例如：

* 在http://a.baidu.com下访问https://a.baidu.com资源会形成协议跨域。
* 在a.baidu.com下访问b.baidu.com资源会形成主机跨域。
* 在a.baidu.com:80下访问a.baidu.com:8080资源会形成端口跨域。



​	浏览器解决跨域问题的方法有多种，包括JSONP、Nginx转发和CORS等。其中，JSONP和CORS需要后端参与。

## 实现跨域之JSONP


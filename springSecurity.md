# 保护Spring

## 自动配置Spring Security

### 启用Spring Security

在项目的pom.xml文件中，添加如下的<dependency>条目：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

在载入secrity后启动spring项目，在日志中会打印这样的东西。

![image-20210403103834750](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20210403103834750.png)

现在访问网页都需要输入这样的一个密码才可以访问。

![image-20210403104533474](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20210403104533474.png)

看起来还蛮不错的。但是这样的效果不是我们想要的。我们接下来去配置我们的security。

### 配置Spring Security

![image-20210403105058370](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20210403105058370.png)

![image-20210403105103659](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20210403105103659.png)

首先我们创建这样的一个配置类。

#### 认证的四种方式

spring security为我们提供了4种可选的方案： `这里是认证的四种方式`

	1. 基于内存的用户储存
	2. 基于JDBC的用户存储
	3. 以LDAP作为后端的用户存储
	4. 自定义用户详情服务

不管是那种方案，我们都需要通过覆盖`WebSecurityConfigurerAdapter`基础配置类中的`configure()`方法来进行配置。

```java
/*
认证的规则
*/
@Override 
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    super.configure(auth);
}
```

> 基于内存的用户存储

```java
 @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .passwordEncoder(new BCryptPasswordEncoder())
                .withUser("root")
                .password(new BCryptPasswordEncoder().encode("root"))
                .authorities("ROLE_USER")
                .and()
                .withUser("woody")
                .password(new BCryptPasswordEncoder().encode("123456"))
                .authorities("ROLE_USER");
    }
```

这种基于内存的用户储存，用了建造者风格的接口设定了账号和密码，对应的权限。

这种比较时候角色比较少并且非常简单的时候使用，并且几乎不会发生变化的时候使用。

必须要有个默认的加密方式，不然会爆出这样的错误：There is no PasswordEncoder mapped for the id "null"；

--------------

> 自定义用户详情储存

* 我这里是使用了UserDetails的方式来验证。

* 首先让你的用户pojo实体类实现`UserDetails`方法。

* 里面带`isxxxx`的是判断用户当前的状态的。

* 然后`getPassword()`和`getUsername()`是获取用户的账号和密码的。

这里注意下。还有一个方法。

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    List<GrantedAuthority> auths = new ArrayList<>();
    auths.add(new SimpleGrantedAuthority(this.role));
    return auths;
}
```

`这个方法是获取当前用户拥有那些权限的。`

一般一个用户可以拥有多个权限。 多少个权限的实体就添加多少个。我这里偷懒了就添加了一个。

然后就是实现一个 可以根据用户信息来查找 实现了`UserDetails`对象的方法.

```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userMapper.queryByname(username);
    System.out.println(user);
    return Optional
            .ofNullable(user)
            .orElseThrow(() -> new UsernameNotFoundException("用户名不存在"));
}
```

就是 mvc 里面的 service层，多了一个实现 。

* 这里就是需要你的service层 实现`UserDetailsService`就可以了。如果用户不存在则返回一个异常。

- 这个异常是不会被捕获的，我们可以实现自己的异常，或者是直接抛出一个可以被捕获的异常。

#### http授权

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    
}
```

这个方法接受一个HttpSecurity对象，能够用来配置在Web级别该如何处理安全性的问题。我们可以使用它实现如下的功能：

* 在为某个请求提供服务之前，需要预先满足特定的条件；
* 配置自定义的登录页
* 支持用户退出应用
* 预防跨站请求伪造

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .antMatchers("/add", "/delete")
            .hasRole("USER")
            .antMatchers("/", "/index").permitAll();
    //开启去往登录的页面
    http.formLogin();
    
}
```

这里我们给访问`/add`,`/delete`页面的用户必须要ROLE_USER权限才可以进入了，不然就会进入要求登录的页面。

#### 跨站请求信息伪造

```java
  @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/add", "/delete")
                .hasRole("USER")
                .antMatchers("/", "/**").permitAll()
                .and()
                //开启去往登录的页面
                .formLogin()
                .and()
                //开启退出
                .logout()
                .logoutSuccessUrl("/")
                //关闭跨域请求防御
                .and()
                .csrf()
                .disable();
    }
```

一般不需要关闭这个，如果是使用的jsp或者thymeleaf的模板引擎，会自动渲染一个表单隐藏域在表单中。


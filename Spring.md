#  Spring

## 学习Spring前需要了解的知识与思想

#### 1、控制反转 IoC （Inversion of Control）

##### 为什么需要控制反转呢？

- 可以用来减低计算机代码之间的耦合度
- 将控制权将程序员手中交到用户手上
- 降低代码维护的成本

##### 原理是什么？

-  通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

  > IoC是一个很大的概念,可以用不同的方式实现。其主要形式有两种：
  >
  > - **依赖查找：**容器提供回调接口和上下文条件给组件。EJB和Apache Avalon  都使用这种方式。这样一来，组件就必须使用容器提供的API来查找资源和协作对象，仅有的控制反转只体现在那些回调方法上（也就是上面所说的  类型1）：容器将调用这些回调方法，从而让应用代码获得相关资源。
  > - **依赖注入：**组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系。容器全权负责的组件的装配，它会把符合依赖关系的对象通过JavaBean属性或者[构造函数](https://baike.baidu.com/item/构造函数)传递给需要的对象。通过JavaBean属性注射依赖关系的做法称为设值方法注入(Setter Injection)；将依赖关系作为构造函数参数传入的做法称为[构造器](https://baike.baidu.com/item/构造器)注入（Constructor Injection） [2] 

  spring就是采用的依赖注入（DI）的方式

![1604300161902](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\1604300161902.png)

# 初次使用spring

### Spring文档

[英文官方文档](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)

[中文文档查看](https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference)

#### 环境配置

1. 导入jar包 刚刚开始使用

   - 只要导入了Spring webmvc 的jar包就可以使用了

   - ```xml
     <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>5.2.9.RELEASE</version>
     </dependency>
     ```

2. 配置元数据文件

   - 通过xml方式来配置元数据文件

   - ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.springframework.org/schema/beans
      https://www.springframework.org/schema/beans/spring-beans.xsd">
             <bean id="..." class="...">  
             <!--
             id属性是标识单个bean定义的字符串。 
             -->
             <!--
             class属性定义bean的类型并使用完全限定的类名。
             -->    
             </bean>
     </beans>
     
     
     ```

   - 如果没有任何其他配置在元配置中写的bean对象会在spring启动的时候就创建一个对象实例，并且是采用的单例生成模式。

   - bean标签下面的属性 

     - Class
     - Name
     - Scope（作用域）
       - singleton（默认的单例模式）
       -  prototype （原型模式，就像new对象？）
       - request
         - 将单个bean定义限定为单个HTTP请求的生命周期。也就是说，每个HTTP请求都有一个在单个bean定义后面创建的bean实例。仅在支持web的Spring应用程序上下文中有效。
       - session
       - application
       - websocket
     -  Constructor arguments 
     -  Properties 
     -  Autowiring mode 
     -  Lazy initialization mode 
  -  Initialization method 
     
   -  Destruction method 
   
   - bean的有参构造函数下的情况
   
     - 调用别的bean为参数:
     
     - ```java
       package x.y;
       
       public class ThingOne {
       
           public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
               // ...
           }
       }
       ```
     
     - ```xml
       <beans>
           <bean id="beanOne" class="x.y.ThingOne">
               <constructor-arg ref="beanTwo"/>
               <constructor-arg ref="beanThree"/>
           </bean>
       
           <bean id="beanTwo" class="x.y.ThingTwo"/>
       
           <bean id="beanThree" class="x.y.ThingThree"/>
       </beans>
       ```
     
     - 通过参数类型来填
     
     - ```xml
       <bean id="exampleBean" class="examples.ExampleBean">
           <constructor-arg type="int" value="7500000"/>
           <constructor-arg type="java.lang.String" value="42"/>
       </bean>
       ```
     
     - 通过参数位置来填
     
     - ```xml
       <bean id="exampleBean" class="examples.ExampleBean">
           <constructor-arg index="0" value="7500000"/>
           <constructor-arg index="1" value="42"/>
       </bean>
       ```
     
     - 通过参数名来填
     
     - ```xml
       <bean id="exampleBean" class="examples.ExampleBean">
           <constructor-arg name="years" value="7500000"/>
           <constructor-arg name="ultimateAnswer" value="42"/>
       </bean>
       ```
     
     - ```java
       package examples;
       
       public class ExampleBean {
       
           // Fields omitted
       
           @ConstructorProperties({"years", "ultimateAnswer"})
           public ExampleBean(int years, String ultimateAnswer) {
               this.years = years;
               this.ultimateAnswer = ultimateAnswer;
           }
       }
       ```
   
   - 通过setter方法来注入属性
   
     - ```xml
       <bean id="exampleBean" class="examples.ExampleBean">
           <property name="beanOne">
               <ref bean="anotherExampleBean"/>
           </property>
           <property name="beanTwo" ref="yetAnotherBean"/>
           <property name="integerProperty" value="1"/>
       </bean>
       <bean id="anotherExampleBean" class="examples.AnotherBean"/>
       <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
       ```
     
     - ```java
       public class ExampleBean {
       
           private AnotherBean beanOne;
       
           private YetAnotherBean beanTwo;
       
           private int i;
       
           public void setBeanOne(AnotherBean beanOne) {
               this.beanOne = beanOne;
           }
       
           public void setBeanTwo(YetAnotherBean beanTwo) {
               this.beanTwo = beanTwo;
           }
       
           public void setIntegerProperty(int i) {
               this.i = i;
           }
       }
       ```
   
   - 导入别人写的beans配置文件（**需要再同一路径下**）
   
   - ```xml
     
     <beans>
         <import resource="services.xml"/>
         <import resource="resources/messageSource.xml"/>
         <import resource="/resources/themeSource.xml"/>
     
         <bean id="bean1" class="..."/>
         <bean id="bean2" class="..."/>
     </beans>
     
     
     ```
     
   - 导入别的配置文件会按照导入的顺序重写（后导入会覆盖前面的，如果存在同名）
     - 这种方式多用于团队开发的时候整合beans
     - 如果可以也不要写../来加载文件，这样会导致应用程序对程序之外的文件产生依赖，所以我们需要将配置文件放在同一文件夹下或者同一个resources文件夹下	
#### 获取beans对象并使用


   1. 通过 ClassPathXmlApplicationContext使用
      1. 想要获取beans对象我们需要获取一个``ApplicationContext``工厂
         
         ```java
            // create and configure beans
            ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
         ```
         
         - 可以注意到我们可以通过这种方法加载多个配置文件来加载多个bean。
         
      2. 通过`ApplicationContext`工厂我们来获取我们的实例化的对象
      
            - ```java
              // retrieve configured instance
              PetStoreService service = context.getBean("petStore", PetStoreService.class);
              ```
              
              - 我们获取这个bean对象的时候，默认是通过无参构造函数进行创建的，如果需要通过有参构造来创建对象，需要我们再xml配置文件中编写。
              - 基于Setter的DI是由容器在调用无参数构造函数或无参数静态工厂方法实例化bean后调用bean上的Setter方法来实现的。
              
              
      
      3. 使用我们的对象方法
      
            - ```java
              // use configured instance
              List<String> userList = service.getUsernameList();
              ```
            

#### c和p命名空间的使用

使用这种命名空间可以方便的

（在配置文件中声明的p命名空间的地址可能发生改变所以导致了在idea中会报错，但是还是可以使用）

1. 使用p命名空间（通过set注入的方式）

   - ```xml
     xmlns:p="http://www.springframework.org/schema/p"
     ```

     需要在beans的属性中增加这个就可以使用了

   - ```xml
     <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
         destroy-method="close"
         p:driverClassName="com.mysql.jdbc.Driver"
         p:url="jdbc:mysql://localhost:3306/mydb"
         p:username="root"
         p:password="misterkaoli"/>
     ```

   - 效果同等与

     ```xml
     <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
     <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
     <property name="username" value="root"/>
     <property name="password" value="misterkaoli"/>
     ```

2. 使用c命名空间（使用构造器的方式）

   - ```xml
     xmlns:c="http://www.springframework.org/schema/c"
     ```

   - ```xml
     <!-- 传统的通过构造器的方式来创建对象 -->
     <bean id="beanOne" class="x.y.ThingOne">
     <constructor-arg name="thingTwo" ref="beanTwo"/>
     <constructor-arg name="thingThree" ref="beanThree"/>
     <constructor-arg name="email" value="something@somewhere.com"/>
     </bean>
     
     <!-- 通过c命名空间来创建对象 -->
     <bean id="beanOne" class="x.y.ThingOne" 
     c:thingTwo-ref="beanTwo"
     c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>
     ```

#### 自动装配

使用自动装配 ，在xml配置文件中给bean对象 添加autowire属性

```xml
<bean id="people" class="com.kuang.pojo.People"
      autowire="byName">
</bean>
```

通过设置byName可以实现自动装配相同名字的对象。

还有一种是使用byType方式会寻找对应的相同的类

# spring中使用到的注解

需要准备： 

1. 导入约束 。**context约束** 

   - ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
     xsi:schemaLocation="http://www.springframework.org/schema/beans
     https://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/context
     https://www.springframework.org/schema/context/spring-context.xsd">
         <context:annotation-config/>
     </beans>
     ```

2. 导入依赖包

   - aop 的包

3. 指定要扫描的包，这个包下的注解就会生效

   - ```xml
     <context:component-scan
     base-package="com.kuang.pojo"
     />
     ```

#### @Autowired 

spring中通过注解方式来自动装配

1. 作用域

   - 属性上方
   - set方法
   - 一般写在属性上方（可以忽略set方法使用）

2. 使用前提，这个对象必须要在IOC容器中存在并且名字相同。

3. 内部使用required = false 说明这个对象可以为null，否则不允许为空

   - ```java
     @Autowired(required=false)
     private Cat cat;
     ```

4. 优先使用byType方式来匹配

#### @Nullable

字段标记了这个注解可以，说明这个字段可以为``null``

#### @Qualifier

1. 作用域

   - 属性上方

2. 作用

   - 指定装配的值的名字；

3. 实例

   - ```java
     @Qualifier("cat233")
     private Cat cat;
     ```

#### ~~@Required~~（springframework5.1开始弃用）

1. 作用域
   - set方法
2. 作用
   - 说明这个属性是必须要的不能为空。

#### @Resource

1. 作用域
   - 属性上面
2. 作用
   - 自动装配，但是没有
3. 参数
   - name 指定装配的bean对象的名字

#### @Component

1. 作用域

   - 类上方

2. 作用

   - 使用注解的方式来生成一个bean对象，说明这个对象被spring管理了

3. 实例

   - 等价于

   ```xml
   <bean id="user" class="com.kuang.pojo.User"/>
   ```

4. 衍生注解

   1. dao层注解
      - @Repository   -和Component一样的。
   2. service层
      - @Service
   3. Controller层
      - @Controller

#### @Value

1. 作用域
   - set方法
2. 作用
   - 给属性注入默认属性

#### @Scope

 	1. 作用域
 	  - 类上方
 	2. 作用

- 类的作用域设定
3. 类

#### @RequestMapping

作用在controller里面的方法上，决定接收哪一种类型的url请求

value  填写路径

Method 填写请求的方法 RequestMethod.Get....

#### @GetMapping

这个是相当于 @RequestMapping(value ="",method=RequestMethod.Get)

#### @ResponseBody

作用在controller的方法上，这样这个方法就不会走视图解析器了，直接返回，一般用于json等消息传递。

#### @RestController 

作用在controller类上面，这个类里面的所有方法只会返回字符串

#### @EnableScheduling

说明这个类是一个定时任务类

#### @Scheduled

作用在定时方法上，进行定时任务.

```
@Scheduled(cron = "0 0 5 * * ?")
```

> “0 0 12 * * ?”                每天中午12点触发
>
> “0 15 10 ? * *”               每天上午10:15触发
>
> “0 15 10 * * ?”               每天上午10:15触发
>
> “0 15 10 * * ? *”             每天上午10:15触发
>
> “0 15 10 * * ? 2005”          2005年的每天上午10:15 触发
>
> “0 * 14 * * ?”                在每天下午2点到下午2:59期间的每1分钟触发
>
> “0 0/5 14 * * ?”              在每天下午2点到下午2:55期间的每5分钟触发
>
> “0 0/5 14,18 * * ?”           在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
>
> “0 0-5 14 * * ?”              在每天下午2点到下午2:05期间的每1分钟触发
>
> “0 10,44 14 ? 3 WED”          每年三月的星期三的下午2:10和2:44触发
>
> “0 15 10 ? * MON-FRI”         周一至周五的上午10:15触发
>
> “0 15 10 15 * ?”              每月15日上午10:15触发
>
> “0 15 10 L * ?”               每月最后一日的上午10:15触发
>
> “0 15 10 ? * 6L”              每月的最后一个星期五上午10:15触发
>
> “0 15 10 ? * 6L 2002-2005”    2002年至2005年的每月的最后一个星期五上午10:15触发
>
> “0 15 10 ? * 6#3”             每月的第三个星期五上午10:15触发
>
> 0 6 * * *                     每天早上6点
>
> 0 /2 * *                      每两个小时
>
> 0 23-7/2，8 * * *             晚上11点到早上8点之间每两个小时，早上八点
>
> 0 11 4 * 1-3                  每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点

####  @ConfirgurationProperties

从配置文件中导入到类中

#### @Validated

数据校验

jsr303校验![img](https://upload-images.jianshu.io/upload_images/3145530-8ae74d19e6c65b4c?imageMogr2/auto-orient/strip|imageView2/2/w/654)

![img](https://upload-images.jianshu.io/upload_images/3145530-10035c6af8e90a7c?imageMogr2/auto-orient/strip|imageView2/2/w/432)



# 在spring中使用mybatis

### 	引入包

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

### 编写一个Spring-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">   
</beans>
```

#### 配置数据源

```xml
<!--       数据源配置 --> 
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">        			<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url"  value="jdbc:mysql://localhost:3306/UserDataBase?											useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>   
</bean>    
```
#### 编写 dao层和实现的mapper

一个xml和一个dao层

#### 配置sqlSessionFactory

```xml
<!--    sqlSessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>   
    <!--        连接mybaits配置文件-->   
    <property name="configLocation" value="classpath:mybatis-config.xml"/>  
    <!--        mapper文件配置进来-->  
    <property name="mapperLocations" value="classpath:com/chen/mapper/StackroomMapper.xml"/>
</bean>
```

#### 编写一个service实现类

偷懒的话就继承SqlSessionDaoSupport

```java
public class StackroomMapperImpl implements StackroomMapper {
    //使用一个SqlSessionTemplate 来获取操作，SqlSessionTemplate是线程安全的
    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    public List<Stackroom> getStack() {
        StackroomMapper mapper = sqlSession.getMapper(StackroomMapper.class);
        return mapper.getStack();
    }
}
```

#### 让spring来管理sqlSession

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">    
    <!--       使用构造器注入-->   
    <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
```

#### 让Spring来管理service层对象

```xml
<bean id="StackroomMapper" class="com.chen.service.StackroomMapperImpl">    
    <property name="sqlSession" ref="sqlSession"/>
</bean>
```

#### 获取service对象

```java
//传入的class类是接口的类  面向接口编程
StackroomMapper stackroomMapper = context.getBean("StackroomMapper", StackroomMapper.class);
```

#### 开启事务

标准配置

要开启 Spring 的事务处理功能，在 Spring 的配置文件中创建一个 `DataSourceTransactionManager` 对象：

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <constructor-arg ref="dataSource" />
</bean>
```

 注意：为事务管理器指定的 `DataSource` **必须**和用来创建 `SqlSessionFactoryBean` 的是同一个数据源，否则事务管理器就无法工作了。 

#### 开启事务通知

```xml
<!--开启事务通知-->   
<tx:advice id="txAdvice" transaction-manager="transactionManager">       
<!--给那些方法配置事件-->       
    <tx:attributes>           
        <tx:method name="*"/>        
    </tx:attributes>  
</tx:advice>
<!--    配置事务aop-->   
<aop:config>     
    <aop:pointcut id="txPont" expression="execution(* com.chen.mapper.*.*(..))"/>       
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPont"/>    
</aop:config>
```

# springmvc

### 1. 导入jar包

spring mvc的jar包

```xml
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-webmvc</artifactId>
<version>5.2.9.RELEASE</version>
</dependency>
```

jstl (可选)

```xml
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>jstl</artifactId>
<version>1.2</version>
</dependency>
```

jsp-api（可选）

```xml
<dependency>    <groupId>javax.servlet.jsp</groupId>    <artifactId>jsp-api</artifactId>    <version>2.2</version></dependency>
```

servlet-api（必须）

```xml
<dependency>    <groupId>javax.servlet</groupId>    <artifactId>servlet-api</artifactId>    <version>2.5</version></dependency>
```

jackson (可选)

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.3</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.11.3</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-annotations -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.11.3</version>
</dependency>

```



### 2. mvc controller（底层原理）

![1606273347037](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\1606273347037.png)

1. DispatchercherServlet 表示前端控制器，是整个SpringMvc的控制中心。
2. HandlerMapping 是处理器映射。DispatcherServlet来调用，HandlerMapping，HandlerMapping根据请求的url查找Handler。
3. HandlerExecution表示具体的Handler，主要作用是根据url查找控制器。传回DispatcherServlet
4. HandlerAdapter表示处理器适配器，其安装特定的规则去执行Handler。
5. Handler让具体的Controller执行。
6. Controller将具体的执行信息返回给HandlerAdapter 如ModelAndView
7. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。
8. DispatcherServlet调用视图解析器ViewResolver来解析HandlerAdapter传递的逻辑视图名
9. 视图解析器将解析的逻辑视图传给DispathcerServlet
10. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图
11. 最终展示。

### 3. 开始使用

1. 给项目添加web框架

2. 在project Structure项目里面添加lib目录并将依赖添加进来

3. 在web.xml中配置DispatcherServlet

   ```xml
   <!--    配置DispatcherServlet 请求分发器-->
   <servlet>
   <servlet-name>springmvc</servlet-name>
   <servlet-class>org.springframework.web.servlet.DispatcherServlet
   </servlet-class>
   <!--        绑定springmvc的配置文件-->
   <init-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>classpath:springmvc-servlet.xml</param-value>
   </init-param>
   <!--        启动级别为1-->
   <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
   <servlet-name>springmvc</servlet-name>
   <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

4. 编写springxml配置文件 例如 springmvc-servlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       xmlns:context="http://www.springframework.org/schema/context"       xmlns:mvc="http://www.springframework.org/schema/mvc"       xsi:schemaLocation="http://www.springframework.org/schema/beans       https://www.springframework.org/schema/beans/spring-beans.xsd       http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd       http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd"> 
    <!--    自动扫描包-->    
    <context:component-scan base-package="com.chen.controller"/>    <!--    让spring mvc不处理静态资源-->   
    <mvc:default-servlet-handler/>   
    <!--    支持mvc注解驱动-->   
    <mvc:annotation-driven/>  
    <!--视图解析器-->   
    <bean class="org.springframework.web.servlet.
                 view.InternalResourceViewResolver"
          id="viewResolver">      
        <property name="prefix" value="/WEB-INF/jsp/"/>        		         <property name="suffix" value=".jsp"/>   
    </bean>
</beans>
```

5. 在com.chen.controller 中创建一个controller

```java
@Controller
public 
class HelloController {    
    @GetMapping("/hello")    
    public String hello(Model model) {                                        model.addAttribute("msg", "123456");    
           return "text";    
      }
}
```

@controller 申明一下是一个controller 然后通过扫描包就可以获取到了

@GetMapping("") 接收get请求并且路径是这个的。

或者用 @RequestMapping(value="",RequestMethod.Get)

### json乱码问题配置

```java
@RequestMapping(path="/api/fileupload",produces = { "application/json;charset=UTF-8" })
///在方法上添加produces。
```

```xml
<   <!--    支持mvc注解驱动-->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                        <property name="failOnEmptyBeans" value="false"/>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

### 配置拦截器

让一个类实现HandlerInterceptor 

重写3个方法

```java
public class MyInterceptor implements HandlerInterceptor {
    
    //return false 表示拦截，return true表示进入到下一个拦截器。
//执行方法前的拦截
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }
//执行方法后的拦截
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }
//结束后的清理
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

在springxml配置中配置拦截器

```xml
<mvc:interceptors>
        <mvc:interceptor>
  <!--/**表示类请求下的所有方法-->
            <mvc:mapping path="/**"/>
            <!--我们写的拦截器-->
            <bean class="com.chen.controller.MyInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

### 请求转发的三种方式

```java
return "forward:/WEB_INF/pages/success.jsp";
```

```java
　request.getServletDispatcher("/WEB-INF/pages/success.jsp").forward(request,response);
```

```java
return "success";
//走视图解析器
```

### 从外部读取properties文件

方法一

```xml
<context:property-placeholder location="classpath:db.properties"/>
//location值为参数配置文件的位置，这里是在类路径下
<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
```

方法二

```xml
导入多个properties
 <bean
  class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
               <value>classpath:db.properties</value>
             </list>
        </property>
    </bean>
```

### 异常解析映射器

```xml
<!-- 1.在文件上传解析时发现异常，此时还没有进入到Controller方法中 -->
 
<bean id="exceptionResolver" class= "org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
<property name="exceptionMappings"> 
    <props>
<prop key= "org.springframework.web.multipart.MaxUploadSizeExceededException">
/error 
</prop>
<!-- 遇到MaxUploadSizeExceededException异常时，跳转到error.jsp页面 --> </props> 
    </property>
</bean>
```

# spring-AOP

[`AOP学习文章`](https://www.cnblogs.com/joy99/p/10941543.html)

### 0、使用到的注解

- `@Aspect`

  - 说明这个类是一个切面类

- `@Component`

  - 说明这个是一个组件会被spring给管理

- `@Before`

  - 在方法执行之前执行的事情

- `@After`

  - 在方法完成之后执行还未返回

- `@Poincut`

  - 切入点就是在什么方法上进行切入
  - 属性argNames是方法的参数属性按照顺序匹配

  

  

### 1、定义一个切面类

```java
@Aspect
@Component
public class BuyAspectj {
    @Before("execution(* com.chen.service.Ibuy.buy(..))")
    public void before(){
        System.out.println("走进了商店");
    }
}

```

#### execution的范围![img](https://img2018.cnblogs.com/blog/758949/201905/758949-20190529225530124-1714809272.png) 

####  在execution里面写的东西![img](https://img2018.cnblogs.com/blog/758949/201905/758949-20190529225507103-314276426.png) 

```java
execution(* com.sharpcj.aopdemo.test1.IBuy.buy(..)) && within(com.sharpcj.aopdemo.test1.*) && bean(girl)
```

例如上面那个 within

#### 切面的切入时机

 ![img](https://img2018.cnblogs.com/blog/758949/201905/758949-20190529225613898-1522094074.png) 

#### 执行顺序

![1607475811089](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\1607475811089.png)

​	

### 2、定义一个切入点

```java
 @Pointcut("execution(* com.sharpcj.aopdemo.test1.IBuy.buy(..))")
    public void point(){}

 @After("point()")
    public void haha() {
        System.out.println("After ...");
    }
```

###  3、通过注解处理通知中的参数

```java
@Pointcut("execution(String com.sharpcj.aopdemo.test1.IBuy.buy(double)) && args(price) && bean(girl)")
    public void gif(double price) {
    }
```

在execution里面要写返回的参数和方法的参数类型了

```java
  @Around("gif(price)")
    public String hehe(ProceedingJoinPoint pj, double price){
        try {
            pj.proceed();
            if (price > 68) {
                System.out.println("女孩买衣服超过了68元，赠送一双袜子");
                return "衣服和袜子";
            }
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return "衣服";
    }
```

### 底层源码分析



# springboot

## 视图控制器

如果我们的请求都是比较简单的get方法的话，我们可以配置一个视图控制器来控制。

当然这需要我们js那些不是相对路径的。

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/add").setViewName("user/add");
        registry.addViewController("/delete").setViewName("user/delete");
    }
}
```

这样就完成了一个视图控制了。可以对get请求进行转发。

## 配置文件的位置

* file:./config/
* file:./
* classpath:/config/
* classpath:/

`@configurationProperties`来进行从配置文件中读取数据

## 整合Druid

从maven仓库获取 

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.5</version>
</dependency>

```

spring 的配置 yaml

```yaml
  datasource:
    username: 
    password: 
    url: jdbc:mysql//localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
```


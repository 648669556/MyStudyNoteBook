# Mybatis 学习

## 1、1 持久化

1. 数据持久层

   数据持久层是什么？

   - 持久化就是将程序的数据在持久状态（数据库中）和瞬时状态（内存中）直接进行转换的过程。
   - 储存在数据库中，或者放在文件中。
   - 内存，**断电就不见了**

2. 为什么要持久化？

   - 有一些对象不能让他们丢掉了。
   - 内存太贵重了。

## 1、2需要导的包

1. mysql驱动
2. mybatis
3. junit

## 1、3 Mybatis 中文文档

- ​	https://mybatis.org/mybatis-3/zh/index.html

## 1、4 创建一个Mybatis 的步骤

### 需要用到的一些类

   #### 1、SqlSessionFactoryBuilder

   ​ 这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。          

   - 作用域：局部方法变量

   #### 2、SqlSessionFactory

   - 作用域：应用作用域

   ​          应该使用静态单例模式或者单例模式来保存他。**没必要多次创建。**        

   #### 3、SqlSession

   - 作用域：请求或方法作用域

   - 不是线程安全的，每一个线程都有自己的SqlSession实例。

   - 不能将SqlSession 实例的引用放在任何类型的托管作用域中（HttpSession）

   - ```java
     try (SqlSession session = sqlSessionFactory.openSession()) {
      // 你的应用逻辑代码
     }finally{
     // 关闭 SqlSession；
     }
     ```

   在所有代码中都遵循这种使用模式，可以保证所有数据库资源都能被正确地关闭。

------



### 1. 创建一个xml 用来配置Mybatis 。

- ```xml
  <configuration>
    <environments default="development">
      <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
          <property name="driver" value="${driver}"/>
          <property name="url" value="${url}"/>
          <property name="username" value="${username}"/>
          <property name="password" value="${password}"/>
        </dataSource>
      </environment>
    </environments>
  <!--每一个Mapper.xml 都要在 Mybatis 的核心配置中配置注册 --> 
    <mappers>
      <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
  </configuration>
  ```

  可以放在 resource 文件夹下面 一般默认名字为 “**mybatis-config.xml**”

------



### 2 . 获取到 SqlSessionFactory

##### 创建一个 MybatisUtils 类 来执行获取工厂的操作
每个基于 ``MyBatis`` 的应用都是以一个 ``SqlSessionFactory``  的实例为核心的。``SqlSessionFactory`` 的实例可以通过 ``SqlSessionFactoryBuilder``        获得。而 ``SqlSessionFactoryBuilder`` 则可以从 XML 配置文件或一个预先配置的        Configuration 实例来构建出 SqlSessionFactory 实例。 

1. ```java
   String resource = "mybatis-config.xml";
   InputStream inputStream = Resources.getResourceAsStream(resource);
   SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
   ```

   通过这种方式可以获取到 **SqlSessionFactory**对象

   这一步放在static中 

   ```java
   static{
       //在这里获取SqlSessionFactory 对象 
   }
   ```

------

### 3 . 其他的一些操作

##### 创建实体类

- 用于和数据库中的表交互储存

##### 编写 dao层的 接口 

- 编写接口 例如    List<User> getUser();
- 不用再去像以前一样再编写一个 UserImp实现类了

### 4. 编写dao层的 实例化 （映射器）

mybatis中通过一个 ``mapper.xml ``文件来实例化一个 dao层的接口 

```xml
<!-- namespace :命名空间 一般是dao层的绝对路径 id=对应的方法名字 例如（getUser） -->
<!--resultType 内填写的是实体类的绝对路径-->
<!--parameterType传入的参数类型-->
<mapper namespace="org.mybatis.example.BlogMapper">  
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

​        **命名解析：**为了减少输入量，MyBatis对所有具有名称的配置元素（包括语句，结果映射，缓存等）使用了如下的命名解析规则。      

- 全限定名（比如“com.mypackage.MyMapper.selectAllThings）将被直接用于查找及使用。        
- 短名称（比如 “selectAllThings”）如果全局唯一也可以作为一个单独的引用。 如果不唯一，有两个或两个以上的相同名称（比如 “com.foo.selectAllThings” 和        “com.bar.selectAllThings”），那么使用时就会产生“短名称不唯一”的错误，这种情况下就必须使用全限定名。        

##### 如果是maven项目 那么需要配置： 

- ```xml
  <build>
          <resources>
              <resource>
                  <directory>src/main/java</directory>
                  <includes>
                      <include>**/*.properties</include>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>false</filtering>
              </resource>
              <resource>
                  <directory>src/main/rescources</directory>
                  <includes>
                    <include>**/*.properties</include>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>false</filtering>
              </resource>
          </resources>
      </build>
  ```
  
  不然可能出现找不到文件的异常

------

### 5.获得User类的例子

1. 从工厂拿到Session ；

   - ```java
     SqlSession session = sqlSessionFactory.openSession();
     ```

2. 读dao层 中的方法；

   - ```java
     UserMapper usermapper = session.getMapper(User.class);
     ```

3. 使用dao层对象的某个方法；

   - ```java
     User user = usermapper.selectUser();
     ```

4. 关闭Session();

   - ```java
     session.close();
     ```

-------

## 1.5 Mybatis 配置属性

#### 1. properties

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

- 可以从外部的配置表中读取属性
- 同样可以写在 下面以键值对的形式
- 外部配置优先级>在xml配置

#### 2.settings

```xml
<!--这是一个完整的setting设置-->
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

> 这里列出了 可能 用到的比较多的 设置

| 设置名                   | 描述                                                         | 默认值                                                       | 有效值 |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ |
| lazyLoadingEnabled       | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。特定关联关系中可通过设置 `fetchType`属性来覆盖该项的开关状态。 | true \| false                                                | false  |
| cacheEnabled             | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true   |
| logImpl                  | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| **LOG4J** \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |
| mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False  |

#### 3.环境配置

 **不过要记住：尽管可以配置多个环境，但每个 ``SqlSessionFactory``  实例只能选择一种环境。** 

-  **每个数据库对应一个 SqlSessionFactory 实例** 

-  为了指定创建哪种环境，只要将它作为可选的参数传递给 ``SqlSessionFactoryBuilder`` 即可。可以接受环境配置的两个方法签名是：        

```java

SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```

> xml中的配置：

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

- default 表示的是默认的环境工厂环境是那个。
- id 为每一个环境的 id
- transactionMannager是事务管理器
- dataSource表示的是数据源的配置

```xml
<transactionManager type="JDBC">
  <property name="..." value="..."/>
</transactionManager>
```
事务管理器中的设置：

 ``提示``如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

- type：可以选择【JDBC】|【Managed】
  -  JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。 
  - MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为
- 

数据源的设置：

```xml
 <dataSource type="POOLED">
      ...
</dataSource>
```

- type 可以选择 [POOLED] |[UNPOOLED]|[JNDI]
  - UNPOOLED 意思很明显就是不使用 连接池的方式
    - driver ：驱动名
    - url：数据库的登录地址
    - username：用户名
    - password：密码
    -  `defaultTransactionIsolationLevel`  ：默认的连接事务隔离等级
  - UNPOOLED 详情见官方文档

#### 4.取别名（typeAliases）

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

​			这种方式是给 一个实体类取一个别名

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

- 这是 给一个包下面 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。

- 比如 `domain.blog.Author` 的别名为author`；

- 若有注解，则别名为其注解值。 

- ```java
  @Alias("author")
  public class Author {
      ...
  }//通过注解的方式来配置别名 ，虽然方便但是难以维护
  ```

## 在程序中使用映射文件

### select



## 缓存

> 开启全局缓存的步骤

1、在设置里面开启缓存

​	cacheEnabled 在配置文件里面写。

```xml
<setting name="cacheEnabled" value="true"/>
```



2、在sql映射配置表内写 一个 <cache/>

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

​          这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。        

​          可用的清除策略有：        

-  `LRU` – 最近最少使用：移除最长时间不被使用的对象。          
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。          
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。          
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。          

默认的清除策略是 LRU。

​          flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。          默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。        

​          size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。        

​          readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。          因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。          速度上会慢一些，但是更安全，因此默认值是 false。        

​          提示 二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行  flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。        
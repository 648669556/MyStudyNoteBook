# JAVA WEB

## 1.JSP

### 1.JSP是什么：HTML+Java代码

### 2.三大元素

- 1.脚本元素
- 1.JSP注释
	- <%-- 注释体 --%>
	
	- 2.声明变量
- <%! int i=1;%>
		
- 3.JSP表达式
	- <%=表达式%> 输出
	
	- 4.JSP脚本
- <% 脚本体%>
	
- 2.动作元素

	- 1.<jsp:usebean>
- 1.javaBean具备什么样的规则
		
	- 1.要有getter setter发方法
			- 2.必须要有无参的构造函数
			- 3.这个类的访问修饰符必须是public
			- 4.必须要有包结构，不能放在缺省的默认包下面
		
- 2.<jsp:include>
	- 3.<jsp:forward>
	
- 3.指令元素
- 1.<@include>与 <jsp:include>动作元素有什么区别: 先包含在编译，先编译再包含 具体参考文档

### 3.四大作用域

- 1.page：当前页面
- 2.request：一次请求转发
- 3.session：会话
- 4.application：服务器

### 4.九大内置对象

- 1.page
- 2.pageContext
- 3.request
- 4.response
- 5.session
- 6.application
- 7.out
- 8.config
- 9.exception

## 2.Servlet

### 1.生命周期： 创建-初始化（init()）-服务（Service()）-销毁(destroy())

### 2.创建Servlet

- 1.创建一个类继承HttpServlet
- 2.使用向导的方式创建

### 3.初始化

- 1.初始化参数

	- 1.在web.xml当中 配置<init-parma>
	- 2.在servlet当中init方法 调用servletConfig参数的getInitParameter
	- 3.给自定义属性赋值

- 2.次数：只会执行一次
- 3.优先级别：最先执行
- 4.自启：<load-on-starup> 值 1

### 4.服务

- 1.doGet
- 2.doPost
- 3.Get与Post的区别：① 参数：Get请求参数是作为地址栏的一部分进行传输的，Post请求参数是在我们的请求体当中 ② Get 是有长度限制 Post 没有长度显示 ③ 传输文件、视频、图片等等只能使用Post ④ 请求速度：Get较快 Post较慢 ⑤ 直接在地址栏的请求属于Get请求

### 5.执行次数

- 1.初始化方法：执行一次
- 2.服务方法：没调用一次执行一次
- 3.销毁方法：执行一次

### 6.配置

- 1.XML配置方式：WEB-INF底下的web.xml文件当中

	- 1.servlet 和 servlet-mapping 标签

		- 1.servlet： servlet-name、servlet-class、init-parma、load-on-startup
		- 2.servlet-mapping：servlet-name、url-pattern

- 2.注解配置方法：@WebServlet 注解

	- 1.XML已有的子标签，在注解当中都有其属性对应

### 7.servlet调试步骤：见文档

## 3.Filter

### 1.过滤器：过滤我们servlet请求、JSP页面

### 2servlet与filter区别：

- 1.共同点：

	- 1.都要注册
	- 2.都有初始化和销毁方法
	- 3.都有两种配置方式

- 2.不同的：

	- 1.servlet 继承HttpServlet抽象类 filter 实现Filter接口
	- 2.初始化方法当中参数不同：servletConfig 和 filterConfig对象
	- 3.执行方法不同：① servlet 执行 service方法 filter 执行dofilter方法
② 方法参数： servlet： HttpServletRequest和HttpServletResponse类的对象 filter：ServletRequest、ServletResponse、FilterChain参数
	- 4.执行顺序：启动顺序：Listener->Filter->Servlet 销毁顺序：Servlet->Filter->Listener
	- 5.filter-mapping标签当中有个子标签是servlet没有的，dispatch标签

### 3.dispatch标签

- 1.REQUEST：请求
- 2.INCLUDE：包含
- 3.FORWARD：转发
- 4.ERROR：错误

### 4.Filter放行

- 1.chain.doFilter()
- 2.doFilter() 与chain.doFilter()有什么区别 见文档

### 5.使用场景

- 1.字符集统一，解决字符集乱码
- 2.使页面不缓存，解决页面缓存问题
- 3.检测用户是否登录

## 4.Listener

### 1.配置方式

- 1.XML
- 2.注解

### 2.使用场景

- 1.统计网站在线人数

### 3.会话失效：web.xml 当中session-config对象当中配置标签session-timeout标签

## 5.EL表达式

### 1.EL 表达式语言

### 2.作用域对象

- 1.pageScope
- 2.requestScope
- 3.sessionScope
- 4.applicationScope
- 5.原则：就近原则，如果参数不写作用域对象的时候，它会从离它最近的作用域去获取，直至没有

### 3.判断、运算、逻辑、判空、遍历：见文档

## 6.JSTL

### 1.JSLT：java jsp 标签库

### 2.分类

- 1.核心标签库
- 2.SQL标签库
- 3.XML标签库
- 4.EL标签库
- 5.I18N标签库

### 3.导入JSTL的jar包

### 4.使用tablib指令在jsp页面当中调用，不能再HTML当中使用

- 1.属性值

	- 1.uri：标签库的网址
	- 2.prefix：缩写

### 5.核心标签库标签分类

- 1.通用标签
- 2.迭代标签

	- 1.foreach标签
	- 2.forTokens标签

- 3.条件标签
- 4.url标签

## 7.JQuery

### 1.如何使用jQuery：导入JS

### 2.选择器

- 1.标签选择器：$(‘标签名’)
- 2.ID选择器：$(‘#id’)
- 3.伪类选择器：$(‘.class’)
- 4.层次选择器：父子、邻居、隔壁、祖孙后代
- 5.组合选择器：$(‘#id，.class’)
- 6.通配符选择器：$(‘*’)
- 7.……

### 3.样式

- $('p').css()

	- 1.一个参数表示获取样式值
	- 2.表示设置这个样式

### 4.属性

- $('p').attr()

	- 1.一个参数表示获取属性值
	- 2.表示设置这个属性

### 5.事件

- $('p').bind()

	- 1.第一个参数表示事件的名字，第二参数事件实现

### 6.动画

- $('p').animate()

### 7.$等于jQuery

### 8.jQuery与DOM对象转换

- 1.jQuery ->DOM ：var jquerydemo=$('p');
                               var domdemo=jquerydemo[0];
- 2.DOM->jQuery：var domdemo=document.getElementById('demo');
var jquerydemo=$(domdemo)

## 8.Json

### 1.类型

- 1.简单类型

	- 1.数组类型

		- [‘值1’，'值2']

	- 2.对象类型

		- {‘属性名’:‘属性值’，‘属性名2’:‘属性值2’}

- 2.复杂类型

	- 1.数组类型与对象类型综合起来

### 2.概念：传输数据文本格式

### 3.fastJson 阿里巴巴的jar包

*XMind - Trial Version*
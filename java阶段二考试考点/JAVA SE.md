# JAVA SE

## 13.面向对象

### 1.继承

- 1.概念：把共有的属性或者方法提取出来生成一个父类对象，子类去继承父类，去使用这些共有属性或方法
- 2.优点：提高代码的复用性，使类层次结构更清楚
- 3.关键字：extends
- 4.单继承
- 5.不能被继承的父类成员

	- 1.private 成员函数或者属性
	- 2.子类与父类不同包，且使用的是默认的访问权限
	- 3.构造方法

### 2.封装

- 1.关键字：private
- 2.步骤：① 私有化成员属性 ② 提供公开的getter/setter方法

### 3.多态

- 1.override（重写）
- 2.overrload（重载）
- 3.区别： ① override： 一般发生在继承关系当中，子类去复写父类已有的方法（不论是不是抽象方法），访问修饰符 返回值类型 方法名 参数均相同，只有方法体不同
② overload：一般发生在一个类当中（继承关系也是可以），方法名相同，参数列表不同，方法体不同

## 14.数组

### 1.声明方式： 语句：①访问修饰符 数组类型 [ ] 数组名；②访问修饰符  数据类型 数组名 [ ];

### 2.数组长度是在声明时候就固定下来

### 3.数组初始化：①new 数据类型 [长度] ② {值1，值2，值3…… }

### 4.数组的赋值：数组名 [ 下标] =值;

### 5.数组长度：数组对象.length属性

### 6.数组索引最大值：数组对象. length-1，索引从0开始

### 7.数组的排序：使用Arrays工具类

- 1.排序：Arrays.sort(数组名); 

	- 1.使用Arrays排序要遵循的规则：① 整数类型的数组可以直接排序 ② 引用类型的数组排序要元素类实现Compareable接口

- 2.打印：Arrays.toString(数组名);
- 3.数组的拷贝：① 深拷贝 System.arraycopy(); ② 浅拷贝 Arrays.copyof();

### 8.二维数组（了解）：int [] [] 数组名;

## 15.框架集合

### 1.Collection

- 1.List：有序序列集合，按照我们放入元素的顺序排列，长度可变，可以通过索引下标获取

	- 1.ArrayList
	- 2.LinkedList

- 2.Set：无序集合，不能有重复元素，长度可变，只能迭代获取元素

	- 1.HashSet
	- 2.TreeSet

- 3.添加元素  boolean add(Object o)
- 4.删除元素  boolean remove(Object o)
- 5. 包含元素 boolean contains(Object o) 
- 6.获取迭代 Iterator iterator()
- 7.返回容器大小 int size()
- 8.清空元素 void clear()
- 9.判空 boolean isEmpty()
- 10.转换成数组 Object[] to Array()

### 2.Map：键-值对形式的集合，键不能重复，键可以为null值，有且只有一个null值，所有map的方法都是围绕键进行的的

### 3.Comparator 比较器

- 1.概念：强行对某个对象列表(数组)进行整体排序的比较器
- 2.使用场景：① 可以将 Comparator 传递给 Collections.sort 或 Arrays.sort，从而允许在排序顺序上实现精确控制。② 可以使用 Comparator 来控制某些数据结构(如有序 set或有序映射)的顺序，或者为那些没有自然顺序的对象列表(数组)提供排序

## 16.枚举（了解）

### 1.关键字：enum

- 1.概念：枚举可以理解为列举所有，穷举的意思。
- 2.枚举与其他类的区别：① 使用的关键字不同，枚举使用关键字是enum，普通类使用的关键字是class； ② 普通类可以使用extends去继承其他类，而enum 不能使用 extends 关键字继承其他类，因为 enum 已经继承了 java.lang.Enum（java是单一继承）。
- 3.底层实现：要使用 enum 关键字，隐含了所创建的类型都是 java.lang.Enum 类的子类（java.lang.Enum 是一个抽象类）。枚举类型符合通用模式 Class Enum<E extends Enum<E>>，而 E 表示枚举类型的名称。枚举类型的每一个值都将映射到 protected Enum(String name, int ordinal) 构造函数中，在这里，每个值的名称都被转换成一个字符串，并且序数设置表示了此设置被创建的顺序。

## 17.常用工具类

### 1.算术类：Math

### 2.随机数：Random

### 3.日期日历类：Date、Calendar

### 4.对象类：Object

### 5.API手册的使用

## 18.字符串处理类

### 1.StringBuffer

### 2.StringBuilder

### 3.区别：StringBuffer字符串处理类是线程安全性的，处理效率较慢，StringBuilder字符串处理类是线程不安全的，处理效率较快

## 19.异常与异常捕捉

### 1.异常throwable

- 1.错误 Error：机器错误
- 2.异常 Exception：都是人为

### 1.异常捕捉

- 1.语法结构：try{  }catch{ } finally{ } 
- 2.try{ }捕捉代码块，把我们有可能出现问题或者异常包裹起来，如果出现异常进行抓取
- 3.catch{ } 捕获代码块，将捕捉到的异常进行处理
- 4.finally { } 必定会执行的代码块，不论发生任何的异常，都会执行的代码块，哪怕是return语句也不能逃脱

## 20.泛型

### 1.E:元素

### 2.T：类型

### 3.K：键

### 4.V：值

### 5.？：占位符

### 6.限定参数的类型

### 7.概念：创建集合容器时规定其允许保存的元素类型，然后由编译器负责添加元素的类型合法性检查，在取用集合元素时则不必再进行转型处理。

### 8.指定上下界：
① 指定上界 	<? extends superclass>
superclass表示用作上界的类的名称。构成上界的类（由superclass指定）也包含在限制之内
② 指定下界	<? super subclass>
有subclass和subclass的超类才是可接受的变元类型。

## 21.多线程

### 1.生命周期（重点）：① 创建状态 ② 就绪状态 ③ 运行状态 ④ 阻塞状态 ⑤ 死亡状态（销毁状态）

### 2.创建多线程的方式：① 继承Thread类 ② 实现Runable接口（推荐）

### 3.死锁的四个条件（重要）：① 互斥条件 ② 占有且申请 ③ 不可抢占资源 ④ 循环等待 这四个条件必须全部存在，才形成死锁

### 4.yield方法：使我们的线程暂时小憩，释放它所占有资源，进入一个休憩状态，作用：打破死锁

### 5.线程与进程：
进程：每个独立运行的程序称为一个进程
线程：程序内部多个任务(顺序控制流)并发执行，每个任务称为一个线程。所以，线程是一个程序内部的顺序控制流

### 6.守护线程

- 1.线程的分类

	- 1.主线程：JVM自动程序入口方法main()所产生的线程
	- 2.子线程：主线程中创建的线程
	- 3.用户线程：主线程中创建用于完成用户指定任务的线程
	- 4.守护线程：在后台运行为其他线程提供服务的线程，也称为后台线程

- 2.设置守护线程 Thread.setDaemon(Boolean on) 表示是否设置为守护线程，这个方法必须在启动线程前调用

### 7.线程的优先级别：线程优先级指的是线程被线程调度器调度和执行的优先级别。调度器倾向于让优先级高的线程先执行，但并不是优先级低的线程得不到执行，仅仅是执行的频率较低。主线程的优先级别默认为5，子线程的优先级别与其父线程相同

### 8.互斥锁：synchronize关键字

- 1.方法（函数）
- 2.语句块（属性或者对象）

## 22.内部类（了解）

### 1.语法结构 class A{  class B{ } } 

### 2.必须要定义外部类对象才可以去调用到内部类的对象

## 23.IO流

### 1.分类

- 1.类型： ① 字符流 ② 字节流
- 2.流向： ① 输入流 ② 输出流
- 3.流的概念：一连串流动的字符,是以先进先出方式发送信息的通道。

### 2.字节流

- 1.InputStream 输入流
- 2.OutputStream 输出流

### 3.字符流

- 1.Reader
- 2.Write

### 4.文件对象 File：在java中用来封装和访问文件和目录名对象，是文件和目录路径名的抽象表示形式。

### 5.序列化和反序列化

- 1.ObjectOutputStream
- 2.ObjectInputStream
- 3.序列化的类必须要实现Serializable接口

## 24.接口与抽象类

### 1.接口

- 1.关键字：interface 实现 implements
- 2.多实现
- 3.在JDK1.7之前，所有的方法都是抽象方法
- 4.属性：常量
- 5.无默认无参构造方法

### 2.抽象类

- 1.关键字：abstract class 继承 extends
- 2.单继承
- 3.可以是抽象方法，也可以不是抽象方法
- 4.属性：跟类当中定义的属性一样
- 5.无默认无参构造方法

## 25.正则表达式

### 1.*：匹配0次或多次

### 2.？：匹配0次或1次

### 3.$：匹配结尾

### 4.+：匹配1次或多次

### 5.^：匹配开始

### 6.常见通用的正则表达式，比如邮箱，手机号码…… 见附件

## 26.JDBC

### 1.步骤：① 导入连接jar包 ② 注册驱动 ③ 创建连接 ④ 创建执行器 ⑤ 编写sql语句  ⑥使用执行执行SQL语句 ⑦ （可选）需要一个结果集对象去接收执行结果 ⑧ 关闭结果集、执行器、连接

- 1.注册驱动：Class.forName(com.mysql.jdbc.Driver) 数据库8之前

### 2.先开后关的原则

- 1.数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统宕机。Connection的使用原则是尽量晚创建，尽量早的释放。

### 3.事务

- ① connection.setAutoCommit(false);//打开事务。
② connection.commit();//提交事务。
③ connection.rollback();//回滚事务。
④ SavePoint sp = connection.setSavepoint();	//设置保存点

## 12.注释与注解

### 1.注释

- 1.文档注释 ：/** 注释体 */
- 2.多行注释： /* 注释体 */
- 3.单行注释： //

### 2.注解与注释的区别

- 1.注解 是以@符开头
- 2.注解作用的位置：① 类 ② 属性 ③ 函数（方法）
- 3.自定义注解（了解）
- 4.注解是有作用的，注释是没有任何作用的 只起到说明的作用

## 11.关键字

### 1.static

- 1.概念：静态的
- 2.静态的属性：值会保留，直接可以用类名调用，不需要实例化
- 3.静态的方法：也不需要实例化，可以用类名.方式去调用
- 4.静态代码块：最优先执行 执行顺序：静态代码块>代码块>构造函数

### 2.final

- 1.类

	- 1.不能被继承

- 2.属性

	- 1.不能被修改
	- 2.必须在声明的时候赋值

- 3.方法（函数）

	- 1.不能被重写

### 3.this

- 1.概念：指代的当前类的实例

### 4.super

- 1.当前类的父类的实例
- 2.在构造函数当中，super构造器必须要在构造方法当中的第一行

### 5.break

- 1.循环和分支：跳出所在的当前的循环或者分支结构当中
- 2.switch-case：每一个case选项都需要填写break，可以不填写但是会继续执行下面case体直到break的出现

### 6.continue

- 1.循环当中，使用continue，放弃执行当次循环此条continue语句之后的代码，开始下一次的循环，跟break一样，只对它当前所在的循环生效，外层循环不生效

### 7.return

- 1.用于返回函数返回值，相当于结束方法
- 2.如果有finally代码块，return 也不可以逃脱，依旧会执行finally代码块

### 8…………

### 9.true 和 false 这两个不是关键字，虽然在eclipse当中显示是紫红色且加粗，但是它们不是关键字，boolean类型值

## 10.程序执行结构

### 1.顺序结构

- 1.瀑布式的流程方式

### 2.分支结构

- 1.if ：单路分支
- 2.if-else ：双路分支
- 3. if-else嵌套 ： 多路分支
- 4.switch-case ：针对某个变量的值进行判断
- 5.三目运算符：用于赋值

### 3.循环结构

- 1.for：计次循环
- 2.foreach循环：遍历
- 3.while循环：条件循环
- 4.do-while：至少执行一次的条件循环

## 9.包与包结构

### 包：把我们定义生成的类归类化

### 包的命名规则及表示法：① 所有字母都要小写 ② 表示下一层级的包中间使用英文的.

### 类当中包的要求：① package 语句 是表示我们这个类所在包的位置，且必须要在源文件的第一行，如果没有任何的包结构，可以没有package语句，package语句最多只能有一条  ② import语句在我们的package语句之后，可以没有import语句，import语句可以有多条

## 8.访问修饰符

### 1.private：本类下面可以调用，其他均不可调用

### 2.default：① 不需要去写default关键字 默认省略 ②作用域：本包下面可以来回调用

### 3.protected：作用域：本包下及它的子类均可以调用

### 4.public：作用域同一个项目内，均可以调用

### 5.作用域大小：public>protected>default>private

## 7.运算符

### 1.算数运算符：+ - * / %

### 2.逻辑运算符： 与 && 或 || 非 !

### 3.关系运算符：> < >= <= == !=

### 4.位运算符（了解）：~  >> << 

### 5.三目运算符

- 1.语法结构：a?x:y; a关系表达式 a真 返回x 否则返回y
- 2.使用场景：用于赋值

### 6.赋值运算符（优先级低）： =

### 7.一元运算符：正号+ 负号 -

### 8.小括号：优先级别 高

### 9.优先级别：小括号为最高，最低为赋值负号 ++ 或 -- 如果放在变量的后面，他的优先级别比赋值运算符还要低

## 6.构造器

### 1语法：访问修饰符 类名 ( 参数 ){  } 一般来说访问修饰符：publc类型

### 2.作用：生成类的对象实例

### 3.父子类的构造方法：先执行我们父类的构造方法再执行子类的构造方法

## 5.语法结构

### 1.类

- 1.访问修饰符 class 类名{  }

### 2.属性

- 1.访问修饰符 数据类型 变量名;

### 3.方法

- 1.访问修饰符 返回值类型 方法名( 参数) {  }

### 4.(静态)代码块

- （static）{ }

## 4.数据类型

### 1.基本数据类型

- 1.数值型

	- 1.整数型：默认值 0

		- 1.byte：8位
		- 2.short：16位
		- 3.int：32位
		- 4.long：64位

	- 2.浮点型：默认值 0.0

		- 1.单精度浮点型：float
		- 2.双精度浮点型：double

- 2.布尔型

	- boolean：默认值 false

		- 1.true
		- 2.false

- 3.字符型

	- char

		- 1.默认值：空
		- 2.字符型是不是可以和我整数型进行转换

### 2.引用数据类型

- 1.数组
- 2.基本数据类型的包装类
- 3.类对象
- 4.接口对象
- 5.枚举

### 3.基本数据类型与引用数据类型的区别

- 1.基本数据类型只可以.出来.class 包装类可以.出很多的方法和属性
- 2.默认值：定义数组时显示默认值不一样
- 3.基本数据类型关键字，在编译器上面显示紫红色粗体，引用数据类型它本质上是一个类

### 4.类型转换

- 1.自动类型转换（向上转型）：缺陷：无
- 2.强制类型转换（向下转型）：缺陷：1.精度丢失 2.越界

### 5.装拆箱

- 1.自动装拆箱

	- 1.装箱：基本数据类型转换成它的包装类
	- 2.拆箱：包装类转换成对应的基本数据类型

### 6.比较

- 1.双等于 ==

	- 1：基本数据类型 比较值是否相等，返回值 布尔类型 引用数据类型：比较内存当中的地址

- 2.equals

	- 1.基本数据类型与引用数据类型：表示都是比较值和类型 返回值 布尔类型

- 3.compareTo

	- 1.一般用于比较基本数据类型或者某个引用数据（例如：对象）当中一个属性值进行比较判断 返回值结果 int 类型 前者比后者大 大于0 ，后者比前者大 小于0 两者相等 等于0

### 7.全局变量与局部变量

- 1.全局变量：在类当中定义的变量，叫做全局变量（成员变量）
- 2.局部变量：① 函数的参数 ：函数内部 ② 函数方法体当中定义的变量：在函数内，自它本身定义以下代码可以使用 ③ 条件分支、循环分支

## 3.分隔符

### 1.空格：区分单词与单词之间的分割

### 2.逗号：分割参数，声明多个变量

### 3.分号：语句的结束，for循环当中3个参数的分割

### 4.冒号：switch-case，三目运算符，foreach循环

### 5.点：层级关系

### 6.大括号：函数的作用域范围，数组的赋值，代码块

### 7.中括号：数组

### 8.小括号：函数的参数列表分割，for，while，if，do-while……

### 9.引号：赋值字符串，字符型赋值单引号

### 10.->：lambda表达式(JDK1.8之后才出现的)

### 11.尖括号：泛型

## 2.标识符

### 概念：通俗来说，标识符指代变量名，函数名，类名，包名

### 标识符的命名规则

- 1.不能以数字开头
- 2.标识符只可以使用_、$、数字、英文构成
- 3.不能使用我们JAVA当中的关键字
- 4.标识符严格区分大小写
- 5.见名知意(建议)

### 标识符常用的命名规范

- 变量名：驼峰式的命名规范（学生姓名：stuNameDemo）
- 函数：与变量名一致
- 类：每个单词首字母都大写
- 包：所有都小写
- 常量：所有的字母都是大写

## 1.环境变量的配置

### 1.JAVA_HOME

- JDK安装的路径

### 2.CLASSPATH

- .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;  

### 3.Path

- %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

*XMind - Trial Version*
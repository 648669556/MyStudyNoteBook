# Web前端知识

## HTML

### HTML：（Hyper Text Markup Language）超文本标记语言

### 闭合方式：

- 1.成对出现（标签闭合） 例如<html></html>
- 2.自闭合  例如 <meta />、<br/>、<hr/>

### 标签的分类

- 1.块级标签：独占一行

	- 1.div  布局标签
	- 2.h1-h6  一级-六级标题
	- 3.p 段落
	- 4.table 表格
	- 5.form 表单
	- 6.ol\ul 有序列表\无序列表

- 2.行级标签（内联标签）：只占一行的一部分

	- 1.span 布局标签
	- 2.image 图片
	- 3.input 表单控件
	- 4.label 标签
	- 5.a 超链接

### 文档流：从上之下，从左至右

### table表格标签

- tr 行标签

	- td 单元格标签
	- th 表头标签 
	- 区别：th标签包裹的文字加粗 且 居中

- tbody、thead、tfoot 表格布局标签
- caption  表格标题
- 合并单元格：都只能作用于td或者th当中

	- 跨行 rowspan
	- 跨列 colspan

### form 表单标签

- 属性

	- method：提交方式  get/post
	- action：提交url

- input标签 根据type属性值变化而变化

	- type=text 文本框
	- type=password 密码框
	- type=button 普通按钮
	- type=radio 单选

		- 1.提交值 value
		- 2.单选按钮一定是2个或者2个以上选项，规定多个单选按钮为一组，使用我们name属性，把name属性设置一致即可实现单选

	- type=checkbox 多选

		- 跟我们单元框一样，同样需要设置name值表示，两个或者两个以上的所有的多选框作为一组数据

	- type=file 文件选择器

		- 1.一旦使用了file文件选择器控件，form表单 method属性只能使用post

	- type=image 图片
	- type=hidden 隐藏域
	- type=submit 提交按钮
	- type=reset 重置按钮
	- type=date 日期选择器

- select 下拉框

	- option 子项：提交的值是value里面的值，并不是option标签内的文字

- textarea 文本域

	- 作用：让用户输入大量的文字的

### a 超链接标签

- 页面跳转

	- <a href="地址">请点击我</a>

- 锚记

	- 定义锚记<a name="demo"></a>          
<a href="#demo">跳转到锚记</a> //跳转锚记href使用#开头

- 功能性连接

	- <a href="mailto:xxxxx@163.com">请联系我</a>

### HTML注释

- 1.HTML注释是有作用的，作用是为了解决用户浏览器版本兼容问题
- 2.能否可以看见

	- 1.用户页面是无法看见的
	- 2.F12 控制台源码当中是可以看见的

### 特殊符号

-  以&开头，以分号结尾

	- 1.空格 &nbsp;
	- 2.大于 &gt;
	- 3.小于 &lt;
	- 4.大于等于 &ge;
	- 5.小于等于 &le;
	- 6.版权符号 © &copy;
	- 7.引号 &quot;

## CSS

### CSS:层叠样式表 Casading Style Sheet

### CSS 类型

- 1.头部样式表

	- 写在我们style标签里面<style></style>

- 2.外部样式表

	- 使用link标签 链接外部样式 <link rel="styleSheet" href="路径">
	- @import 导入样式

- 3.内联样式（行间样式） 

	- style属性

### CSS的选择器：

- ID选择器

	- 以#开头 例如：#id名{ 样式1：样式值1}

- 伪类选择器

	- 以英文的.开头 例如.class名{  样式1:样式值1}

- 标签选择器

	- 以标签开头 例如 <h1>{ 样式1：样式值1}

### CSS常用的属性

- background 背景颜色
- color 字体颜色
- font-family 更换字体
- font-size 字体大小

### CSS作用原则

- 就近原则

### CSS特性

- 归类性

	- 以，区分 例如 h1,#demo{ color : red}

- 继承性
- 情景选择

	- 以 空格区分

### 盒子模型（重要）

- margin 外边距

	- margin-top
	- margin-left
	- margin-right
	- margin-bottom

- border 边框

	- 同上

- padding 内边距

	- 同上

- 参数个数

	- 一个参数 例如 margin:10px

		- 该样式的上下左右外边距都为10像素

	- 二个参数（使用空格分隔） 例如 margin：10px 15px;

		- 上下为10像素 左右为15像素

	- 四个参数 例如 margin：20px 15px 10px 5px

		- 上右下左的顺序 规律 以上为起点，顺时针转一圈

## JavaScript

### Javascript与Java有没有关系？ 没有

### Java 与JavaScript 区别

- 1.Java 强变量语言 Javascript弱变量语言
- 2.Java编译执行 Javascript解释执行
- 3.Java面向对象 Javascript 基于对象
- 4.代码格式不一样
- 5.嵌入方式不同

### 用法

- 使用<script type="text/javascript"></script>
- <script src="路径"></script>

### 声明变量

- var 关键字

	- 1.let 定义一个变量一样
	- 2.const 定义一个常量

### 声明方法

- function 关键字
- return false;

	- 抑制默认事件

### 声明数组

- 1.var nums=new Array(); 定义一个空数组
- 2.var nums=new Array(5); 定义长度为5的数组（元素5个都为empty）
- 3.var nums=new Array("张三","李四");
- 4.var nums=["张三","李四"]
- Javascript数组长度不是固定的，而且可以自动扩容

### 11内置对象

- 1.Array 数组
- 2.Number 数字
- 3.Object
- 4.Boolean
- 5.RegExp 正则
- 6.Date 日期
- 7.Math 数学
- 8.Function 函数
- 9.String 字符串
- 10.Error 异常
- 11.Global 全局

	- eval 执行
	- with 提取公因类

### 宿主对象（了解）

- 浏览器对象(Navigator) ----提供有关浏览器的信息
- 屏幕对象(screen) ----反映了当前用户的屏幕设置
- 　 窗口对象(Window) ----Window对象处于对象层次的最顶端，它提供了处理Navigator窗口的方法和属性。
- 　 位置对象(Location) -----Location对象提供了与当前打开的URL一起工作的方法和属性，它是一个静态的对象。
- 　 历史对象(History) -----History对象提供了与历史清单有关的信息。
- 　 文档对象(Document) -----document对象包含了与文档元素(elements)一起工作的对象，它将这些元素封装起来供编程人员使用。

### DOM

- Document Object Module
- 节点树

	- 表示我们HTML页面各个节点层级关系

- 获取节点的方式

	- 根据标签名去获取

		- doucment.getElementsByTagName

	- 根据伪类获取

		- document.getElementsByClassName

	- 根据ID获取

		- document.getElementById

- 增加节点/追加节点

	- document.createElement()
	- 案例一

	- 案例二

- 移除节点

	- 移除子节点

		- 节点对象.removeChild(元素)

	- 移除自身节点

		- 节点对象.remove()

*XMind - Trial Version*
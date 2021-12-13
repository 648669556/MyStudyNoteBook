### 插槽slot

 动态的可拔插的组件。

> 使用实例

```html
<div id = "app">
    <p>
        列表书籍
    </p>
    <ul>
        <li>JAVA</li>
       	<li>Linux</li>
        <li>Python</li>
    </ul>
</div>
```

### 各种关键字

tips

> 在Mustache语法中可以进行判断 例如 三元运算符，和jsp的el表达式挺相似的。

#### model

`v-model`：将 value 值与对应的属性进行绑定

使用案例:

```vue
<select name="" id="" v-model="selected">
    <option value="">请选择</option>
    <option value="浙江大学">浙江大学</option>
    <option value="清华大学">清华大学</option>
    <option value="北京大学">北京大学</option>
</select>
```

#### if

`v-if`:如果条件为真，则会在虚拟dom中存在

使用案例：

```vue
<view >
    <h1 v-text="msg" v-if="selected ==='浙江大学'"></h1>
    <h1 v-else>你不配看! :(</h1>
</view>
```

#### else-if

`v-else-if`：多个条件判断

#### else

`v-else`:如果之前的if不对则会为这个

#### show

`v-show`：条件如果为真则会显示，如果为假则会将内容display

#### for

`v-for`: 列表渲染使用的

还要配合一个 `v-key`一起使用 保证组件和数据捆绑唯一

使用案例

```vue
<view v-for="(item,index) in days" :key="item.id">
    <li v-text="item.name"></li>
</view>
```

> 嵌套多重循环的时候需要指定不用的索引！！！

block标签

block标签可以把换行移除然后就把文字输出


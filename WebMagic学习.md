### 快速开始

#### 使用Maven

导入两个jar包`WebMagic-core`、`webMagic-extension`。

```xml
<!-- https://mvnrepository.com/artifact/us.codecraft/webmagic-core -->
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-core</artifactId>
    <version>0.7.4</version>
</dependency>
<!-- https://mvnrepository.com/artifact/us.codecraft/webmagic-extension -->
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-extension</artifactId>
    <version>0.7.4</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
    <scope>compile</scope>
</dependency>
```

ps:其实不知道为什么必须要使用slf4j的日志才可以启动。希望以后可以增加更换其他日志的方法把。不过影响也不大。

---------------

### 文件架构

![image](http://code4craft.github.io/images/posts/webmagic.png)

#### 4个组件

1、Downloader

`Downloader`负责从互联网上下载页面，以便后续处理。一般对于我使用来说，其实这一个模块我们不用关心太多。我们只需要在使用到ip代理的时候去主动的设置一下代理的Downloader就可以了。这个后面会讲

2、PageProcessor

`PageProcessor`这个是这个爬虫框架的核心了，这个部分主要是解析页面，然后决定需要把什么url链接加入到接下来需要爬取的队列。这里需要我们使用者去定制使用了。

3、Scheduler

`Scheduler`负责管理待抓取的队列，以及一些去重的给你做，WebMagic默认提供了JDK的内存队列来管理URL，并使用了集合去重。所以一些我们已经爬取过的网页是不会再次去访问的。

这个我们使用者一般也不用关心。

4、Pipeline

负责抽取结果的处理，对得到的数据我们要怎么处理呢？存到数据库？写到文件夹，还是仅仅在控制台输出？这里就是也需要我们自己去定制使用的地方。



#### 部分对象解释

1、**Request**

`Request`是对URl地址的一层封装，他是`PageProcessor`与`DownLoader`交互的**载体**，我们可以在这里附上一些信息，通过在`Request`对象里面的一个`extra`字段，这个字段是一个KEY-Value的结构，我们可以在这里添加一些信息来完成一些功能，例如我们可以通过这个url为什么类型，然后在其他地方读取到然后判断。

使用样例：

```java
/*在PageProcess中
例如我们需要对一个url进行标注一下，这个url下面的都是作者的信息我们可以这样
*/
page.getRequest().putExtra("type"."authorinfo");
    
//在另一个page或者当前的
    page.getRequest().getExtra("type");

```

如此我们就可以很方便的判断页面的种类了。

**2、Page**

`Page` 代表了从`Downloader`下载到的一个页面，他可能是HTML或者JSON或者其他格式的东西。

这个Page里面包含了很多的信息，几乎一个页面需要的信息都有了

![image-20201223202218923](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20201223202218923.png)

这是Page的属性，看到属性我们也可以知道我们可以通过page获取很多的信息了。

3、**ResultItems**

`ResultItem`可以把他当成一个Map，他保存了`PageProcessor`处理的信息，给`Pipeline`使用。

一个页面如果有需要处理的数据我们就需要把数据保存下来放到这个`ResultItem`里面，

例如

```java
page.putField("type", "authorinfo");
```

这样就是在`ResultItem`里面存入了一个`type`为键的值为`authorinfo`的键值对

我们只需要在`Pipeline`里面取出来用就可以了。

然后需要注意的是，我们有时候并不需要爬取一个网页的信息，那么我们也不信息他进入我们的`Pipeline`里面，这里我们就可以使用一个方法给他设置跳过。这个属性其实也是`ResultItems`里面的。

```java
page.setSkip(true)
```

在`PageProcess`里面我们使用这个方法就说明我们当前的这个页面不希望进入处理。

### 快速上手

> 代码示范

1、去写一个属于我们自己的`PageProcessor`

```java

public class AuthorInfoProcessor implements PageProcessor {
    //这个属性来说一下，这个是我们的页面重试次数还有设置每两次请求延迟的地方
    private final Site site = Site.me().setRetryTimes(3).setSleepTime(5000);

    @Override
public void process(Page page) {
page.putField("type", "authorinfo");
String s=page.getHtml().xpath("//div[@id=\"profile_block\"]/a[1]/@href").toString();
String substring = s.substring(0, s.length() - 1);
String authorid = substring.substring(substring.lastIndexOf("/") + 1);
page.putField("authorid", authorid);
page.putField("age", page.getHtml()
              .xpath("//div[@id=\"profile_block\"]/a[2]/@title")
              .regex("[0-9]+-[0-9]+-[0-9]+").get()
             );
List<String>info=
    page.getHtml()
    .xpath("//div[@id=\"profile_block\"]/allText()")
    .all();
String infos=info.get(info.size()-1);
String fansinfo = infos.substring(infos.indexOf("粉丝"), infos.indexOf("关注"));
String attentioninfo = infos.substring(infos.indexOf("关注"));
String fans = fansinfo.substring(fansinfo.indexOf(" ")).trim();
String attention = attentioninfo.substring(attentioninfo.indexOf(" ")).trim();
page.putField("fans",fans);
page.putField("attention",attention);
}

    @Override
    public Site getSite() {
        return site;
    }
}

```

2、编写主程序

```java
public static void main(String[] args) {
    Spider.create(new GithubRepoPageProcessor())
            //从https://github.com/code4craft开始抓    
            .addUrl("https://www.cnblogs.com/zkweb/")
            //设置Scheduler，使用Redis来管理URL队列
            .setScheduler(new RedisScheduler("localhost"))
            //设置Pipeline，将结果以json方式保存到文件
            .addPipeline(new JsonFilePipeline("D:\\data\\webmagic"))
            //开启5个线程同时执行
            .thread(5)
            //启动爬虫
            .run();
}
```

怎么样，看起来是不是特别简单，我们只需要添加必要的`url`和`PageProcessor`类就可以了。

解释一下几个配置

1、**create**

这个方法里面需要我们放入我们的`PageProcessor`即可。

2、addUrl

这个看起来就知道是什么把哈哈哈哈

3、addPipeline

这个方法我们来添加自己的数据处理的东西。我们可以添加多个

4、thread

这个就是创建一个多大的线程池来爬取

5、run

执行run方法就是会一直等到爬虫结束了程序才会向下走。相当于直接执行了一个方法一样。`Runnable`接口之间执行run方法类似。

6、start

start方法不会被阻塞，就是新建了要给线程去处理我们的爬虫程序所以不会被阻塞。

### 编写爬虫程序

1、**实现PageProcessor**

我们想要实现PageProcessor是很简单的。

仅仅需要我们在我们的类中实现接口`PageProcessor`就可以了。

一共需要实现两个方法，`process()`和`getSite()`

`process()`方法就是我们处理页面的主体方法。我们在这里处理获得的html页面和Json信息等等。

```java

import us.codecraft.webmagic.Page;
import us.codecraft.webmagic.Site;
import us.codecraft.webmagic.processor.PageProcessor;

public class MyPageProcess implements PageProcessor {
    final Site site =new Site().setRetryTimes(3).setSleepTime(1000);
    @Override
    public void process(Page page) {
        
    }

    @Override
    public Site getSite() {
        return site;
    }
}
```

例如上面这样的就是一个`PageProcessor`页面，我们需要处理获得信息仅仅在process里面处理就可以了。

像下面这样：

```java
 page.getHtml().xpath("//a[@Class=\"text\"]").all();
```

#### 定制我们的Pageprocessor

>  页面判断是否匹配?

```java
page.getUrl().regex("www.cnblogs.com\\/").match()
```

上面这句话就是获取到页面的`url`然后通过正则表达式判断是否是符合正则规则的规则。返回值为`boolean`。

> 获取页面的html内容

```java
 page.getHtml()
```

上面这句话就是获取到了整个页面的html信息。当然如果你的是json页面你可能需要这个：

```java
 page.getRawText()
```

通过这个我们可以获取整个json的字符串.

上面两种是我们用到最多的。

当然我们在日常使用的时候，当然不会想要获取到整个页面的html代码，这对我们是没有必要的。

所以我们有许多种过滤和筛选信息的方式。

```java
//通过xpath的规则来获取所有符合的内容
page.getHtml().xpath("//a[@Class=\"text\"]").all();
//当前页面下面的所有的链接
page.getHtml().links();
//符合正则表达式的内容
page.getHtml().regex("[_]");
//css匹配选择器
page.getHtml().css("div.table");
//这个是类似jquery的写法来匹配内容。
page.getHtml().$("div.table");
```

我们获取信息的时候有许许多多种的方式。当然我使用最多的还是通过xpath的方式，所以来讲讲怎么使用xpath方式。

`"//*[@id="name"]"/text()`

上面这个//就是全局的意思。然后后面这个 *就是任意的意思，他可以是div可以是a可以是span。但是内部的@代表一个属性 我们这里写的是id所以就是id属性，如果我们写的是class的话就是匹配class是这个的。 当我们需要匹配多个的时候，我们可以使用`or` 来帮我们。

我们可以这样写`//*[@class=\"a\" or @class=\"b\"]/text()`这样我们就是获取了任意一个class为a或者b下面的text文本。

![image-20201224134910156](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20201224134910156.png)

这是官方文档中的给出来的我们可以使用的方法。

这是官方文档中的`xpath`写法的解释

| Name                  | Expression           | Support         |
| --------------------- | -------------------- | --------------- |
| nodename              | nodename             | yes             |
| immediate parent      | /                    | yes             |
| parent                | //                   | yes             |
| attribute             | [@key=value]         | yes             |
| nth child             | tag[n]               | yes             |
| attribute             | /@key                | yes             |
| wildcard in tagname   | /*                   | yes             |
| wildcard in attribute | /[@*]                | yes             |
| function              | function()           | part            |
| or                    | a \| b               | yes since 0.2.0 |
| parent in path        | . or ..              | no              |
| predicates            | price>35             | no              |
| predicates logic      | @class=a or @class=b | yes since 0.2.0 |

> 将我们需要继续进去爬的页面放入队列

有些时候我们需要将获取的链接放入队列来支持爬取。我们可以使用

2个方法来实现。

```java
page.addTargetRequest();
page.addTargetRequests();
```

这两个方法都实现了重载。可以传入`String `或者`Request` 显然，带一个s的就是多个的意思。我们可以将List<String> 或者List<Request> 类型的东西放入。

> 将爬取到的数据放入我们的数据集ResultItems中

```
page.putField(String，Object);
```

我们通过这样的一个方法将我们的爬取到的信息存储到ResultItems中以供Pipeline来处理。

#### 定制我们的Pipeline

> Pipeline主要是进行数据持久化操作的地方，是需要把数据放到哪里。

我们需要将我们的类实现`Pipeline` 同时也有个方法叫做`process`

我们需要在process里面写我们需要处理的操作。

```java
@Component
public class CnblogPipeline implements Pipeline {
    final BlogService service;
    final AuthorPipeline authorPipeline;
    final AuthorInfoPipeline authorInfoPipeline;

    @Autowired
    public CnblogPipeline(BlogService service, AuthorPipeline authorPipeline, AuthorInfoPipeline authorInfoPipeline) {
        this.service = service;
        this.authorPipeline = authorPipeline;
        this.authorInfoPipeline = authorInfoPipeline;
    }

    @Override
    public void process(ResultItems resultItems, Task task) {
        int pick = 0;
        boolean isAuthor = false;
        String authorid = null;
        List<String> authorUrl = null;
        if (resultItems.get("type").equals("Picked")) {
            pick = 1;
        } else if (resultItems.get("type").equals("author")) {
            authorPipeline.process(resultItems, task);
            authorid = resultItems.get("authorid");
            isAuthor = true;
        } else if (resultItems.get("type").equals("authorinfo")) {
            authorInfoPipeline.process(resultItems, task);
            return;
        }
        if (!isAuthor) {
            authorUrl = resultItems.get("authorUrl");
        }
        List<String> titles = resultItems.get("titles");
        List<String> url = resultItems.get("url");
        List<String> views = resultItems.get("views");
        List<String> time = resultItems.get("time");
        List<String> id = resultItems.get("id");
        for (int i = 0; i < titles.size(); i++) {
            try {
                if (!isAuthor) {
                    String temp = authorUrl.get(i);
                    String authortemp = temp.substring(0, temp.length() - 1);
                    authorid = authortemp.substring(authortemp.lastIndexOf("/") + 1);
                }
                service.addDateTime(Integer.parseInt(id.get(i)), LocalDate.now(), Integer.parseInt(views.get(i)));
                service.addBlog(Integer.parseInt(id.get(i)), titles.get(i), url.get(i), time.get(i), Integer.parseInt(views.get(i)), authorid, pick);
            } catch (DuplicateKeyException e) {
            }
        }
    }
}
```

这里我使用了要给主要的Pipeline来处理各个`ResultItem` 我这里是为了不重复写代码才这样写的，大家可以使用添加多个`pipeline`的方式来处理，判断是那个页面来处理数据，我们这里就是通过写到数据库当中来存储。当然我这里是使用了`spring`来管理我们的对象。

**重点：**

`resultItems.get()`通过这个方法来拿到我们的数据

`Task`参数是一个接口可以让我们拿到我们页面的`Site`和`uuid`

![image-20201224141244652](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20201224141244652.png)

![image-20201224141218782](C:\Users\陈俊宏\AppData\Roaming\Typora\typora-user-images\image-20201224141218782.png)



这个是我们的方法实现;

到这里需要我们自己必须要定制或处理数据的地方就结束了。

接下来讲讲我们可能会用到的地方。

### 处理非Http GET请求

一般爬虫只会需要使用到get方法就足够了，但是有时候我们需要使用到post请求。

只需要我们把放入请求队列的方法的request添加

```java
Request request = new Request("http://xxx/path");
request.setMethod(HttpConstant.Method.POST);
request.setRequestBody(HttpRequestBody.json("{'id':1}","utf-8"));
```

注意，我们的post请求不会去重的。不像我们的get请求。当你多次请求同一个post地址的时候是不会去除的。

HttpRequestBody内置了几种初始化方式，支持最常见的表单提交、json提交等方式。

| API                                                          | 说明                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| HttpRequestBody.form(Map\ params, String encoding)           | 使用表单提交的方式                   |
| HttpRequestBody.json(String json, String encoding)           | 使用JSON的方式，json是序列化后的结果 |
| HttpRequestBody.xml(String xml, String encoding)             | 设置xml的方式，xml是序列化后的结果   |
| HttpRequestBody.custom(byte[] body, String contentType, String encoding) | 设置自定义的requestBody              |
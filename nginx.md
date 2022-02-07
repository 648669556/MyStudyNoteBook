### 什么是Nginx

Nginx是一个高性能的HTTP和反向代理Web服务器，是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点开发Web服务器。Nginx源代码以类BSD许可证的形式发布，其第一个公开版本0.1.0发布于2004年10月4日，2011年6月1日其1.0.4版本发布。Nginx因高稳定性、丰富的功能集、内存消耗少、并发能力强的而闻名全球，目前得到非常广泛的使用，比如说百度、京东、新浪、网易、腾讯、淘宝等都是其用户。

### 代理方式

### 正向代理



![image-20211216212524638](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211216212524638.png)

> **正向代理最大的特点是客户端非常明确要访问的服务器地址**

就是给客户端配置目标服务器的信息，例如IP和端口号，这样就可以直接访问到目标的机器上。

### 反向代理

![image-20211216213017684](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211216213017684.png)

> **客户端不知道目标服务器的地址**

客户端向反向代理服务器直接发送请求，接着反向代理将请求转发给目标服务器，并将目标服务器的响应结果按原路返回给客户端。

正向代理和反向代理的使用场景说明：

- 正向代理的主要场景主要发生在客户端。由于**网络不通**等物理原因，需要通过正向代理服务器这种中间转发环节，顺利访问目标服务器。当然，也可以通过正向代理服务器，对客户端某些详细信息进行一些伪装和改变。
- 反向代理的主要场景主要发生在服务端。服务提供方可以通过反向代理服务器，轻松实现目标服务器的**动态切换**，实现多目标服务器的**负载均衡**等。

通俗点来说，正向代理（如squid、proxy）是对客户端的伪装，隐藏了客户端的IP、头部或者其他的信息，服务器得到的是伪装过的客户端信息；

反向代理（Nginx）是对目标服务器的伪装，隐藏了目标服务器的IP、头部或者其他信息，客户端得到的是伪装过的目标服务器信息。

### 安装Nginx以及其扩展（Openresty)

参考博客：

- [Windows平台Openresty安装和启动（图文死磕)](https://www.cnblogs.com/crazymakercircle/p/12111283.html)
- [Linux平台Openresty安装（图文死磕）](https://www.cnblogs.com/crazymakercircle/p/12115651.html)
- [Openresty服务器下的Lua开发调试（图文死磕）](https://www.cnblogs.com/crazymakercircle/p/12112568.html)

### nginx配置文件上下文结构

nginx可以根据配置文具中的配置指令，就对应到某个模块执行命令。

![image-20211217110544451](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211217110544451.png)

可以简单的认为配置项是一个 key/value对

如果指令参数由简单的字符串组成，就需要用分号结束；指令参数如果是复杂的多行字符串，配置项就需要用花括号`{}`围起来

**配置项的具体功能和他处于的作用域强相关**

### http请求处理的11个阶段

#### post-read阶段

改写请求的来源地址

#### server-rewrite阶段

请求地址重写阶段

**写在server配置块中**

#### find-config阶段

根据请求url地址去匹配location路由表达式

#### rewrite阶段

location中的指令开始生效

可以注册rewrite阶段的指令

例如：

- break
- if
- return
- rewrite
- set
- 第三方lua模块中的set_by_lua指令和rewrite_by_lua指令

#### post-rewrite阶段

请求地址Url重写提交阶段，防止递归修改uri造成死循环

#### preaccess阶段

访问权限检查准备阶段

控制访问频率的模块（ngx_limit_req）和限制并发度的模块（ngx_limit_zone）在这个阶段注册

#### access阶段

访问权限检查阶段

配置的一般都是访问控制类型的任务，检查用户的访问权限、检查用户的来源ip地址是否合法。

#### post-access阶段

访问权限检查提交阶段

如果请求不被运行访问nginx服务器，该阶段负责向用户返回错误的响应。

#### try-files阶段

访问静态文件资源，可以让请求顺序的访问多个静态文件资源。

指令 `try_files`

```nginx
root /var/www/;
# root指令把“查找文件的根目录”配置为 /var/www/
location = /try_files-demo {
	try_files /foo /bar /last;
}
#对应到前面try_files的最后一个URI
location /last {
	echo "uri: $uri ";
}
}
```

try_files指令接受两个以上任意数量的参数，每个参数都指定了一个URI，则Nginx会在try-files
阶段，依次把前N-1个参数映射为文件系统上的对象（文件或者目录），然后检查这些对象是否存在。一旦Nginx发现某个文件系统对象存在，则查找成功，就会在try-files阶段把当前请求的URI改写
为该对象所对应的参数URI（但不会包含末尾的斜杠字符，也不会发生 “内部跳转”）。如果前N-1 个参数所对应的文件系统对象都不存在，try-files阶段就会立即发起“内部跳转”，跳转到最后一个参数（第N个参数）所指定的URI。

#### content阶段

大部分的HTTP模块会介入内容生产阶段，content_by_lua、echo等指令都是在这里注册的。

每一个location只能有一个内容处理程序。

#### log阶段

日志模块处理阶段记录日志

### Nginx的基础配置

#### events事件驱动配置

一个典型的配置

```nginx
events {
use epoll; #使用epoll类型IO多路复用模型
worker_connections 204800; # 最大连接数限制为20W
accept_mutex on; # 各个Worker通过锁来获取新连接
}
```

1. worker_connections 指令

用以配置每个worker进程能够打开的最大并发连接数量。

2. use指令

选择配置IO多路复用的模型，例如select 、epoll、poll

3. accept_mutex指令

配置各个worker是否通过互斥锁，有序接受收到的请求。on表示通过互斥锁有序接受请求，off则会通知所有的worker参与争抢。配置off参数，会造成“惊群”问题影响性能。

#### 虚拟主机配置

> 虚拟主机的套接字配置

listen指令配置监听套接字

1. 直接监听端口

   ```nginx
   server{
       listen 80;
   }
   ```

2. 监听ip和端口

   ```nginx
   server{
       listen 127.0.0.1:80;
   }
   ```

> 虚拟主机名称配置

核心指令 `server_name`

可以用主机名称来做领域区分，例如下面



```nginx
#后台管理服务虚拟主机demo
server {
listen 80;
server_name admin.crazydemo.com; #后台管理服务的域名前缀为admin
    location / {
    default_type 'text/html';
    charset utf-8;
    echo "this is admin server";
    }
}
#文件服务虚拟主机demo
server {
listen 80;
server_name file.crazydemo.com; #文件服务的域名前缀为admin
    location / {
    default_type 'text/html';
    charset utf-8;
    echo "this is file server";
    }
}
#默认服务虚拟主机demo
server {
listen 80 default;
server_name crazydemo.com *.crazydemo.com; #如果没有前缀，这是默认访问的虚拟主机
    location / {
    default_type 'text/html';
    charset utf-8;
    echo "this is defalut server";
    }
....
}
```

**匹配优先级**

1. 字符串精确匹配

例如我请求的地址为admin.crazydemo.com则首先会匹配到名称为admin.crazydemo.com的虚拟管理主机

2. 左侧*通配符匹配

如果精确匹配没有匹配到则会开始匹配左侧*通配符

例如我们请求了 `abc.crazydemo.com` 则会匹配到 *.crazydemo.com虚拟主机。

3. 右侧* 通配符匹配

如果左侧的没有匹配到则执行这个

4. 正则表达式匹配
5. 默认

#### 错误页面配置

指令格式：

```nginx
error_page [code...] [=[response]]uri;
```

例如下面的：

```nginx
#后台管理服务器demo
server {
listen 80;
server_name admin.crazydemo.com;
root /var/www/; 
    location / {
    default_type 'text/html';
    charset utf-8;
    echo "this is admin server";
    }
# 设置错误页面
error_page
404 /404.html;
# 设置错误页面
error_page 500 502 503 504 /50x.html;
}
```

为了防止404页面被劫持，可以修改掉响应状态码。

```nginx
error_page 404 =200 /404.html;
```

#### 长连接相关配置

#### Nginx核心模块内置变量


（1）$arg_PARAMETER： 请求URL中的以PARAMETER为名称的参数值。请求参数即URL
的“?”号后面的name=value形式的参数对，$arg_name得到的值为value。另外，$arg_PARAMETER中的参数名称不区分大小写，$arg_name不仅可以匹配name参数，也可以匹配NAME、Name请求参数，Nginx会在匹配参数名之前，自动把原始请求中的参数名调整为全部小写的形式。
（2）$args：请求URL中的整个参数串，其作用与$query_string相同。
（3）$binary_remote_addr：二进制形式的客户端地址。
（4）$body_bytes_sent：传输给客户端的字节数，响应头不计算在内。
（5）$bytes_sent：传输给客户端的字节数，包括响应头和响应体。
（6）$content_length：等同于$http_content_length，用于获取请求体body的大小。指的是Nginx从客户端收到的请求头中Content-Length字段的值，不是发送给客户端响应中的Content-Length字段值，如果需要获取响应中的Content-Length字段值，则使用$sent_http_content_length变量。
（7）$request_length：请求的字节数（包括请求行、请求头和请求体）。注意，由于$request_length是请求解析过程中不断累加的，如果解析请求时出现异常，则$request_length只是已经累加部分的长度，并不是Nginx从客户端收到的完整请求的总字节数（包括请求行、请求头、请求体）。
（8）$connection：TCP连接的序列号。
（9）$connection_requests：TCP连接当前的请求数量。
（10）$content_type：请求中的Content-Type请求头字段值。
（11）$cookie_name：请求中名称name的cookie值。
（12）$document_root：当前请求的文档根目录或别名。
（13）$uri：当前请求中的URI（不带请求参数，参数位于$args变量）。$uri变量值不包含主机名，如“/foo/bar.html”。此参数可以修改，可以通过内部重定向。
（14）$request_uri：包含客户端请求参数的原始URI，不包含主机名，此参数不可以修改，例如：“/foo/bar.html? name=value”。
（15）$host：请求的主机名。优先级如下：HTTP请求行的主机名 > HOST请求头字段> 符合请求的服务器名。
（16）$http_name：名称为name的请求头的值。如果实际请求头name中包含中划线“-”，那么需要将中划线“-”替换为下划线“_”；如果实际请求头name中包含大写字母，可以替换为小写字母。如获取Accept-Language请求头的值，变量名称为$http_accept_language。
（17）$msec：当前的Unix时间戳。Unix时间戳是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数，不考虑闰秒。
（18）$nginx_version：获取Nginx版本。
（19）$pid：获取Worker工作进程的PID。
（20）$proxy_protocol_addr：代理访问服务器的客户端地址，如果是直接访问，该值为空字符串。
（21）$realpath_root：当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径。
（22）$remote_addr：客户端请求地址。
（23）$remote_port：客户端请求端口。
（24）$request_body：客户端的请求主体。此变量可在location中使用，将请求主体通过proxy_pass、fastcgi_pass、uwsgi_pass和scgi_pass传递给下一级的代理服务器。
（25）$request_completion：如果请求成功，值为“OK”，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空。
（26）$request_filename：当前请求的文件路径，由root或alias指令与URI请求结合生成。
（27）$request_length：请求的长度，包括请求的地址、HTTP请求头和请求主体。
（28）$request_method：HTTP请求方法，比如“GET”或“POST”等。
（29）$request_time：处理客户端请求使用的时间，从读取客户端的第一个字节开始计时。
（30）$scheme：请求使用的Web协议，如“http”或“https”。
（31）$sent_http_name：设置任意名称为name的HTTP响应头字段。如需要设置响应头Content-length，那么将“－”替换为下划线，大写字母替换为小写，变量为$sent_http_content_length。
（32）$server_addr：服务器端地址为了避免访问操作系统内核，应将IP地址提前设置在配置文件中。
（33）$server_name：虚拟主机的服务器名，如crazydemo.com。
（34）$server_port：虚拟主机的服务器端口。
（35）$server_protocol：服务器的HTTP版本，通常为“HTTP/1.0”或“HTTP/1.1”。
（36）$status：HTTP响应代码。
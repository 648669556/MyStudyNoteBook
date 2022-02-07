### 安装集群环境

#### 安装虚拟机环境

https://zhuanlan.zhihu.com/p/259833884

具体的虚拟机安装看这个进行

相应的box在我的阿里云盘上

#### 启动多个集群 配置

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|

    #需要启动的虚拟机的台数
  (1..3).each do |i|
  # boxes at https://vagrantcloud.com/search.
    config.vm.define "node#{i}" do |node|
        # 相应的box名称
    node.vm.box = "chen/centos-7-base"
        # 节点名称
    node.vm.hostname = "node#{i}"
        # 私有网络id地址
    node.vm.network "private_network", ip: "192.168.56.1#{i}"
        # 同步文件夹
    node.vm.synced_folder "./data", "/home/vagrant/data"
        # 虚机设置
     node.vm.provider "virtualbox" do |vb|
         #节点名称
      vb.name = "node#{i}"
         #分配内存
      vb.memory = "1024"
         #分配cpu核心数
      vb.cpus = 1
     end
    end
  end
end
```

### 下载zookeeper

https://downloads.apache.org/zookeeper/

选择相应的版本进行下载就可以了

重点在集群的配置上

当然你还要保证你的java jdk已经安装了

### 配置zookeeper集群

- 首先要确保你的虚拟机的防火墙已经关闭了
- 可以相互ping 通

> zookeeper项目大体

![image-20211215102317163](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211215102317163.png)

- bin 就是我们程序平常启动关闭那些软件的入口

- conf就是我们的zookeeper配置存储的地方

- logs就是我们日志存储的地方

- data文件夹和log文件夹都是我们创建的

为了方便集群的配置我们创建了conf文件夹和data文件夹。他们的内部有结构是这样的

> **conf**

  ![image-20211215102734811](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211215102734811.png)   

> data

<img src="https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211215102807198.png" alt="image-20211215102807198" style="zoom:50%;" />



下面要做的有两步：

1. 在data文件夹下的每个zoo文件夹下面创建一个myid文件

![image-20211215102932723](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211215102932723.png)

这个文件全称就叫myid 没有后缀名

里面的内容就是 一个数字‘1’  也就是你节点的编号

zoo-1 下面就创建一个内容为 1 的myid文件，其他的zoo文件夹类似。

2. 复制zoo_sample.cfg 为 zoo.cfg来配置我们自己的配置。

> zoo-1下的zoo.cfg 文件内容

```ruby
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/home/vagrant/data/zookeeper/data/zoo-1
dataLogDir = /home/vagrant/data/zookeeper/log/zoo-1
# the port at which the clients will connect
clientPort=2181
server.1 = 0.0.0.0:2888:3888
server.2 = 192.168.56.12:2888:3888
server.3 = 192.168.56.13:2888:3888
```

> zoo-2 下的zoo.cfg 内容

```ruby
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/home/vagrant/data/zookeeper/data/zoo-2
dataLogDir = /home/vagrant/data/zookeeper/log/zoo-2
# the port at which the clients will connect
clientPort=2181
server.1 = 192.168.56.11:2888:3888
server.2 = 0.0.0.0:2888:3888
server.3 = 192.168.56.13:2888:3888
```

> zoo-3 下的zoo.cfg内容

```ruby
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/home/vagrant/data/zookeeper/data/zoo-3
dataLogDir = /home/vagrant/data/zookeeper/log/zoo-3
# the port at which the clients will connect
clientPort=2181
server.1 = 192.168.56.11:2888:3888
server.2 = 192.168.56.12:2888:3888
server.3 = 0.0.0.0:2888:3888

```

这里注意到我们本机上的 ip地址为 0.0.0.0才行，



可能还是运行不起来，我们需要把 conf文件夹下面的 `log4j.properties` 文件都复制一份到zoo-*文件夹下面。不然可能会因为没有配置文件无法启动。



> 然后就是到我们的bin目录下

![image-20211215103812615](https://myselfd.oss-cn-hangzhou.aliyuncs.com/uPic/image-20211215103812615.png)

我们这里主要使用到的是 `zkServer.sh` 或者 `zkServer.cmd` 来启动我们的zookeeper 。具体调用那个需要看我们的操作系统，如果是windows那么就是cmd文件，如果是mac或者linux那么就是sh文件。

不过sh文件应该都可以尝试调用一下。在linux系统上我使用这样的命令来启动我们不同配置下的zookeeper

```shell
./zkServer.sh --config ../conf/zoo-1 start
```

这样来启动我们的zookeeper



### 检查运行状态

查看zookeeper是否正常运行的命令

```shell
./zkServer.sh --config ../conf/zoo-1 status
```

如果没有正常运行可以去 `zookeeper/logs`文件夹下面看对应的日志信息。

### zookeeper相关命令

> get命令

获取某个节点的信息，注意一定是`/`开头

> stat

stat命令查看节点的状态信息

```properties
cZxid = 0x100000002
ctime = Tue Dec 14 12:40:05 UTC 2021
mZxid = 0x100000002
mtime = Tue Dec 14 12:40:05 UTC 2021
pZxid = 0x100000009
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 10
numChildren = 1
```

- cZxid - 节点创建时的zxid
- ctime - 节点创建时间
- mZxid - 节点最近一次更新时的zxid
- mtime - 节点最近一次更新的时间
- cversion - 子节点数据更新次数
- dataVersion - 本届点数据更新次数
- aclVersion - 节点acl（授权信息）的更新次数
- numChildren - 子节点个数

> set

设置节点数据的

```sh
set /test 8888888
```

> ls

　ls命令用于获取路径下的节点信息，注意路径为绝对路径，如:ls /storm

> ls2

比ls多输出一个本节点信息 好比 ls + stat

> listquota

显示配额

> setquota

设置某个节点的节点个数和数据长度的配额

> delquota

删除配额

> create 

create命令用于创建节点，其中-s为顺序充点，-e临时节点　

```sh
create /zookeeper/node1"test_create" world:anyone:fdsfds
```

> delete

　delete命令用于删除节点，如delete /nodeDelete

> addauth

addauth命令用于节点认证，使用方式：如addauth digest username:password

> setAcl

setAcl命令用于设置节点Acl

　　Acl由三部分构成：1为scheme，2为user，3为permission，一般情况下表示为scheme🆔permissions

> getAcl

　获取节点的Acl，如getAcl /node1

scheme和id

**world**: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的

**auth**: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)

**digest**: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication

**ip**: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段

**super**: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)

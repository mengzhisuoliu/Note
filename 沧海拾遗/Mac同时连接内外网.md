# Mac同时连接内外网

很多公司或者机构为了安全性，内部网络和外部网络是不联通的，但是有时候开发过程中同时需要内外网环境，例如：使用外网下载依赖包和构建工具，但程序需要在内网运行才能访问内网服务。这时候如果需要来回的切换网络就比较麻烦了，此时可以借用 Mac 的双网卡同时连接内外网。

### 1. 前置条件

首先你的 Mac 需要一个网线接口，由于新版 Mac 都不带网线接口，所以需要买一个接口转换器，以便可以使用有线网卡。

加入说我们有两个网络，一个是内网，一个是外网，那么请让其中一个走无线网络，另一个走有线网络。例如：**内网使用有线，外网使用无线。**

> 通常保密等级比较高的内网环境都是不开放无线网络的，因此大部分情况下，内网环境都需要使用有线网络。

### 2. 查看路由表

```shell
# 运行之后你就能看到路由表信息了
$ netstat -rt

Routing tables

Internet:
Destination        Gateway            Flags        Refs      Use   Netif Expire
default            192.168.3.1        UGSc          262        0     en0       
127                localhost          UCS             0     7173     lo0     
```

注意网关地址 Gateway，如果连接有多个网络，通常情况下，它们的网关地址是不应该相同的，否则可能引起冲突，导致网络无法访问。

> 第一列：Destination，目标地，意思是：后面的参数代表着，如果前往这个ip的话，应该如何分配网关，网卡等，以及状态信息，都是针对前往这个ip的情况的
>
> 第二列：Gateway，网关，意思是：如果需要前往这个ip，应该从哪个网关过去，这里有两种情况，即有内外网用不同网关的，也有内外网用相同网关的。
>
> 第三列：Flags，标志位，和本文的重点无关，暂略
>
> 第四列：Refs，可以简单的理解为重要性，相同的ip，相同的网关，用这个重要性来区分使用哪个网卡
>
> 第五列：Use，使用情况
>
> 第六列：Netif，网卡号，net interface，如图，我有两张网卡，这里的en0，en10分别代表了我的两张网卡，不同的机器名称可能不同，但是意思是一样的，至于那张是无线网卡，需要自己去区分
>
> 第七列：Expire，和本文的重点无关，暂略

### 3. 设置外网优先

#### 3.1 打开网络偏好设置

- 点击菜单栏上 wifi 图标，然后选择 **打开网络偏好设置**
- 或者左上角**苹果图标** => **系统偏好设置** => **网络**

#### 3.2 设定服务顺序

**查看两个网络是否均已经成功连接，如果成功，则设置一下优先级。**

![QQ20190627-225217@2x](http://gcsblog.oss-cn-shanghai.aliyuncs.com/blog/2019-06-27-150742.jpg?gcssloop)

**通过拖拽把优先级高的排到上面，我的 Wi-Fi 是外网，则把 Wi-Fi 排到上面。**

![QQ20190627-225303@2x](http://gcsblog.oss-cn-shanghai.aliyuncs.com/blog/2019-06-27-150741.jpg?gcssloop)

### 4. 为特定服务设定内网网关

按照上面的排序，则请求默认都会走外网的网关，此时我们需要把内网服务指定特定的内网网关。

假设服务地址是 192.168.1.102，内网网关地址是 192.168.1.225，则通过下面命令让其走内网网关。

```shell
sudo route add -net 192.168.1.102 192.168.1.225
```

### 5. 其它

一、公司里内外网分两个路由：

此时只需要修改前往公司内网地址的网关就可以了，比如你需要访问的内网地址是10.10.15.*，而公司的内网网关是10.10.15.255，那么就这样写 

> sudo route add -net 10.10.15.0 -netmask 255.255.255.0 10.10.15.255

其中，0表示的是默认default，netmask是子网掩码，不是重点就不提了。

二、公司里内外网是一个路由：

此时需要有线网卡转发内网链接，无线网卡转发外网链接，写法如下

> sudo route add -net 10.10.15.0 -netmask 255.255.255.0 -interface en10

如果存在的话先sudo route delete一下。


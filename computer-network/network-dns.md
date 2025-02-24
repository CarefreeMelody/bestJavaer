# DNS 协议

* [DNS 协议](#dns-协议)
   * [DNS 基本概述](#dns-基本概述)
   * [DNS 工作概述](#dns-工作概述)
      * [分布式、层次数据库](#分布式层次数据库)
      * [DNS 层次结构](#dns-层次结构)
   * [DNS 查询步骤](#dns-查询步骤)
      * [DNS 解析器](#dns-解析器)
      * [DNS 查询类型](#dns-查询类型)
   * [DNS 缓存](#dns-缓存)
      * [DNS 缓存的工作流程](#dns-缓存的工作流程)
      * [DNS 缓存方式](#dns-缓存方式)
         * [浏览器缓存](#浏览器缓存)
         * [操作系统内核缓存](#操作系统内核缓存)
   * [DNS 报文](#dns-报文)
      * [报文段首部](#报文段首部)
      * [资源记录部分](#资源记录部分)
      * [SOA 记录](#soa-记录)
   * [DNS 安全](#dns-安全)
      * [DNS 防火墙](#dns-防火墙)
   * [总结](#总结)

> 这是计算机网络连载系列的第八篇文章，前七篇文章见
>
> [计算机网络基础知识总结](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247486242&idx=1&sn=fac49b0b79515a5ed6afd4b341aff87b&chksm=e999fe30deee772637e1c52fb9001c60e60a772e7adba6701329c81974e76c57bb7b2e570225&token=850264305&lang=zh_CN#rd)
>
> [TCP/IP 基础知识总结](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247486408&idx=1&sn=c332ae7ae448f3eb98865003ecade589&chksm=e999fedadeee77cc6281d1b170bd906b58220d6cd83054bc741821f4167f1f18ceee9ba0e449&token=850264305&lang=zh_CN#rd)
>
> [计算机网络应用层](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247486507&idx=1&sn=622cc363b34bce54f4953076faa1cad6&chksm=e999f939deee702f2444df83ad9805de8c70fb88b89d299fdf0a82b3463e253f32372963c039&token=1398464113&lang=zh_CN#rd)
>
> [计算机网络传输层](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247487108&idx=1&sn=7b47f421bb1dee4edb357a10399b7fec&chksm=e999fb96deee7280a17bfff44c27ef11a60e93e48f9da738670a779ecf6accb5a6a4ebd3cbcc&token=1398464113&lang=zh_CN#rd)
>
> [计算机网络网络层](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247487683&idx=1&sn=e0949e72e039759545450852d8bc0ada&chksm=e999e5d1deee6cc7ab9e42b50329924fee39c45955516b406046605d27928825a0f628d13e7c&token=1398464113&lang=zh_CN#rd)
>
> [计算机网络链路层](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247488884&idx=1&sn=0fdb91b7f5081d2e24c82d891fcc6126&chksm=e999e066deee69704d162b97be2ff0d33225fa9a3d12e4d3bec90a34996e7db7134535f36e8e&token=1398464113&lang=zh_CN#rd)
>
> [计算机网络 ARP 协议](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247487804&idx=1&sn=f001a24a308053b3723dfb12d36045ee&chksm=e999e42edeee6d383fbb411792e22e4028bb8c2441255786f50cf848443af7b1bd5e382078dc&token=1398464113&lang=zh_CN#rd)

试想一个问题，我们人类可以有多少种识别自己的方式？可以通过身份证来识别，可以通过社保卡号来识别，也可以通过驾驶证来识别，尽管有多种识别方式，但在特定的环境下，某种识别方法会比其他方法更为适合。因特网上的主机和人类一样，可以使用多种方式进行标识。互联网上主机的一种标识方法是使用它的主机名，比如 www.baidu.com、www.google.com 等。这是我们人类习惯的记忆方式，因特网中的主机却不会这么记忆，它们喜欢定长的、有层次结构的 IP 地址。

那么路由器如何把 IP 地址解析为我们熟悉的网址呢？这时候就需要 DNS 出现了。

![image-20220522221654727](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522221654727.png)

<div align = "center">图 8-1</div>

DNS 的全称是 *Domain Name Systems*，它是一个由分层的*DNS 服务器（DNS server）*实现的分布式数据库；它还是一个使得主机能够查询分布式数据库的应用层协议。DNS 协议运行在 UDP 协议上，使用 53 端口。

## DNS 基本概述

与 HTTP、FTP 和 SMTP 一样，DNS 协议也是一种应用层的协议，DNS 使用**客户-服务器**模式运行在通信的端系统之间，在通信的端系统之间通过 UDP 运输层协议来传送 DNS 报文。

DNS 通常不是一门独立的协议，它通常为其他应用层协议所使用，这些协议包括 HTTP、SMTP 和 FTP，将用户提供的主机名解析为 IP 地址。

下面根据一个示例来描述一下 DNS 解析过程：

你在浏览器键入 www.someschool.edu/index.html 时会发生什么？为了使用户主机能够将一个 HTTP 请求报文发送到 Web 服务器 www.someschool.edu ，会经历如下操作：

* 同一台用户主机上运行着 DNS 应用的客户端。
* 浏览器从上述 URL 中抽取出主机名 www.someschool.edu ，将这台主机名传给 DNS 应用的客户端。
* DNS 客户端向 DNS 服务器发送一个包含主机名的请求，请求 DNS 服务器解析这个主机名的 IP 地址。
* DNS 客户端最终会收到一份回答报文，其中包含该目标主机的 IP 地址。
* 一旦浏览器收到目标主机的 IP 地址后，它就能够向位于该 IP 地址 80 端口的 HTTP 服务器进程发起一个 TCP 连接。

除了提供 IP 地址到主机名的转换，DNS 还提供了下面几种重要的服务：

* `主机别名(host aliasing)`，有着复杂主机名的主机能够拥有一个或多个其他别名，比如说一台名为 relay1.west-coast.enterprise.com 的主机，同时会拥有 enterprise.com 和 www.enterprise.com 的两个主机别名，在这种情况下，relay1.west-coast.enterprise.com 也称为**规范主机名**，而主机别名要比规范主机名更加容易记忆。应用程序可以调用 DNS 来获得主机别名对应的规范主机名以及主机的 IP地址。

* `邮件服务器别名(mail server aliasing)`，同样的，电子邮件的应用程序也可以调用 DNS 对提供的主机名进行解析。

* `负载分配(load distribution)`，DNS 也用于冗余的服务器之间进行负载分配，这种负载又叫做**内部负载**。繁忙的站点例如  cn com 被冗余分布在多台服务器上，每台服务器运行在不同的端系统，每个都有着不同的 IP 地址。由于这些冗余的 Web 服务器，一个 IP 地址集合因此与同一个规范主机名联系。DNS 数据库中存储着这些 IP 地址的集合。由于客户端每次都会发起 HTTP 请求，所以 DNS 就会在所有这些冗余的 Web 服务器之间循环分配了负载。

  还有一种负载是**全局负载**，全局负载一般部署在多个机房之间，每个机房都会有自己的 IP 地址，当用户访问某个域名时，会在这些 IP 之间进行轮询，如果某个数据中心挂了，就会将对应的 IP 地址删除，比如某个 DNS 客户端会轮询访问北京和上海的机房，一个挂了会直接使用另外一个，这就是全局负载的概念。

## DNS 工作机制

假设运行在用户主机上的某些应用程序（如 Web 浏览器或邮件阅读器） 需要将主机名转换为 IP 地址。这些应用程序将调用 DNS 的客户端，并指明需要被转换的主机名。DSN 客户端收到 DNS 后，会使用 UDP 通过 53 端口向网络上发送一个 DNS 查询报文，经过一段时间后，DNS 客户端会收到一个主机名对应的 DNS 应答报文。因此，从用户主机的角度来看，DNS 就像是一个黑盒子，其内部的操作你无法看到。但是实际上，实现 DNS 这个服务的黑盒子非常复杂，它由分布于全球的大量 DNS 服务器以及定义了 DNS 服务器与查询主机通信方式的应用层协议组成。

DNS 最早的设计是只有一台 DNS 服务器。这台服务器会包含所有的 DNS 映射。这是一种集中式单点设计，这种设计并不适用于当今的互联网，因为互联网有着数量巨大并且持续增长的主机，这种集中式的设计会存在以下几个问题

* `单点故障(a single point of failure)`，单点通常上只有一台 DNS 服务器，如果 DNS 服务器崩溃，那么整个网络随之瘫痪。
* `通信容量(traaffic volume)`，单个 DNS 服务器不得不处理所有的 DNS 查询，这种查询级别可能是上百万上千万级。
* `远距离集中式数据库(distant centralized database)`，单个 DNS 服务器不可能靠近所有的用户，假设在美国的 DNS 服务器不可能临近让澳大利亚的查询使用，其中查询请求势必会经过低速和拥堵的链路，造成严重的时延。
* `维护(maintenance)`，维护成本巨大，而且还需要频繁更新。

所以 DNS 不可能集中式设计，因为集中式设计完全没有可扩展能力，因此采用**分布式设计**，这种设计的特点如下。

### 分布式、层次数据库

分布式设计首先解决的问题就是 DNS 服务器的扩展性问题。因此 DNS 使用了大量的 DNS 服务器，它们的组织模式一般是层次方式，并且分布在全世界范围内。**没有一台 DNS 服务器能够拥有因特网上所有主机的映射**。相反，这些映射分布在所有的 DNS 服务器上。

大致来说有三种 DNS 服务器：*根 DNS 服务器、 顶级域(Top-Level Domain, TLD) DNS 服务器和权威 DNS 服务器*。这些服务器的层次模型如下图所示。

![image-20220522222621839](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222621839.png)

<div align = "center">图 8-2</div>

假设现在一个 DNS 客户端想要知道 www.amazon.com 的 IP 地址，那么上面的域名服务器是如何解析的呢？

首先，客户端会先根服务器之一进行关联，它将返回顶级域名 com 的 TLD 服务器的 IP 地址。然后客户端与这些 TLD 服务器之一联系，它将为 amazon.com 返回权威服务器的 IP 地址。最后，该客户与 amazom.com 权威服务器之一联系，它为 www.amazom.com 返回其 IP 地址。

### DNS 层次结构

我们现在来讨论一下上面域名服务器的层次系统。

* `根 DNS 服务器` ，有 400 多个根域名服务器遍及全世界，这些根域名服务器由 13 个不同的组织管理。根域名服务器的清单和组织机构可以在 https://root-servers.org/ 中找到，根域名服务器提供 TLD 服务器的 IP 地址。
* `顶级域 DNS 服务器`，对于每个顶级域名比如 com、org、net、edu 和 gov 和所有的国家级域名 uk、fr、ca 和 jp 都有 TLD 服务器或服务器集群。所有的顶级域列表参见 https://tld-list.com/ 。TDL 服务器提供了权威 DNS 服务器的 IP 地址。
* `权威 DNS 服务器`，在因特网上具有公共可访问的主机，如 Web 服务器和邮件服务器，这些主机的组织机构必须提供可供访问的 DNS 记录，这些记录将这些主机的名字映射为 IP 地址。一个组织机构的权威 DNS 服务器收藏了这些 DNS 记录。

## DNS 查询步骤

下面我们来描述一下 DNS 的查询步骤，从 DNS 解析 IP 再到 DNS 报文返回的一系列流程。

>注意：通常情况下 DNS 会将查找的信息缓存在浏览器或者计算机本地中，如果有相同的请求到来时，就不再会进行 DNS 查找，而会直接返回结果。

通常情况下，DNS 的查找会经历下面这些步骤

1. 用户在浏览器中输入网址 www.example.com 并点击回车后，查询会进入网络，并且由 DNS 解析器进行接收。

2. DNS 解析器会向根域名发起查询请求，要求返回顶级域名的地址。

3. 根 DNS 服务器会注意到请求地址的前缀并向 DNS 解析器返回 com 的顶级域名服务器(TLD)的 IP 地址列表。

4. 然后，DNS 解析器会向 TLD 服务器发送查询报文。

5. TLD 服务器接收请求后，会根据域名的地址把权威 DNS 服务器的 IP 地址返回给 DNS 解析器。

6. 最后，DNS 解析器将查询直接发送到权威 DNS 服务器。

7. 权威 DNS 服务器将 IP 地址返回给 DNS 解析器。

8. DNS 解析器将会使用 IP 地址响应 Web 浏览器。

一旦 DNS 查找的步骤返回了 example.com 的 IP 地址，浏览器就可以请求网页了。

整个流程如下图所示

![image-20220522222702760](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222702760.png)

<div align = "center">图 8-3</div>

### DNS 解析器

进行 DNS 查询的主机和软件叫做 *DNS 解析器*，用户所使用的工作站和个人电脑都属于解析器。一个解析器要至少注册一个以上域名服务器的 IP 地址。DNS 解析器是 DNS 查找的第一站，其**负责与发出初始请求的客户端打交道**。解析器启动查询序列，最终使 URL 转换为必要的 IP 地址。

![image-20220522222728653](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222728653.png)

<div align = "center">图 8-4</div>

DNS 递归查询和 DNS 递归解析器不同，该查询是指向需要解析该查询的 DNS 解析器发出请求。DNS 递归解析器是一种计算机，其接受递归查询并通过发出必要的请求来处理响应。

### DNS 查询类型

DNS 查找中会出现三种类型的查询。通过组合使用这些查询，**优化的 DNS 解析过程可缩短传输距离**。在理想情况下，可以使用缓存的记录数据，从而使 DNS 域名服务器能够直接使用非递归查询。

1. `递归查询`：在递归查询中，DNS 客户端要求 DNS 服务器（一般为 DNS 递归解析器）将使用所请求的资源记录响应客户端，或者如果解析器无法找到该记录，则返回错误消息。

   ![image-20220522222808624](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222808624.png)

   <div align = "center">图 8-5</div>

2. `迭代查询`：在迭代查询中，如果所查询的 DNS 服务器与查询名称不匹配，则其将返回对较低级别域名空间具有权威性的 DNS 服务器的引用。然后，DNS 客户端将对引用地址进行查询。此过程继续使用查询链中的其他 DNS 服务器，直至发生错误或超时为止。

   ![image-20220522222823402](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222823402.png)

   <div align = "center">图 8-6</div>

3. `非递归查询`：当 DNS 解析器客户端查询 DNS 服务器以获取其有权访问的记录时通常会进行此查询，因为其对该记录具有权威性，或者该记录存在于其缓存内。DNS 服务器通常会缓存 DNS 记录，查询到来后能够直接返回缓存结果，以防止更多带宽消耗和上游服务器上的负载。

## DNS 缓存

*DNS 缓存(DNS caching)* 有时也叫做*DNS 解析器缓存*，它是**由操作系统维护的临时数据库**，它包含有**最近的网站和其他 Internet 域的访问记录**。也就是说， DNS 缓存只是计算机为了满足快速的响应速度而把已加载过的资源缓存起来，再次访问时可以直接快速引用的一项技术和手段。那么 DNS 的缓存是如何工作的呢？

### DNS 缓存的工作流程

在浏览器向外部发出请求之前，计算机会拦截每个请求并在 DNS 缓存数据库中查找域名，该数据库包含有最近的域名列表，以及 DNS 首次发出请求时 DNS 为它们计算的地址。

### DNS 缓存方式

DNS 数据可缓存到各种不同的位置上，每个位置均将存储 DNS 记录，它的生存时间由 TTL(DNS 字段) 来决定。

#### 浏览器缓存

现如今的 Web 浏览器设计默认将 DNS 记录缓存一段时间。因为越靠近 Web 浏览器进行 DNS 缓存，为检查缓存并向 IP 地址发出请求的次数就越少。发出对 DNS 记录的请求时，浏览器缓存是针对所请求的记录而检查的第一个位置。

在 chrome 浏览器中，你可以使用 chrome://net-internals/#dns 查看 DNS 缓存的记录。

![image-20220522222836592](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222836592.png)

<div align = "center">图 8-7</div>

#### 操作系统内核缓存

在浏览器缓存查询后，会进行操作系统级 DNS 解析器的查询，操作系统级 DNS 解析器是 DNS 查询离开你的计算机前的第二站，也是本地查询的最后一个步骤。

## DNS 报文

共同实现 DNS 分布式数据库的所有 DNS 服务器存储了*资源记录(Resource Record, RR)*，RR 提供了主机名到 IP 地址的映射。每个 DNS 回答报文中会包含一条或多条资源记录。RR 记录用于回复客户端查询。

资源记录是一个包含了下列字段的 4 元组。

```
(Name, Value, Type, TTL)
```

RR 会有不同的类型，下面是不同类型的 RR 汇总表。

| DNS RR 类型 | 解释                                        |
| ----------- | ------------------------------------------- |
| A 记录      | IPv4 主机记录，用于将域名映射到 IPv4 地址   |
| AAAA 记录   | IPv6 主机记录，用于将域名映射到 IPv6 地址   |
| CNAME 记录  | 别名记录，用于映射 DNS 域名的别名           |
| MX 记录     | 邮件交换器，用于将 DNS 域名映射到邮件服务器 |
| PTR 记录    | 指针，用于反向查找（IP地址到域名解析）      |
| SRV 记录    | SRV记录，用于映射可用服务。                 |

<div align = "center">表 8-1</div>

DNS 有两种报文，一种是查询报文，一种是响应报文，并且这两种报文有着相同的格式，下面是 DNS 的报文格式。

![image-20220522222853912](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222853912.png)

<div align = "center">图 8-8</div>

下面我们就来看一下详细的报文字段。

### 报文段首部

报文段首部是 DNS 报文的基础结构部分，下面我们对报文段首部中的每个字节进行描述。

* 事务 ID: TransactionID 由客户端设置，由服务器返回。TransactionID 占用 2 个字节。它是 DNS 的标识，对于同一个请求报文和响应报文来说，这个字段的值是相同的，以此来区分客户端请求和响应。
* 标志：标志字段占用 2 个字节。标志字段有很多，而且也比较重要，下面我给你列出来了所有的标志字段。

![image-20220522222904531](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222904531.png)

<div align = "center">图 8-9</div>

每个字段的含义如下

* `QR(Response)`: 1 bit 的 QR 标识报文是查询报文还是响应报文，查询报文时 QR = 0，响应报文时 QR = 1。
* `OpCode`: 4 bit 的 OpCode 表示操作码，这个值通常是 0 ，代表标准的请求和响应。OpCode = 4 表示这是一个通知；OpCode = 5 表示这是一个更新请求。而其他值（1-3）是被弃用的。
* `AA(Authoritative)`: 1 bit 的 AA 代表授权应答，这个 AA 只在响应报文中有效，值为 1 时，表示名称服务器是权威服务器；值为 0 时，表示不是权威服务器。
* `TC(Truncated)`: 截断标志位，值为 1 时，表示响应已超过 512 字节并且已经被截断，只返回前 512 个字节。
* `RD(Recursion Desired)`: 这个字段是期望递归字段，该字段在查询中设置，并在响应中返回。该标志告诉名称服务器必须处理这个查询，这种方式被称为一个**递归查询**。如果该位为 0，且被请求的名称服务器没有一个授权回答，它将返回一个能解答该查询的其他名称服务器列表。这种方式被称为**迭代查询**。
* `RA(Recursion Available)`: 可用递归字段，这个字段只出现在响应报文中。当值为 1 时，表示服务器支持递归查询。
* `Z`: 保留字段，在所有的请求和应答报文中，它的值必须为 0。
* `AD`: 这个字段表示信息是否是已授权，已授权就是 true。
* `CD`: 这个字段表示是否禁用安全检查，禁用检查就是 true。

- `rcode（Reply code）`：这个字段是返回码字段，表示响应的差错状态。当值为 0 时，表示没有错误；当值为 1 时，表示报文格式错误（Format error），服务器不能理解请求的报文；当值为 2 时，表示域名服务器失败（Server failure），因为服务器的原因导致没办法处理这个请求；当值为 3 时，表示名字错误（Name Error），只有对授权域名解析服务器有意义，指出解析的域名不存在；当值为 4 时，表示查询类型不支持（Not Implemented），即域名服务器不支持查询类型；当值为 5 时，表示拒绝（Refused），一般是服务器由于设置的策略拒绝给出应答，如服务器不希望对某些请求者给出应答。

相信读者跟我一样，只看这些字段没什么意思，下面我们就通过抓包的方式，看一下具体的 DNS 报文。

![image-20220522222916495](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522222916495.png)

<div align = "center">图 8-10</div>

现在我们可以看一下具体的 DNS 报文，通过 `query` 可知这是一个请求报文，这个报文的标识符是 `0xcd28`，它的标志如下。

* QR = 0 实锤了这就是一个请求。
* 然后是四个字节的 OpCode，它的值是 0，表示这是一个标准查询。
* 因为这是一个查询请求，所以没有 AA 字段出现。
* 然后是截断标志位 Truncated，表示没有被截断。
* 紧随其后的 RD = 1，表示希望得到递归回答。
* 请求报文中没有 RA 字段出现。
* 然后是保留字段 Z。
* 紧随其后的 0 表示未经身份验证的数据是不可接受的。
* 没有 RCODE 字段的值。

然后我们看一下响应报文。

![image-20220522223057224](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223057224.png)

<div align = "center">图 8-11</div>

可以看到，标志位也是 `0xcd28`，可以说明这就是上面查询请求的响应。

查询请求已经解释过的报文我们这里就不再说明了，现在只解释一下请求报文中没有的内容。

* 紧随在 OpCode 后面的 AA 字段已经出现了，它的值为 0 ，表示不是权威 DNS 服务器的响应。
* 最后是 RCODE 字段的响应，值为 0 时，表示没有错误。

### 查询区

查询区通常指报文格式中查询的部分。这部分用来显示 DNS 查询请求的问题，包括查询类型和查询类。

![image-20220522223107310](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223107310.png)

<div align = "center">图 8-12</div>

这部分中每个字段的含义如下：

* `查询名（Query Name）`：指定要查询的域名，有时候也是 IP 地址，用于反向查询。
* `查询类型（Query Type）`：DNS 查询请求的资源类型，通常查询类型为 A 类型，表示由域名获取对应的 IP 地址。
* `查询类（Query Class）`：地址类型，通常为互联网地址，值为 1 。这个查询类的值通常是 1、254 和 255，分别表示互联网类、没有此类和所有类。

同样的，我们再使用 wireshark 查看一下查询区域。

![image-20220522223118788](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223118788.png)

<div align = "center">图 8-13</div>

可以看到，这是对 mobile-gtalk.l.google.com 发起的 DNS 查询请求，查询类型是 A（0x0001），那么得到的响应类型应该也是 A ，A 表示的是 IPv4 类型，如果 Type 是 AAAA，那么就表示的是 IPv6 类型。

![image-20220522223129433](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223129433.png)

<div align = "center">图 8-14</div>

如上图所示，响应类型也是 A。

### 资源记录部分

资源记录部分是 DNS 报文的最后三个字段，包括回答问题区域、权威名称服务器记录、附加信息区域，这三个字段均采用一种称为资源记录的格式，如下图所示。

![image-20220522223142611](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223142611.png)

<div align = "center">图 8-15</div>

资源记录部分的字段含义如下

* `Name`：DNS 请求的域名。
* `Type`：资源记录的类型，与查询部分中的查询类型是一样的。
* `Class`：地址类型、与问题中的查询类值一样的。
* `TTL`：以秒为单位，表示资源记录的生命周期。
* `RDLENGTH（资源数据长度）`：资源数据的长度。
* `RDATA（资源数据）`：表示按查询段要求返回的相关资源记录的数据。

资源记录部分只有在 DNS 响应包中才会出现。下面我们就来通过响应报文看一下具体的字段示例。

![image-20220522223154603](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223154603.png)

<div align = "center">图 8-16</div>

其中，域名的值是 mobile-gtalk.l.google.com ，类型是 A，类是 1，生存时间是 5 秒，数据长度是 4 字节，资源数据表示的地址是 63.233.189.188。

### CNAME 记录

CNAME 是 DNS 的一种记录类型，它的全称是 *Canonical Name Record*，这个类型能够将某些 DNS 别名映射到 DNS 命名系统中。

一个很简单的例子，如下所示

```
www.cxuanblog.edu  
IN  
CNAME  
www.cxuanblog.com
```

这是啥意思呢？

这表示的是如果用户在浏览器中输入的使 www.cxuanblog.edu 这个域名，其实输入的是 www.cxuanblog.com 这个域名，如果你打算把博客搬家后，你输入的旧域名其实会直接跳转到新域名的网页下。

CNAME 还有一种普遍的做法就是把它作为**公共域名**进行访问。

### 反向 DNS 查询

我们上面一直讨论的是 DNS -> IP 的这种转换方式，这种方式也是 DNS 的精髓所在。但是如果你认真看了图 7 - 1 的话，你会发现还存在一种 IP -> DNS 的转换方式，这种反向的转换也被叫做 **反向 DNS 查询**。他们之间的关系很像 ARP 和 RARP 。

反向 DNS 查询向 DNS 服务器查询 *PTR（Pointer Record）记录*，如果服务器没有 PTR 记录，则无法解析反向查找这个过程。PTR 也是一种 RR 资源记录，见表 7 - 1。

PTR 记录会存储 IP 地址，反向查询时，PTR 中存储的 IP 地址会颠倒过来，并附上 *.in-addr.arpa* 字段，比如如果域的 IP 地址为 192.137.8.22，那么反向查询时，PTR 记录就是 22.8.137.192.in-addr.arpa 。

反向 DNS 查询通常用于电子邮件协议中，电子邮件服务器会检查电子邮箱中的电子邮件消息是否来自真实有效的服务器，垃圾邮件发送者经常使用被劫持机器的，这些邮件过来后就不会有 PTR 记录。电子邮件服务器会拒绝不支持反向查找的服务器或者不太合法的服务器邮件。

### SOA 记录 

如果是权威 DNS 服务器的响应的话，会显示记录存储有关区域的重要信息，这种信息就是 `SOA` 记录。所有的 DNS 区域都需要一个 SOA 记录才能符合 IETF 标准。 SOA 记录对于区域传输也很重要。

SOA 记录除具有 DNS 解析器响应的字段外，还具有一些额外的字段，如下

![image-20220522223207185](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223207185.png)

<div align = "center">图 8-17</div>

具体字段含义

* `PNAME`：即 Primary Name Server，这是区域的主要名称服务器的名称。
* `RNAME`：即 Responsible authority's mailbox，RNAME 代表管理员的电子邮件地址，@ 用 . 来表示，也就是说 admin.example.com 等同于 admin@example.com。
* `序列号`： 即 Serial Number ，区域序列号是该区域的唯一标识符。
* `刷新间隔`：即 Refresh Interval，在请求主服务器提供 SOA 记录以查看其是否已更新之前，辅助服务器应等待的时间（以秒为单位）。
* `重试间隔`：即 Retry Interval ，服务器应等待无响应的主要名称服务器再次请求更新的时间。
* `过期限制`：即 Expire limit ，如果辅助服务器在这段时间内没有收到主服务器的响应，则应停止响应对该区域的查询。

上面提到了主要名称服务器和辅助名称服务器，他们之间的关系如下。

![image-20220522223219531](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223219531.png)

<div align = "center">图 8-18</div>

这块我们主要解释了 RR 类型为 A（IPv4） 和 SOA 的记录，除此之外还有很多类型，这篇文章就不再详细介绍了，读者朋友们可以阅读 《TCP/IP 卷一 协议》和 cloudflare 的官网 https://www.cloudflare.com/learning/dns/dns-records/ 查阅，值得一提的是，cloudflare 是一个学习网络协议非常好的网站。

### 区域传输和 DNS NOTIFY

区域传输通常指一块区域内 DNS 服务器中的 RR 资源更新，这样做的目的是为了保证多台服务器保证内容同步。如果区域中一台服务器失效了，那么其他服务器可以临时顶上，充当临时 DNS 服务器的角色。区域传输通常在*轮询（polling）*后开启，在轮询中，从服务器会周期性的检查主服务器，查看区域是否已经更新，区域传输需要开启。

一旦启动区域传输，就会存在两种传输方式：

1. 全量传输：即传输整个区域的消息，全量传输会传输整个区域（使用 AXFR）的消息。
2. 增量传输：增量传输就是传输一部分消息，增量传输使用（使用 DNS IXFR）的消息。

但是使用轮询这种方式有一些弊端，因为从服务器会定期检查主服务器上内容是否更新，这是一种资源浪费，因为绝大多数情况下都是一次无效检查，所以为了改善这种情况，DNS 设计了 *DNS NOTIFY* 机制，DNS NOTIFY 允许修改区域内容后主服务器通知从服务器内容需要更新，应该启动区域传输。

## DNS 网络排查工具

DNS 常用的排查工具有两种，一种是 *nslookup*，这是一般书籍中推荐使用的排查工具，下面我们先来介绍一下这个工具的使用，一会儿我们再来介绍另外一种工具。

### nslookup

nslookup 是一款用来解决 DNS 相关问题排查的工具。

它主要分为两种模式，一种是**交互模式**，一种是**非交互模式**。交互模式就是一问一答式的，而非交互模式就是一次执行的。

比如你要使用交互式，就直接在命令行中输入 nslookup。

![image-20220522223231318](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223231318.png)

<div align = "center">图 8-19</div>

这样就会开始一个 nslookup 的命令提示符，然后你再输入想要查询的域名即可，如下所示：

![image-20220522223242997](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223242997.png)

<div align = "center">图 8-20</div>

非交互式就是直接输入 nslookup 你想要查询的内容即可，比如我们还以 baidu 为例子。

![image-20220522223257525](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223257525.png)

<div align = "center">图 8-21</div>

其实查询出来的内容是一样的，使用方式其实也大相径庭。

nslookup 一般用于查询下面这些常见的场景：

1. nslookup 能够查询主机的 IP 地址；
2. nslookup 能够查询 IP 地址的域名；
3. nslookup 能够查询域名的邮件服务器。

可以通过 nslookup -querytype 查询域名的邮件服务器，如下

![image-20220522223308813](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223308813.png)

<div align = "center">图 8-22</div>

会分为两种查询结果，一种是 *Non-authoritative answer*，这表明我们想查询的这个网址是从本地 DNS cache 也就是 DNS 缓存中查询出来的，而不是从本地 DNS 经过 DNS 查询后得到的真实域名。

还有一种就是 *Authoritative answers*，这种就是本地 DNS 经过 DNS 查询后得到的真实域名。

上图还显示了 netease.com 邮件服务器的一些参数，origin 表示源地址，mail addr 表示邮件服务器的地址，serial 表示序列号，refresh 表示刷新间隔，retry 表示重试间隔，expire 表示过期时间， minumum 表示最大长度。

### dig

我们的电脑上有多个网络连接，每个网络连接会有不同的 DNS ，而且 DNS 也分为主 DNS 和备用 DNS，nslookup 会默认使用主 DNS 连接，如果你的主 DNS 没有配置，使用可能会存在下面这种情况。

![image-20220522223318456](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223318456.png)

<div align = "center">图 8-23</div>

与 nslookup 不同的使，dig 也是一款 DNS 网络排查工具，它会从你的网络连接中选取一块可用的连接进行解析和使用，不过 windows 10 下默认不支持 dig 命令工具的使用，mac 倒是支持。

下面是 mac 下的 dig 命令。

![image-20220522223331003](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223331003.png)

<div align = "center">图 8-24</div>

不过，贴心的我给你整理出来了 windows10 下 dig 的安装和配置使用 （https://www.csdn.net/tags/Mtjacg0sMjU1ODQtYmxvZwO0O0OO0O0O.html）

安装完成后，就可以在 windows 10 下使用 dig 了。

![image-20220522223341971](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223341971.png)

<div align = "center">图 8-25</div>

下面我们就来介绍一下 dig 这款工具都用哪些用法以及各个参数的含义，我们以 *dig baidu.com* 来进行说明

![image-20220522223351305](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223351305.png)

<div align = "center">图 8-26</div>

如上图所示，最上面的 

; <<>> DiG 9.16.23 <<>> www.baidu.com  表示  dig 版本和要查询的域信息。

;; global options: +cmd 	表示全局选项，dig 可以查询多个域信息，这里显示应用于所有查询的选项，默认是 +cmd。

;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63799  这行表示头信息，其中操作码 QUERY 表示查询，IQUERY 表示反查询，STATUS 表示监测状态等。

NOERROR 表示这个请求已正常解决，id 是一个随机数字，它用于将请求和响应绑定在一起。

;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0  这一行都是一些标志位，其中

* qr = query , rd = recursion desired ，ra = recursion avaliable 这里其实 DNS 和我们玩了一个文字游戏，因为 rd 翻译过来就是需要递归，这个没什么好说的，默认就是使用递归查询；而 ra 翻译过来是递归可用，这个需要思考下，递归可用我是用还是不用呢？当然你可以用也可以不用，如果你不使用递归的话，那么 DNS 查询方式就是迭代查询。
* QUERY 表示查询数量，ANSWER 表示结果数量
* AUTHORITY 表示来自权威域名服务器的结果数量，为 0 说明是从本地 DNS 中返回的，因为没有权威服务器的返回信息。
* ADDITIONAL 表示附加信息，当其值大于 1 时才会看到额外信息。

**下面是问题区域**

;; QUESTION SECTION:
;www.baidu.com.                 IN      A

依次是正在查询的域名，IN 我们上面提到了它表示互联网查询，A 表示域名映射到 IPv4 地址。

**下面是答案部分**

;; ANSWER SECTION:
www.baidu.com.          183     IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       57      IN      A       220.181.38.150
www.a.shifen.com.       57      IN      A       220.181.38.149

中间的数字表示 TTL ，即可以缓存记录的时间间隔。

最后是统计部分，这块没什么好说的了。

除此之外，dig 还有一些其他查询方式。

### -x 进行反向 DNS 查询

我们知道，DNS 可以把域名转换为 IP ，同时也可以把 IP 转换成对应的域名，其中 -x 就是进行反向 DNS 查询，如下所示：

![image-20220522223402689](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223402689.png)

<div align = "center">图 8-27</div>

可以看到 QUESTION SECTION 和 ANSWER SECTION 中都是 PTR，这表示反向 DNS 查询，后面的域名显示了这是一个 google 的  DNS。反向 DNS 查询中，IP 地址要加上 *in-addr.arpa*。

同样的，我们还可以在查询的时候加上 in-addr.arpa，其结果是一样的。

![image-20220522223415784](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223415784.png)

<div align = "center">图 8-28</div>

我们通常喜欢使用 -x，因为这会减少输入的工作量。

### +noall +answer

这告诉 dig 只打印 DNS 响应中的*ANSWER*部分内容，如下所示

![image-20220522223426856](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223426856.png)

<div align = "center">图 8-29</div>

### +short

dig +short  就像是 dig +noall +answer 的阉割版，它只显示很少的内容。

![image-20220522223436890](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223436890.png)

<div align = "center">图 8-30</div>

### +trace

dig +trace 能够模仿 DNS 解析器在查找域名时的做法 ，即它会从根服务器开始查询，一直到权威 DNS 服务器。相当于链路追踪的一个作用。

![image-20220522223459127](https://picturesforarticle.oss-cn-beijing.aliyuncs.com/img/image-20220522223459127.png)

<div align = "center">图 8-31</div>

除了我们上面介绍的 nslookup 和 dig 之外，还有其他 DNS 检测工具，比如 dog 、drill ，都是很好用的 DNS 网络排查工具，大家可以查阅相关资料进行使用，这里我就不再进行详细的介绍了。

## DNS 安全

几乎所有的网络请求都会经过 DNS 查询，而且 DNS 和许多其他的 Internet 协议一样，系统设计时并未考虑到安全性，并且存在一些设计限制，这为 DNS 攻击创造了机会。

DNS 攻击主要有下面这几种方式：

* 第一种是 Dos 攻击，这种攻击的主要形式是使重要的 DNS 服务器比如 TLD 服务器或者根域名服务器过载，从而无法响应权威服务器的请求，使 DNS 查询不起作用。
* 第二种攻击形式是 DNS 欺骗，通过改变 DNS 资源内容，比如伪装一个官方的 DNS 服务器，回复假的资源记录，从而导致主机在尝试与另一台机器连接时，连接至错误的 IP 地址。
* 第三种攻击形式是 DNS 隧道，这种攻击使用其他网络协议通过 DNS 查询和响应建立隧道。攻击者可以使用 SSH、TCP 或者 HTTP 将恶意软件或者被盗信息传递到 DNS 查询中，这种方式使防火墙无法检测到，从而形成 DNS 攻击。
* 第四种攻击形式是 DNS 劫持，在 DNS 劫持中，攻击者将查询重定向到其他域名服务器。这可以通过恶意软件或未经授权的 DNS 服务器修改来完成。尽管结果类似于 DNS 欺骗，但这是完全不同的攻击，因为它的目标是名称服务器上网站的 DNS 记录，而不是解析程序的缓存。
* 第五章攻击形式是 DDoS 攻击，也叫做分布式拒绝服务带宽洪泛攻击，这种攻击形式相当于是 Dos 攻击的升级版

>那么该如何防御 DNS 攻击呢？

防御 DNS 威胁的最广为人知的方法之一就是采用 DNSSEC 协议。

### DNSSEC

DNSSEC 又叫做 DNS 安全扩展，DNSSEC 通过对数据进行数字签名来保护其有效性，从而防止受到攻击。它是由 IETF 提供的一系列 DNS 安全认证的机制。DNSSEC 不会对数据进行加密，它只会验证你所访问的站点地址是否有效。

### DNS 防火墙

有一些攻击是针对服务器进行的，这就需要 DNS 防火墙的登场了，DNS 防火墙是一种可以为 DNS 服务器提供许多安全和性能服务的工具。DNS 防火墙位于用户的 DNS 解析器和他们尝试访问的网站或服务的权威名称服务器之间。防火墙提供限速访问，以关闭试图淹没服务器的攻击者。如果服务器确实由于攻击或任何其他原因而导致停机，则 DNS 防火墙可以通过提供来自缓存的 DNS 响应来使操作员的站点或服务正常运行。

除了上述两种防御手段外，本身 DNS 区域的运营商就会采取进步一措施保护 DNS 服务器，比如配置 DNS 基础架构，来防止 DDoS 攻击。

## 总结

这篇文章我用较多的字数为你介绍了 DNS 的基本概述，DNS 的工作机制，DNS 的查询方式，DNS 的缓存机制，我们还通过 WireShark 抓包带你认识了一下 DNS 的报文，最后我为你介绍了 DNS 的攻击手段和防御方式。

这是一篇入门 DNS 较全的文章，花了我一周多的时间来写这篇文章，这篇文章了解清楚后，基本上 DNS 的大部分问题你应该都能够回答，面试我估计也稳了。

![image-20210717083948590](https://tva1.sinaimg.cn/large/008i3skNly1gsjnhb9f5xj319s0tsn4g.jpg)

![image-20210717084050334](https://tva1.sinaimg.cn/large/008i3skNly1gsjnidv1r3j315s0fs40g.jpg)








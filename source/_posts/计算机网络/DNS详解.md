---
title: DNS详解
categories: 计算机网络
tags: [计算机网络, DNS, 应用层协议]
copyright_author: 崔帆
copyright_author_href: https://xxxxxx.com
copyright_url: https://xxxxxx.com
top_img: /images/dns.webp
cover: /images/dns.webp
copyright_info: 此文章版权归崔帆所有，如有转载，请注明来自原作者
---

写这篇文章的初衷是这样的：我常在各种资料、面经中提及DNS的一些概念，但大多都比较宽泛。比如`DNS劫持`、`DNS实现负载均衡`、`HTTPDNS`等，但对于这些东西是什么，有什么应用场景。却缺少一个系统性的学习、归纳总结过程。本文的目的便是在此，旨在梳理一下DNS相关内容。

## 1. 什么是DNS

DNS即域名系统，全称是`Domain Name System`。在网络请求的过程中，只要输入一个URL，浏览器就会向这个URL的主机名所对应的服务器发送请求来得到主机的IP。对于浏览器来说，**DNS的作用就是将主机名转换为IP地址**。引用《计算机网络：自顶向下方法》中的概念：

DNS是：

> 0.  一个由分层的DNS服务器实现的**分布式数据库**
> 0.  一个使得主机能够查询分布式数据库的**应用层协议**

也就是说，DNS是一个分布式数据库，由分布在全球各地的很多台DNS服务器组成，每台服务器上都保存了一些数据，我们向其中的某些服务器发送一个请求，包含我们要查询的主机名，他就会向我们返回这个主机名对应的IP地址。

所谓**分布式**就是指世界上没有一台DNS服务器拥有互联网上所有主机的IP地址到域名的映射，每台DNS服务器只负责一部分。

**分层**是指域名系统为了保持唯一性，对域名和域名系统的服务器进行了层次结构的划分。
![04-6.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c724e3c9cde1425a9b620ac7887a46ae~tplv-k3u1fbpfcp-watermark.image?)
## 2. DNS的由来

互联网中是使用IP地址表示主机和路由器的，但是**IP地址不方便记忆**，不便于人类使用。人们一般倾向于使用一些有意义的字符串来表示互联网上的设备。例如[www.baidu.com](www.baidu.com)，因此就存在"字符串到IP地址"转换的必要性。

其实在早期，DNS的名字解析方案还不是分层的，当时有一种叫做`ARPANET`的名字解析解决方案：在一个集中的维护站中维护着一张主机名-IP地址的映射文件(Hsots.txt)，然后每台主机定时从维护站中取出文件，这时候主机名还不是分层的(一个平面)。

`ARPANET`存在的主要问题是当网络中主机数量很大的时候，没有层次的主机名很难分配，文件的管理、发布、查找也很麻烦。

因此才有了后来DNS的**分层的、基于域**的命名机制，它使用若干**分布式的数据库**完成名字到IP地址的转换，是一种定义了主机如何查询这个分布式数据库方式的应用层协议。以UDP作为传输层协议(TCP的开销太大)，运行在53号端口上。

![04-7.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f43b59d60724fd68620e8edbf2f6220~tplv-k3u1fbpfcp-watermark.image?)

## 2. DNS域名层次结构

DNS是一种分层结构，在整个互联网中组成一个树状系统，顶层是系统的根域名，下层为TLD(顶级域)以及二级域名，**叶子就构成了所谓的FQDN**（Fully Qualified Domain Names），根域名通常使用"."来表示，其实际上也是域名组成，全世界目前有13组域名根节点，由少数几个国家进行管理，而国内仅有几台根节点镜像。

DNS采用层级树状结构来进行命名，Internet被分为几百个顶级域；顶级域包含：

-   通用的(generic)

    .com; .edu ; .gov ; .int ; .mil ; .net ; .org ; .firm ; .hsop ; .web ; .arts ; .rec ;

-   国家的(countries)

    .cn ; .us ; .nl ; .jp

每个(子)域下面又可以划分为若干个子域，树叶才是主机名。

域名：从本域往上，直到树根，中间使用"."间隔不同的级别，如`blog.51cto.com.`

域的域名：用来表示一个域。主机的域名：用来表示一个域上的一台主机。

一个域会管理其下的子域。比如.jp 划分为ac.jp , co.jp；.cn划分为edu.cn ，com.cn等。因此，创建一个新的域，必须经过它所属域的同意。

域遵从组织界限，划分是根据逻辑的，而不是物理网络。

![04-1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3343ba7511145e1b504850a2809aa6d~tplv-k3u1fbpfcp-watermark.image?)
## 4. DNS名字服务器

根据域名的层次结构，管理不同层次域名的服务器，可以划分为`根域名服务器`，`顶级域名服务器`，`权威域名服务器`。

通常名字服务器不止一个。一个名字服务器通常会有问题：

> -   可靠性问题：单点故障
> -   扩展性问题：通信容量
> -   维护问题：远距离的集中式数据库...

然后在这些域中，存在着**区域**的概念。区域的划分由区域管理者自己决定。可以把DNS域名层次结构划分为互不相交的区域，每个区域都是树的一部分。**每个区域都有自己的权威名字服务器**，它维护者所管辖区域的所有权威信息。权威名字服务器允许被放置在区域之外，以保障可靠性。
![04-3.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10b2c1e7957542918e2748f3c54fec09~tplv-k3u1fbpfcp-watermark.image?)

**权威域名服务器**：组织机构的DNS服务器；提供组织机构服务器(Web/Email)可访问的主机和IP之间的映射。组织机构可以选择实现自己维护或者某个服务提供商来维护。

它是经过上一级授权对域名进行解析的服务器，同时它可以把解析授权转授给其他人，如COM顶级服务器可以授权xxorg.com这个域名的的权威服务器为NS.ABC.COM，同时NS.ABC.COM还可以把授权转授给NS.DDD.COM，这样NS.DDD.COM就成了ABC.COM实际上的权威服务器了。平时我们解析域名的结果都源自权威DNS。

**顶级域名服务器**(TLD)：负责顶级域名（如com, org, net, edu和gov）和所有国家级的顶级域名（如cn, uk, fr, ca, jp ）Network solutions 公司维护com TLD服务器 ；Educause公司维护edu TLD服务器，它提供了它的下一级，也就是权威域名服务器的IP地址。

**本地DNS域名服务器**：并不严格属于层次结构。每个ISP(居民区的ISP、大学、公司)都有一个本地DNS服务器，当主机发起一个DNS查询的时候，查村会被发往本地DNS服务器。**本地DNS服务器起着代理的作用**，没有缓存的时候**将请求转发到DNS服务器的层次结构中**。

## 5. DNS资源记录

前面提到过DNS是一个定义了如何从分层的分布式数据库中读取数据的应用层协议。那么读取的数据是什么呢？实际上是一条条资源记录(RR)，由区域(权威)名字服务器维护，作用是维护域名-IP地址的映射关系。

RR格式：(Name，Value，Type，TTL)

-   Domain_name：域名

-   Ttl：time to leave：生存时间(权威，缓冲记录)，决定了该资源记录应当从缓存中删除的时间

-   Class：对于Internet，是IN

-   Value：可以是数字、域名或者ASCll串

-   Type：

    0.  **Type = A**：Name为主机，Value为IP地址；该资源记录提供了标准的主机名到IP地址的映射
    0.  **Type = NS**：Name为域名，Value为该域名的权威名字服务器的域名。
    0.  **Type = CNAME**：Name为规范名字的别名，Value为规范名字。主机别名主要是为了通过给一些复杂的主机名提供 一个便于记忆的简单的别名。
    0.  **Type = MX**：Name为规范名字的别名，Value为Name对应的邮件服务器的名字
    0.  **Type = TXT**：为某个主机或域名设置说明。

## 6. DNS查询流程

以下图片都假设没有缓存。

**递归查询**：
![04-4.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f28a093a1f14fb88765a683a6718278~tplv-k3u1fbpfcp-watermark.image?)

特点：每个查询请求，中间的DNS服务器都要为当前请求保存状态，并且最后都要返回查询结果往外层递归，根服务器负担太重。

解决办法：迭代查询

**迭代查询**：
![04-5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edf5bd81d07247aca3209064038f82bf~tplv-k3u1fbpfcp-watermark.image?)

特点：中间的DNS服务器返回的不是查询结果，而是下一个NS的地址，最后由权威名字服务器给出解析结果。

大致流程如下：

0.  首先，主机`cis.poly.edu`向本地DNS服务器发送一个DNS查询报文，其中包含了期待被转换的主机名`gaia.cs.umass.edu.`
0.  本地DNS服务器将该报文转发到根DNS服务器
0.  根DNS服务器注意到`edu`，便向本地DNS服务器返回`edu`对应的对应的顶级域DNS服务器(TLD)的IP地址列表。
0.  本地DNS服务器向其中一台(一般是靠前的)TLD服务器发送查询报文
0.  TLD服务器注意到`umass.edu`前缀，便向本地DNS服务器返回权威DNS服务器的IP地址
0.  本地DNS服务器再向其中一台DNS权威服务器发送查询报文
0.  最终，该权威服务器返回了`gaia.cs.umass.edu`的IP地址
0.  本地服务器将该IP地址发送给主机`cis.poly.edu`，主机就可以用该IP发送请求了

看到这里，大家可能会有个疑问，TLD 一定知道权威 DNS 服务器的 IP 地址吗？

这就需要阅读下面的内容了，DNS如何新增一个域。

## 7. DNS维护：新增域

这需要在上级域的名字服务器中添加两条记录，指向这个新增的子域的域名和域名服务器的地址。同时在新增的子域上运行名字服务器，负责本域的域名解析。

举个例子：在com域中增加`Simon`

0.  到注册等级机构注册域名`simon.com`

    -   需要向该机构提供你权威DNS服务器(基本的、辅助的)的名字和IP地址
    -   然后登记机构在`com`TLD服务器中添加两条资源记录：`(simon.com, dns.simon.com, NS)`和`(dns.simon.com, 211.211.211.1, A)`

0.  在`simon.com`的权威名字服务器中要确保有：

    -   用于web服务器的`www.simon.com`的类型为A的资源记录
    -   用于邮件服务器的`mail.simon.com`的类型为MX的资源记录

现在可以回答上一个问题**TLD 一定知道权威 DNS 服务器的 IP 地址吗**了，新增一个域，一般都是需要经过TLD注册授权的。所以通常情况都是知道的。但有时 TLD 只是知道中间的某个 DNS 服务器，再由这个中间 DNS 服务器去找到权威 DNS 服务器。这种时候，整个查询过程就需要更多的 DNS 报文。

## 8. DNS性能优化

**使用DNS缓存**

为了更快地拿到IP地址，提高性能，DNS服务器一旦学习到了一个映射，就会将该映射缓存起来。根服务器通常都在本地服务器中缓存着，使得根服务器不用经常被访问。缓存的时间由TTL决定。

**DNS实现全局负载均衡**(GSLB)

目的是通过DNS实现互联网上不同地域的服务器间的流量调配，保证用户的请求能被距离用户最近或者服务质量更好的服务器来处理。

现在一般的大型网站都是使用多台服务器提供服务，因此**一个域名可能会对应多个服务器地址**。当用户发起网站域名的DNS请求的时候，DNS服务器会返回域名对应服务器的IP地址列表。但是在每个回答中，会使用一定的算法**轮询这些IP地址**，用户一般会选择前面的地址发送请求，以此来将用户均匀地分配到不同的服务器上去。

![04-8.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b26fb0a013fd4c039e751aba614a584b~tplv-k3u1fbpfcp-watermark.image?)

**智能DNS**

它是GSLB的一种应用，能通过判断服务器的负载，包括CPU占用、带宽占用等数据，决定服务器的可用性，同时判断用户与服务器的链路状况，对各项指标进行综合判断来决定由哪个地点的服务器来提供服务，实现异地服务器群服务质量的保证。


![04-9.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a89fc753576f4e22bad1d6729c00f87c~tplv-k3u1fbpfcp-watermark.image?)

## 9. DNS解析问题

**运营商劫持**

DNS劫持 就是通过劫持了 DNS 服务器，通过某些手段取得某域名的解析记录控制权，进而修改此域名的解析结果，导致对该域名的访问由原 IP 地址转入到修改后的指定 IP，其结果就是对特定的网址不能访问或访问的是假网址，从而实现窃取资料或者破坏原有正常服务的目的。

![04-10.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8374f3f0240649f6844d5c0527ca9062~tplv-k3u1fbpfcp-watermark.image?)
**本地DNS服务器缓存解析结果**

如果TTL时间内目标域名IP发生了变化，LocalDNS没有及时更新，就会导致用户访问不到服务器


![04-11.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed047cef1b1a4da2929910eda2e07ee8~tplv-k3u1fbpfcp-watermark.image?)

**转发解析**

用移动设备访问`baidu.com`的时候，会先到当前运营商的DNS服务器查，然后运营商的DNS服务器再去百度权威服务器查，最终把正确IP地址返回。

上面是正常的情况，但是如果当前运营商(比如联通)的LocalDNS不访问百度权威DNS服务器，而是直接访问了其它运营商(比如电信)的LocalDNS服务器，有些小的运营商就是通过这样做来降低成本。如果电信的LocalDNS对非自家ip的访问限了速那么很明显会影响你的DNS解析时间。


![04-12.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f2f430c692a4d08a7db8205fd55fba6~tplv-k3u1fbpfcp-watermark.image?)

## 10. HTTPDNS

HTTPDNS使用HTTP与DNS服务器进行交互，代替传统的基于UDP的DNS协议，域名解析请求直接发送到HTTPDNS服务端，从而绕过运营商的Local DNS

![04-13.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e3fa9d825c546a2a751511c4115f366~tplv-k3u1fbpfcp-watermark.image?)
![04-14.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/406585ee629140aca8311e1f2c675d66~tplv-k3u1fbpfcp-watermark.image?)
HTTPDNS的特性：

0.  **防止域名被劫持**

由于 HTTPDNS 是通过 IP 直接请求 HTTP 获取服务器 A 记录地址，不存在向本地运营商询问 domain 解析过程，所以从根本避免了劫持问题。

2.  **精准调度**

HTTPDNS能够直接获取到用户的IP地址，从而实现精确定位与导流

3.  **用户连接失败率下降**

通过算法降低以往失败率过高的服务器排序，通过**时间近期访问过的数据**提高服务器排序，通过历史访问成功记录提高服务器排序。

**HTTPS IP Content**

发送HTTPS请求首先要进行SSL/TLS握手，大致过程如下：

> 0.  客户端发起握手请求，携带随机数，支持的算法列表、HTTP协议及版本号等信息。
> 0.  服务端收到请求，选择合适的算法，下发公钥证书和随机数。
> 0.  客户端对服务端公钥证书进行校验通过后，发送随机数信息，用公钥加密该信息。
> 0.  服务端用私钥解密拿到随机数。
> 0.  双方根据选择的加密算法合成session ticket，用于后续数据传输的加密密钥。

上述TLS连接过程和HTTPDNS有关的是证书的校验，验证证书的过程有两个要点：

-   客户端使用本地保存的根证书解开证书链，确认服务端下发的证书是由可信任机构颁发的
-   客户端检查证书的domain域和扩展域，看是否包含本次请求的host

有一点检查不通过，就是不可信任，便会断开当前连接。更多关于HTTPS的内容，请查看我的其他文章：[HTTPS详解]()

当客户端使用HTTPDNS解析域名的时候，**请求URL中的host会被替换成HTTPDNS解析出来的IP**，在验证证书的第二步会出现domain不匹配的情况，导致TLS握手失败。解决方案可参考[HTTPS与httpdns一起用的业务场景解决方案](https://blog.csdn.net/huwei2003/article/details/53507078)

说了这么多DNS有关的内容，最后来问一个最简单的问题，**主机是如何知道DNS服务器的地址呢**？

答：通过DHCP动态获得，或者手动配置。(**装系统的时候内置好了，可以修改**)

DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）：通常被应用在大型的局域网络环境中，主要作用是集中的管理、分配IP地址，使网络环境中的主机动态的获得IP地址、Gateway地址、DNS服务器地址等信息，并能够提升地址的使用率。
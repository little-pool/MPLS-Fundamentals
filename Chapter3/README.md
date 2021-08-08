# MPLS Fundamentals - Chapter 3

## Chapter3 - 标签报文的转发

### 概述

#### 这一章能学到啥

* 了解一个标签报文时如何在网络中转发的
* 看看MPLS都保留了哪些标签，他们都是干嘛用的
* 在MPLS网络中，了解MTU的重要性
* 了解一个标签报文TTL超时后会发生啥
* 了解一个标签报文是如何被分片的



<p>
  上一章 MPLS Architecture 我们了解了MPLS是啥以及标签是如何使用的，这章我们会着重讲解MPLS的数据平面。在网络中标签报文的转发和IP报文有很大区别。区别不仅仅是查表方式的不同（IP转发查路由表，标签转发查LFIB），对报文的操作也存在很大区别。
</p>

<p>
  阅读的时候要注意MPLS保留标签的特殊作用，应为它们会贯穿整本书。
</p>



### 标签报文的转发机制

<p>
  这小结我们关注标签报文如何在MPLS网络中进行转发，其与IP转发有何区别，如何做的负载均衡以及遇到未知标签时LSR的处理方式。
</p>



#### 标签操作

<p>
  如图所示，标签操作一共就有这3种。
</p>

![Figure 3-1. Operations on Labels](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:1587051974/files/1587051974_ch03lev1sec1_image01.gif)

<p>
  LSR收到标签报文后，根据顶层标签去查找LFIB从而判断如何转发。在转发之前LSR会决定去执行swap, push或者pop操作。swap就是把顶层标签替换一下。push就是顶层标签被替换的同事，又在上面压了若干个标签。pop就是弹出顶层标签。
</p>

**Note**

<p>
  LSR的行为，查看顶层标签的前20个bits，然后再LFIB中和本地标签进行匹配，从而决定转发行为。
</p>



#### IP查表 vs 标签查表

<p>
  当一个路由器收到IP报文时，它会执行IP查找行为。在Cisco IOS中路由器会去看CEF表。当路由器收到标签报文时，它会去查LFIB。路由器收可以在接收数据包的二层头部的协议字段中知道里面是一个IP包还是标签包。路由器收到的报文格式与发出的报文格式之间没有必然联系，决定权在于CEF和LFIB中的转发决策。
</p>

<p>
  下图展示了通过CEF与通过LFIB转发的区别
</p>

![Figure 3-2. CEF or LFIB Lookup](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:1587051974/files/1587051974_ch03lev1sec1_image02.gif)

<p>
  如果从LSR进来一个IP报文，转发出去时压上了标签，这就是IP-to-label的情况。如果进来一个标签报文，它可以拿掉标签发出IP报文，也可以继续以标签报文的格式转发。第一种情况为label-to-IP，第二种情况为label-to-label。
</p>

**Note**

<p>
  CEF是啥以及它是如何与MPLS互动的，我们会在第6章介绍。
</p>

<p>
  下图展示了IP-to-label的情况，CEF表中展示了IP报文的处理方式。
</p>

`lactometer#show ip cef 10.200.254.4
10.200.254.4/32, version 44, epoch 0, cached adjacency 10.200.200.2
0 packets, 0 bytes
  tag information set, all rewrites owned
    local tag: 20
    fast tag rewrite with Et0/0/0, 10.200.200.2, tags imposed {18}
  via 10.200.200.2, Ethernet0/0/0, 0 dependencies
    next hop 10.200.200.2, Ethernet0/0/0
    valid cached adjacency
    tag rewrite with Et0/0/0, 10.200.200.2, tags imposed {18}`











#### 标签报文的负载均衡

#### 未知标签

### 保留标签

### 非保留标签

### 标签报文中TTL的行为

### MPLS中的MTU

### MPLS报文的分片机制

### 路径MTU的发现机制
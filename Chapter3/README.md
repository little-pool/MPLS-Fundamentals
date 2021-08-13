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

```
lactometer#show ip cef 10.200.254.4
10.200.254.4/32, version 44, epoch 0, cached adjacency 10.200.200.2
0 packets, 0 bytes
  tag information set, all rewrites owned
    local tag: 20
    fast tag rewrite with Et0/0/0, 10.200.200.2, tags imposed {18}
  via 10.200.200.2, Ethernet0/0/0, 0 dependencies
    next hop 10.200.200.2, Ethernet0/0/0
    valid cached adjacency
    tag rewrite with Et0/0/0, 10.200.200.2, tags imposed {18}
```

<p>
  在LSR上，去往10.200.254.4/32的报文会被压上标签18后，从Ethernet0/0/0转发出去。数据包的下一跳是10.200.200.2。这就是在LSR上IP-to-label的转发过程。在Cisco IOS中，CEF交换是所有IP交换方式中唯一可以处理标签报文的模式。其他模式比如快速转发就不能处理标签包，因为快速转发模式的缓存中无法存储标签信息（太古老，设计时就没考虑到）。所以运行MPLS的路由器必须开启CEF转发。
</p>

<p>
  下图展示了LFIB的样子
</p>

```
lactometer# show mpls forwarding-table
Local  Outgoing     Prefix            Bytes tag  Outgoing    Next Hop
tag    tag or VC    or Tunnel Id      switched   interface
16     Untagged     10.1.1.0/24       0          Et0/0/0     10.200.200.2
17     16           10.200.202.0/24   0          Et0/0/0     10.200.200.2
18     Pop tag      10.200.203.0/24   0          Et0/0/0     10.200.200.2
19     Pop tag      10.200.201.0/24   0          Et0/0/0     10.200.200.2
20     18           10.200.254.4/32   0          Et0/0/0     10.200.200.2
21     Pop tag      10.200.254.2/32   0          Et0/0/0     10.200.200.2
22     17           10.200.254.3/32   0          Et0/0/0     10.200.200.2
24     Untagged     l2ckt(100)        4771050    Fa9/0/0     point2point
```

<p>
  本地标签是LSR自己生成并通告给其他LSR的标签列表，LSR收到标签报文时，报文的栈顶标签必须存在于本地标签列表中。比如LSR收到一个栈顶为22的标签报文，那么它会被swap成17然后从Ethernet0/0/0转发出去。
</p>

<p>
  如果LSR收到一个标签报文栈顶为16，那它会移除整个标签栈，然后继续做IP转发，因为Outgoing为Untagged。这就是一个label-to-IP的例子。如果LSR收到一个栈顶为18的标签包，那它会移除栈顶标签，然后以标签或者IP的形式继续转发报文。下面的例子给出了swap和pop的例子。
</p>

<p>
  下图是一个push操作的例子，入标签23被替换成了20，同时在20上面我们又压上了16。
</p>

```
lactometer#show mpls forwarding-table 10.200.254.4
Local  Outgoing    Prefix            Bytes tag  Outgoing    Next Hop
tag    tag or VC   or Tunnel Id      switched   interface
23     16      [T] 10.200.254.4/32   0          Tu1         point2point

[T]     Forwarding through a TSP tunnel.
        View additional tagging info with the 'detail' option

lactometer#show mpls forwarding-table 10.200.254.4 detail
Local  Outgoing    Prefix            Bytes tag  Outgoing   Next Hop
tag    tag or VC   or Tunnel Id      switched   interface
23     16          10.200.254.4/32   0          Tu1        point2point

        MAC/Encaps=14/22, MRU=1496, Tag Stack{20 16}, via Et0/0/0
        00604700881D00024A4008008847 0001400000010000
        No output feature configured
```

<p>
  要查看LSR对标签报文的操作，你就要输入 show mpls forwarding-table [network {mask | length}] [detail] 命令。通过上面的图你可以看出来 detail 关键字对输出结果的影响。如果指定了detail，你可以看到整个标签栈的变化。在花括号中从左到右，可以看出第一个标签被替换成了20，然后16压在了20上面。如果不指定detail，你只能看到最上层的16。
</p>

<p>
  我们要特别注意一下路由汇总的情况。当你在LSR上做了路由汇总后，它会针对汇总路由单独生成并通告一个标签，但在LFIB中，这个入标签对应的出标签会显示为”Aggregate“。因为入标签对应着汇总路由，LSR没办法直接做top swap然后转发。当出方向为”Aggregate“时，LSR必须移除标签栈然后针对具体的目标前缀做IP转发。下图展示了MPLS网络中Egress PE里LFIB的一个条目。
</p>

<p>
  如果Egress LSR收到了一个带有栈顶23的标签包，那它会移除标签栈，然后依据目标IP地址执行IP转发。
</p>

```
singularity#show mpls forwarding-tablevrf cust-one

Local  Outgoing     Prefix           Bytes tag  Outgoing   Next Hop
tag    tag or VC    or Tunnel Id     switched   interface
23     Aggregate    10.10.1.0/24[V]  0
```

<p>
  现在你知道一个标签报文是如何通过标签操作来寻找下一跳的课。下面我们来看看CEF中的邻接表是干嘛的。它提供了LSR去往任何一个下一跳所需要的完整二层封装，细节我们会在第6章介绍。
</p>

<p>
  我们看看邻接表长什么样，
</p>

```
lactometer#show adjacency detail
Protocol Interface                 Address
IP       Ethernet0/0/0             10.200.200.2(13)
                                   0 packets, 0 bytes
                                   epoch 0
                                   sourced in sev-epoch 4
                                   Encap length 14
                                   00604700881D00024A4008000800
                                   ARP
TAG      Ethernet0/0/0             10.200.200.2(9)
                                   231 packets, 22062 bytes
                                   epoch 0
                                   sourced in sev-epoch 4
                                   Encap length 14
                                   00604700881D00024A4008008847
                                   ARP
IP       Serial0/1/0               point2point(10)
                                   258 packets, 35612 bytes
                                   epoch 0
                                   sourced in sev-epoch 4
                                   Encap length 4
                                   0F000800
                                   P2P-ADJ
TAG      Serial0/1/0               point2point(5)
                                   0 packets, 0 bytes
                                   epoch 0
                                   sourced in sev-epoch 4
                                   Encap length 4
                                   0F008847
                                   P2P-ADJ
```

<p>
  我们再来回顾一下标签的所有操作。
</p>

* Pop - 移除顶层标签，报文会带着剩下的标签或IP报文继续转发。
* Swap - 顶层标签会被替换。
* Push - 顶层标签会被替换，然后会继续压上一个或多个标签。
* Untagged/No Label - 标签栈会被移除，后面执行IP转发。
* Aggregate - 标签栈会被移除，后面执行IP转发。



#### 标签报文的负载均衡

<p>
  如果一个IP前缀有多个等价路径，那么IOS会如下图所示，负载均衡地转发标签报文。我们可以看到本地标签17和18都对应两个出接口。如果标签报文被负载均衡转发，那么两条路径的出标签既可能相同也可能不同。如果出方向的两条路都对应着同一台设备且两个出接口处于同一个标签域，那出标签就是相同的。但如果下一跳对应多个LSR，那么出标签大概率会不同，因为出标签是每个对应下一跳LSR自己本地分配的。
</p>

```
horizon#show mpls forwarding-table
Local  Outgoing    Prefix            Bytes tag   Outgoing    Next Hop
tag    tag or VC   or Tunnel Id      switched    interface
17     Pop tag     10.200.254.3/32   252         Et1/3       10.200.203.2
       Pop tag     10.200.254.3/32   0           Et1/2       10.200.201.2
18     16          10.200.254.4/32   10431273    Et1/2       10.200.201.2
       16          10.200.254.4/32   238         Et1/3       10.200.203.2
```



<p>
  如果一个IP前缀对应的下一跳既有标签又有IP路径，那默认Cisco IOS不会执行负载均衡转发，它会只走标签路径。这是因为很多情况下，标签报文一旦走了IP路径，它很有可能就到不了目的地了。在IPv4-over-MPLS的情况下，即使报文在中途丢失了标签，它也可以通过IP被转发到目的地。然而在其他的场景下，比如MPLS VPN，或者AToM的环境中，失去标签的报文将会在网络中被丢弃。
</p>

<p>
  我们以MPLS VPN为例，MPLS的负载时IPv4的报文，这IPv4报文中的目标地址属于VPNv4的路由，然而P路由器是没有这些路由的，所以P路由器无法转发这样的IPv4报文。同样在AToM中，MPLS的负载时2层的数据帧，P路由器同样没有这些2层转发信息。这就是为什么当标签和IP路径同时存在时，我们不能执行负载均衡而只能从标签路径转发。通常情况下，只有PE设备才有能力处理MPLS的负载报文，因此数据在P设备之间传输时，LSP绝对不能断。
</p>

<p>
  下图展示了一个由两个标签路径组成的路由，当我们在其中一个出接口上关闭LDP时，我们会发现相应的标签路径也会在LFIB中消失。
</p>

```
horizon#show mpls forwarding-table 10.200.254.4
Local  Outgoing    Prefix            Bytes tag   Outgoing   Next Hop
tag    tag or VC   or Tunnel Id      switched    interface
18     18          10.200.254.4/32   56818       Et1/2      10.200.201.2
       18          10.200.254.4/32   160         Et1/3      10.200.203.2
horizon#conf t
Enter configuration commands, one per line. End with CNTL/Z.
horizon(config)#interface ethernet 1/3
horizon(config-if)#no mpls ip
horizon(config-if)#^Z
horizon#horizon#show mpls forwarding-table 10.200.254.4
Local  Outgoing    Prefix            Bytes tag   Outgoing   Next Hop
tag    tag or VC   or Tunnel Id      switched    interface
18     18          10.200.254.4/32   57270       Et1/2      10.200.201.2
```



#### 未知标签

<p>
  通常情况下，收到标签报文时，LSR一定认识它的顶层标签，因为这个标签是LSR自己生成然后通告出去的。但总会有意外，当LSR收到的标签报文带有它未知的顶层标时，可能存在两种做法：拿掉顶层标签试图转发其负载；直接丢弃；Cisco IOS的做法是直接丢弃，没毛病。LSR如果不认识这个顶层标签，它就不可能知道里面是什么，因为标签中没有协议标识字段，所以你拆开你是指望得到一个surprise么？即使走了个狗屎运，你认识里面的标签或者目标IP，你发出去了，你觉得你的做法对下游设备负责么？他们要是不认识怎么办？不还得丢弃么？所以IOS的做法它指定是没毛病。
</p>



### 保留标签

### 非保留标签

### 标签报文中TTL的行为

### MPLS中的MTU

### MPLS报文的分片机制

### 路径MTU的发现机制
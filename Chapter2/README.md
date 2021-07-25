# MPLS Fundamentals - Chapter 2

## Charpter2 - MPLS架构

### 什么是MPLS标签

<p>
  一个MPLS标签由如下32个bit组成
</p>

![Figure 2-1. Syntax of One MPLS Label](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec1_image01.gif)

<p>
  前20个bits是标签的value，范围是0到2^20-1。前16个数值被保留于特殊用途。第20到22个bits为EXP（Experimental）用于QoS的Marking。
</p>

**Note**

<p>
  之前谁也没考虑到MPLS需要QoS的集成，所以那3个bits当时被命名为Experimental。在第12章我们会讲到它们是如何被使用的。
</p>

<p>
  第23个bit是栈底标识Bos，如果这个标签是栈底标签，Bos=1，否则Bos=0。数据包头部的标签栈是由一个个标签组成的，理论上栈中的标签可以有无限多个，但实际上你很难看到带有超过4个标签的标签栈。
</p>

<p>
  24~31bits是8位的TTL。作用和IP的TTL一致，逐跳减1以防止数据包在路由环路中一直被转发，如果在转发过程中路由器发现TTL=0，那么数据包会被丢弃。
</p>

<p>
  下一小节介绍标签栈和标签栈在数据包中的位置。
</p>



#### 标签栈

<p>
  支持MPLS的路由器想要在MPLS网络中转发数据，那么数据包头中就必须有标签。路由器会把标签压到包头的标签栈中，第一个标签叫做top label，最后一个标签叫bottom label。在他们中间还会有任意多个label存在。如下图所示：
</p>

![Figure 2-2. Label Stack](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec1_image02.gif)

<p>
  注意上图中除了最下面的标签外，其他标签的BoS都是0，只有最下面的栈底标签BoS是1
</p>

<p>
  很多MPLS的应用都在标签栈中使用超过1个标签，2个例子就是MPLS VPN和AToM，它们都使用了2层标签。这些我们会在第7章和第10章中详细介绍。
</p>



#### MPLS数据包编码

<p>
  MPLS标签栈在数据包的什么位置？3层头部前面，2层头部后面。
</p>

<p>
  下图告诉我们标签栈在数据包中的具体位置
</p>

![Figure 2-3. Encapsulation for Labeled Packet](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec1_image03.gif)

<p>
  图中的2层封装可以是思科IOS支持的任意二层协议：PPP，HDLC，以太，等等。假如传输层是IPv4，数据链路层是PPP，那么标签栈就位于PPP包头后，IPv4包头前。因为数据链路层后面有增加了一个新成员，因此它头部中就要有一个新的协议标识来表示2层头部的下一层是MPLS标签帮助设备解码。下图给出了不同二层协议针对MPLS的协议标识。
</p>

| **Layer 2 Encapsulation Type**        | **Layer 2 Protocol Identifier Name** | **Value (hex)** |
| ------------------------------------- | ------------------------------------ | --------------- |
| PPP                                   | PPP Protocol field                   | 0281            |
| Ethernet/802.3 LLC/SNAP encapsulation | Ethertype value                      | 8847            |
| HDLC                                  | Protocol                             | 8847            |
| Frame Relay                           | NLPID (Network Level Protocol ID)    | 80              |

<p>
  省略关于ATM和FrameRelay相关内容。
</p>



### MPLS 和 OSI 参考模型

<p>
  OSI七层模型参考下图：
</p>

![Figure 2-4. OSI Reference Model](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec2_image01.gif)

<p>
  最下面是1层也叫物理层，最上面是7层也叫应用层。物理层关心线缆，机械和光学特性，而数据链路层（2层）关心数据包的格式。举个例子，数据链路层的协议有以太，PPP，HDLC和帧中继，而数据链路信号只能在导线两端的设备间传输，这意味着数据链路层的报头会经常被导线末端的设备替换，所以数据链路层只负责两台设备之间的传输，而网络层（3层）关心的是在端到端传输过程中数据的格式，它是运行于数据链路层之上的。市场份额最大的网络层协议就是IP
</p>

<p>
  MPLS适合做哪一层呢？省略一堆废话。。。因此，它不在OSI模型里面，它是2.5层。
</p>



### 标签交换路由器

<p>
  支持MPLS的路由器我们就叫它LSR（标签交换路由器）。它可以理解数据包中的MPLS标签，可以接受并转发MPLS报文，在网络中一共有3中类型的LSR。
</p>

- Ingress LSR - 接收非标签报文，在栈顶压入标签然后继续转发。
- Egress LSR - 接收标签报文，拿掉标签后继续转发，前两种LSR一般是网络的边界LSR。
- Intermediate LSR - 接收标签报文，执行一个label switching操作，继续转发。

<p>
  LSR最标签报文的3中操作，pop（弹出），push（压入），swap（交换）。
</p>


### 标签交换路径

<p>
  标签交换路径（LSP）就是在MPLS网络中通过标签交换进行数据传递的一条LSR序列。就是说，LSP是数据包在MPLS网络中的传输路径。这条路径的第一个LSR叫做Ingress LSR，最后一个LSR叫做Egress LSR。所有中间的LSR叫做Intermediate LSR。
</p>

<p>
  看下面的图，上面的箭头表示方向，因为LSP是单向的。因此反方向从右到左的路径会是另一条LSP。
</p>

![Figure 2-5. An LSP Through an MPLS Network](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec4_image01.gif)

<p>
  注意哈，这里的Ingress LSR并不一定是第一个给数据包打标签的路由器,数据包可能来的时候就有标签了，比如LSP嵌套，就是多层标签。就像下面图中，一条LSP跨越整个MPLS网络，另一条LSP从第三个LSR开始，在倒数第二个LSR结束。因此 ，当数据包到达第二个LSP的Ingress LSR时，它会被打上第二层标签，此时数据包的标签栈就会有两层标签。顶层标签属于这个短的LSP，底层标签属于长的LSP。
</p>

![Figure 2-6. Nested LSP](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec4_image02.gif)



### 转发等价类 - FEC(Forwarding Equivalence Class)

<p>
  FET是具有相同转发路径或被执行相同转发处理的一组数据流。所有相同FEC的数据具有相同的标签，但具有相同标签的数据包不一定属于同一个FEC,因为它们的EXP不一定相同，转发时会应用不同的策略，即它们属于不同的FEC。Ingress LSR决定了数据包属于哪个FEC，因为数据包的标签是它打上去的，它更有权力做数据分类。下面是一些FEC的实例：
</p>

* 匹配到相同3层目标前缀的数据包。
* 属于相同组的组播流量。
* 具有相同转发优先级或相同DSCP的数据包。
* 从相同Ingress LSR的VC发往Egress LSR的VC的跨越MPLS网络的2层帧。
* 3层目标IP匹配到BGP前缀具有相同下一跳的数据包（相同BGP next hop）。

<p>
  下图是一个有趣的例子，在Ingress LSR上，所有匹配到具有相同下一跳的BGP路由的数据都会属于同一个FEC。这就说明在MPLS骨干网中给数据包压啥标签就取决于BGP下一跳。
</p>

![Figure 2-7. An MPLS Network Running iBGP](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch02lev1sec5_image01.gif)

<p>
  
</p>



<p>
  到达Ingress LSR的数据包会用目标IP在LSR的IP转发表中进行匹配。多个IP地址会被匹配到一条BGP前缀上，多个BGP前缀可能会有相同的BGP next-hop，也就是相同的Egress LSR。因此，所有目标IP匹配到相同BGP next-hop的数据包都会被映射到同一个FEC，从而在Ingress LSR打上相同的标签。
</p>



### 标签分发

<p>
  数据包在Ingress LSR上被压入标签，这个标签一定会属于一个LSP，数据包会借助这条LSP穿越MPLS网络。数据包在MPLS网络中传输时栈顶标签每一跳都会被替换。Ingress LSR会在开始时给数据包压入一层或多层标签，传输过程中，Intermediate LSR会把栈顶的入向标签替换成出向标签然后发往下一跳，最后，Egress LSR把栈顶标签剥掉继续转发报文。
</p>

<p>
  我们来看一个IPv4-over-MPLS网络最简单的例子，低配版本中，所有LSR都跑IPv4的IGP。Ingress LSR收到报文后看一下目标IP，然后匹配一个标签给压上，然后发出去。下一个LSR收到一个标签包，将入标签替换成出标签然后继续转发。最后Egress LSR弹出标签，把里面的IPv4报文从出接口转发出去。为了能让上述流程正常运行，相邻的LSR必须在标签和IGP前缀的映射上达成共识，然后他们才会知道接收报文的入向标签应该被替换成哪一个出向标签。因此你需要一种机制来告诉LSR在转发报文时应该使用什么标签。标签是本地有效的（意思是仅在邻接的路由器之间有意义），它没有任何全局意义（除了SR Label）。所以为了在邻接LSR间就标签机制达成共识，它俩必须可以聊天，因此标签分发协议就诞生了。
</p>

<p>
  你可以使用如下两种方式分发标签：
</p>

* 用现有的IP路由协议分发标签，比如IGP，BGP
* 使用一个单独的标签分发协议









### 标签分发协议 LDP

### 标签转发表 LFIB

### MPLS 载荷

### MPLS 标签空间

### 不同的 MPLS 运行模式


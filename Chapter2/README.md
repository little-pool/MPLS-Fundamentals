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

### 转发等价类 - FEC

### 标签分发

### 标签分发协议 LDP

### 标签转发表 LFIB

### MPLS 载荷

### MPLS 标签空间

### 不同的 MPLS 运行模式


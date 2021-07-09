# MPLS Fundamentals
- [MPLS Fundamentals](#mpls-fundamentals)
  - [Chapter 1. MPLS 演进过程](#chapter-1-mpls-演进过程)
    - [MPLS的定义](#mpls的定义)
    - [Pre-MPLS Protocols](#pre-mpls-protocols)
    - [MPLS的优势](#mpls的优势)
      - [Bogus Benefit:](#bogus-benefit)
      - [The Use of One Unified Network Infrastructure:](#the-use-of-one-unified-network-infrastructure)
      - [Better IP over ATM Integration:](#better-ip-over-atm-integration)
      - [BGP-Free Core:](#bgp-free-core)
      - [端到端VPN模型对比Overlay VPN模型:](#端到端vpn模型对比overlay-vpn模型)
        - [1.Overlay VPN model:](#1overlay-vpn-model)
        - [2.Peer-to-Peer VPN Model](#2peer-to-peer-vpn-model)
      - [Optimal Traffic Flow](#optimal-traffic-flow)
      - [Traffic Engineering](#traffic-engineering)
    - [MPLS 与思科 IOS 系统的前世今生](#mpls-与思科-ios-系统的前世今生)
      - [MPLS 标签交换](#mpls-标签交换)
      - [MPLS 应用程序](#mpls-应用程序)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Chapter 1. MPLS 演进过程
### MPLS的定义
<p>
对于传统的IP转发，网络中路由器根据路由协议建立转发规则，每一台路由器再根据这个规则来转发报文，因为这个规则是基于IP地址的，所以路由器只会根据数据报文中的目标IP来判断如何转发。而在MPLS架构中，所有支持MPLS的设备会在网络收敛完成后，预先建立一套标签转发规则，并在发送报文时将计算好的标签绑定到IP头部的外面，因此这个报文可以在网络中以标签为导向逐跳被发送去往目的地，从而取代传统的IP转发模式。
</p>
<p>
MPLS中的标签转发概念其实很早就已经有过应用。帧中继和ATM技术就是使用类似的技术在网络中转发frames（帧）或cells（信元）的。帧中继中，报文可以是任意长度的，而在ATM中却使用着固定长度的报文-5字节的信元头部和48字节的信息。帧中继和ATM的报文在网络中都是根据虚链路（类似于预先创建好的一条条隧道，每一条隧道都只允许指定类别的报文通过）来转发的，而帧中继和ATM中有一个共同的并且最重要的性质-“标签交换”，意思是数据包在虚链路中经过每一跳设备时，IP头部前面的“标签”会被更换。这是标签转发和IP转发最为明显的区别，IP转发中头部的目标IP在转发过程中不会改变，而在MPLS标签交换中，标签的意义不再是真正目的地，而是网络设备的转发依据，而这些标签转发规则的生成仍然要依据IP路由协议。
</p>
<p>
听起来MPLS转发的准备工作并不简单，甚至比IP转发更加繁琐，但正是因为这样，才能让MPLS网络可以具备传统IP网络想都不敢想的功能。
</p>

### Pre-MPLS Protocols

<p>
在MPLS出现之前广域网中最流行的协议是ATM和帧中继。运营商利用他们的广域网承载用户的多种协议数据包从而获取利益。随着互联网的兴起，IP成为了网络中最流行的协议。客户的网络中IP无处不在，并且为了远距离组网，客户还会租用运营商的ATM和帧中继线路。在这些VPN线路中，运营商利用WAN中的设备在其3层设备之间提供二层连接服务，并且保证在不同客户之间进行隔离。当时我们称这种网络为overlay网络
</p>
<p>
Overlay网络至今仍被沿用，但现在客户使用MPLS VPN服务代替了之前的ATM和帧中继链路来搭建自己的overlay网络。下一小节会介绍MPLS的优势，你会从中了解到为什么运营商会如此钟爱用MPLS技术帮助客户组网。
</p>

### MPLS的优势

#### Bogus Benefit:

<p>
很早以前，人们使用标签交换协议的重要原因是为了提高转发速度。大家普遍认为在网络设备中相比利用CPU进行IP转发，查找位于数据包顶部的标签信息从而完成数据包的交换效率是更高的。因为路由器转发IP数据包需要拿到位于IP头部中的目标IP地址，然后在自己的路由表中遍历出最优匹配的条目转发，这个遍历查询过程是比较复杂的因为IP地址会是单播或组播地址并且它有4个8位组共32个bit需要进行匹配，所以IP转发相比需要花费更多的时间。
</p>
<p>
虽然很多朋友根据上面的分析认为相比IP转发，路由器可以更快速的对简单的标签进行处理，但是实际上设备中IP交换都是由特殊的硬件处理的而非CPU。当今接口速率动不动就40G，100G甚至400G，在如此多的高速端口面前，设备的CPU是无法完成所有IP转发决策任务的。实际上CPU只负责控制平面的工作。
</p>
<p>
控制平面的任务就是在数据进行高速转发之前，一系列路由协议做出的所有准备工作。其中主要的组件有路由协议，路由表和一些为数据平面做准备的协议信令。数据平面就是数据包通过路由器或交换机进行数据转发的路径。数据转发这件事儿或者说这个数据平面现在都是由专用的芯片ASIC（基于应用的集成电路）完成的而非CPU，利用ASIC在数据平面进行IP转发的速度是不输给标签交换的。这里说一下ASIC和CPU一样，都是芯片，只不过ASIC会将内部结构设备的更适用于某些特殊的应用场景，从而获得特殊任务下的极限速度，好像一个偏科的学生，而CPU更像是一个各科成绩都平平的兄弟，啥啥都能干，但都一般。因此如果你只是单一的认为我们在网络中部署MPLS是因为转发速度问题，那你就太年轻了。
</p>

#### The Use of One Unified Network Infrastructure:

<p>
在MPLS网络中，收到一个报文后，我们会根据数据包的目的地或其他的预先条件来为数据包打上标签，并通过统一的规则进行流量转发。这是MPLS架构最大的优势 - 定义了这样一套统一的架构。正如IP之所以能统一当今网络的一个重要原因是因为所有的应用都利用它进行数据通信，包括移动通信系统。
</p>
<p>
使用基于IP的MPLS架构，你可以传输更多种数据。将标签打在数据包上面，数据就可以在MPLS网络中去往正确的目的地，因为全网的标签转发规则是根据IP路由表生成的，因此标签转发表一旦生成，标签内部的数据就不仅限于IP报文，它们可能是IPv4，IPv6，以太网，HDLC，PPP以及其他2层协议的报文。
</p>
<p>
正是因为这个MPLS可以支持任何2层协议通过其骨干网进行传输的特性，因此大家称之为AToM - Any Transport over MPLS。骨干网中的设备在传输AToM流量的时候，并不需要关心MPLS的负载而只需要观察数据包外层的标签即可转发流量。换句话讲，MPLS标签交换使用了更简单的方式在同一张网络中传输了多种协议的报文。网络设备唯一需要做的就是通过自己的标签转发表，将数据包的入向标签更换为出向标签然后把它发给正确的下一跳。
</p>
<p>
总而言之，AToM可以让运营商在只维护一张统一架构网络的情况下，为客户提供所有类型的二层数据传输服务。
</p>

#### Better IP over ATM Integration:

<p>
在过去的十年中，IP在众多3层协议（如AppleTalk，IPX，DECnet）中脱颖而出，原因是IP简单且实用。同时2层协议的佼佼者就是ATM，尽管它大言不惭地说要做端到端（就是LAN中也都用它）结果没有实现（当时LAN中的主流协议是以太网，FDDI，令牌环），因此它的成功仅限于运营商广域网核心。当时很多运营商在它们的IP骨干中部署ATM，因为技术比较复杂，所以只有很少的解决方案。
</p>
<p>
其中一种方案就是大家熟知的RFC1483（一个90后网工表示根本没听说过），这个RFC的主题叫做“Multiprotocol Encapsulation over ATM Adaptation Layer 5”，里面讲述了阿什杜德后方可鼎折覆餗灯笼裤发生的，总之很复杂 :-)
</p>
<p>
另一种解决方案叫走LANE – LANEmulation，在以太网统一边缘LAN的市场之后，ATM就搞出了这么一套方案，让用户觉得运营商的广域网就好像是连接多个以太网中间的交换机一样。
</p>
<p>
最后，ATM搞出了一套叫做MPOA – Multiprotocol over ATM的解决方案打算统一天下，但无奈也是过于复杂。
</p>
<p>
综上所述，IP over ATM的所有方案想法都是不错，但实施和维护起来都是非常复杂，正式因为这一点才推动了大家发明了MPLS。另外，在ATM交换机中运行MPLS的前提条件是我们需要让这台交换机变得更加智能，因为这台交换机需要同时运行IP和LDP（label distribution protocol）协议。这里第5章会详细说明MPLS和ATM在软件架构上区别。
</p>

#### BGP-Free Core:

<p>
当运营商利用IP网络来转发流量时，网络中所有的路由器都需要根据数据包的目标IP地址进行查询转发，因此如果数据包是去往运营商外部网络时，这些外部网络的路由条目就必须存在于所有路由器的路由表中，而只有BGP中存在外部网络信息，这将意味着运营商所有的路由器都要运行BGP协议。
</p>
<p>
但是！MPLS可以让路由器基于标签而不是目标地址转发数据包，而标签信息位于数据包的头部目的是告诉所有途径的路由器应该将其发送给哪一个下一跳。因此核心路由器不需要再基于目标地址来转发数据，所以我们讲MPLS让运营商核心路由器摆脱了BGP协议。
</p>
<p>
尽管运营商核心路由器不用运行BGP，但MPLS网络的边界设备诶还是需要运行BGP从而查看数据包真正的目标IP地址的。对于每一条BGP路由前缀，MPLS网络入向边界路由器会拥有它的BGP下一跳IP地址信息，这个IP地址就是相对应的MPLS网络出向边界路由器的IP地址。而我们将标签和这些下一跳IP相关联就相当于和那些BGP前缀相关联。因为这样一来网络中所有路由器都可以根据这些基于BGP下一跳分配的标签来转发数据流量了，因为IGP – interior gateway routing protocol像OSPF或ISIS会让网络中所有路由器都知道这些BGP下一跳IP的地址。
</p>

![Figure 1-1. BGP-Free MPLS Network](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image01.gif)

<p>
上图所示，如果没有MPLS一个运营商全网200多台设备都要 BGP（反射器得累死）但部署MPLS后，只有50多台边界设备需要运行BGP。
</p>
<p>
其实当今运营商网络都是标签交换而不是IP转发，因此理论讲他们是有余力去运行BGP的，但是目前互联网的路由已经超过了70万，慎重考虑后，还是不让它们运行BGP比较好，这样减少路由表条目从而降低了大量内存占用。所以为了减小核心路由器的复杂性，还是别run了吧。
</p>

#### 端到端VPN模型对比Overlay VPN模型:

<p>
VPN就是在一张公共的网络设备中，实现仿真的专用网络。专用的网络需要让同一个客户的所有站点间互相访问并且完全隔离于其他的客户VPN网络。一个VPN通常只属于一个客户并且只实现单客户站点间的互相访问。
</p>
<p>
目前大多数运营商可以为客户提供基于两种模型的VPN服务：
</p>

- Overlay VPN model
- Peer-to-Peer VPN model

##### 1.Overlay VPN model:

<p>
在Overlay模型的VPN服务中，运营商为客户提站点间的路由器提供点到点链路或虚链路连接（这里只要针对客户路由器间使用的二层协议类型）。客户的站点间路由器通过运营商网络直接建立邻接关系。运营商的路由器或交换机帮客户承载数据但不和客户的设备有任何路由上的往来。（就是二层VPN）
</p>
<p>
这些点到点服务可能是1层，2层甚至3层传输，例如，1层的TDM，E1，E3，SONET和SDH（我93年的，说实话我都没听过）。2层的X.25,ATM或帧中继
</p>

![Figure 1-2. Overlay Network on Frame Relay](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image02.gif)

<p>
上图展示了一个建立在帧中继网络上的overlay网络。其中运营商利用帧中继网络建立的虚链路帮助客户在边界路由器间建立连接。
</p>
<p>
从客户的角度看，建立起来的3层overlay网络就像是路由器直连一样如下图所示
</p>

![Figure 1-3. Overlay Network: Customer Routing Peering](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image03.gif)

<p>
上面这种overlay的服务也可以通过3层的协议实现。GRE – Generic Routing Encapsulation就是在IP网络中通过隧道实现overlay功能中最常用的技术。GRE会将原本的IP报文封装一层GRE包头和IP包头。GRE头部描述了内层的协议是什么，IP包头用于数据包在运营商网络中传输。下图可以看到GRE在IP环境中搭建overlay网络的样子，除了IP之外，GRE还支持不同的underlay网络。
</p>

![Figure 1-4. Overlay Network on GRE Tunnels](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image04.gif)

<p>
我们也可以通过IPsec来加密GRE隧道中的数据来使数据更安全。
</p>

##### 2.Peer-to-Peer VPN Model

<p>
在这种模型中，虽然也是帮客户转发数据，但是运营商参与了客户的路由过程。也就是运营商和客户的边界路由器建立了直连的3层邻居关系。结果就是客户要和运营商之间运行路由协议。下图表示了这种Peer-to-Peer VPN model的概念。
</p>

![Figure 1-5. Peer-to-Peer VPN Model](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image05.gif)

<p>
在MPLS诞生之前，peer-to-peer VPN模型可以通过运营商与客户的路由器之间的IP路由实现。但VPN模型中需要实现客户间的隔离，因此第一种方法我们可以通过ACL来对发往或从客户路由器收到的数据进行过滤，另一种方法就是从控制平面直接过滤路由，或者同时做这两件事。
</p>
<p>
在MPLS流行之前，overlay模型的VPN更为普及，原因是peer-to-peer模型的VPN配置过于复杂 – 客户为了添加一个site要同时在多个site进行大量配置。MPLS VPN作为一个MPLS的应用就可以解决上述的问题，让peer-to-peer模型的VPN部署更简单。在MPLS VPN中，客户的路由器我们叫做customer edge(CE)，运营商侧的IP设备我们叫做provider edge(PE)。
</p>
<p>
MPLS VPN体系中，多用户间的隔离是通过VRF（virtual routing/forwarding）以及骨干网中的标签交换机制实现的。VRF确保了不同用户间的路由信息保持独立，MPLS实现了客户的数据包在骨干网中通过标签而不是IP头部中的信息进行路由传输。下图简述了该模式。
</p>

![Figure 1-6. MPLS VPN with VRF](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image06.gif)

<p>
下图展示了将MPLS VPN技术应用在peer-to-peer VPN模型中的样子。
</p>

![Figure 1-7. Peer-to-Peer MPLS VPN Model](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image07.gif)

<p>
这样一来，为客户添加一个site只意味着需要PE和新的CE建立联系。你不必再需要像overlay VPN模型一样为新的site创建很多条虚链路或像之前的peer-to-peer VPN模型一样，添加很多路由及数据平面相关的过滤配置。这就是MPLS VPN这个应用为运营商带来的最大的好处。
</p>
<p>
虽然大部分运营商的客户都用hub-and-spoke模型进行组网，但仍然有一些愣头青客户在骨干网中采用全互联组网。在全互联的模式下，帧中继技术和MPLS VPN技术的差别就比较大了，使用帧中继的话，假设有n个站点，每一个站点都需要建立n-1条虚链路，相比之下，MPLS VPN模式中，每一个客户边界路由器只需要和运营商边界路由器建立好关系就搞定了。
</p>
<p>
MPLS VPN技术对运营商的另一个绝对优势就是我们只需要维护PE和CE之间的关系，而在overlay模式中，对于每一个站点，我们都要维护它与其他站点之间的关系It is much easier to predict the traffic and thus the bandwidth requirement of one site than to predict the complete traffic model between all the customer sites.
</p>
<p>
公平起见，我们再来看看相比overlay VPN，peer-to-peer VPN的劣势：
</p>

- 客户必须与运营商间运行路由协议，这意味着大家要共同承担控制平面的风险。
- 因为会有很对客户的CE接到同一个PE上，所以对于运营商的PE来说，控制平面和数据平面的压力都会很大

<p>
第一个劣势是针对客户的，peer-to-peer模式中客户在路由层面上不能做到端到端的控制而overlay模式就可以。第二个劣势是针对运营商的，因为PE设备参与了所有客户的路由管理，这意味着PE要对客户的路由扩展还有收敛负责。
</p>

#### Optimal Traffic Flow

<p>
ATM和帧中继就是一个纯粹的2层VPN，客户路由器之间想要连接就需要建立virtual circuits虚链路。这意味着任何两个边界路由器之间要完成可达，必须为他们单独建立一条虚链路。如果客户的需求是站点间any-to-any访问，那我们就需要在所有边界路由器建立全互联的虚链路，很痛苦。
</p>
<p>
所以一个更好的方案是建立一个hub-spoke型的网络，如下图：
</p>

![Figure 1-8. Non-Fully Meshed Overlay ATM Network](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image08.gif)

<p>
这样做的结果是所有spoke端的流量都会经过hub站点CE2。当我们使用MPLS-VPN（peer-to-peer）时流量总是直达的，因此优化了站点间的流量。如果我们想在overlay VPN模型中也实现这一点，那就必须建立一个全互联的虚链路。
</p>

#### Traffic Engineering

<p>
流量工程的重要作用是优化网络基础设施的利用率，例如那些非最优路径上的低利用率链路资源。这就意味着流量工程需要有能力操纵流量让它们不总是走IGP最优路径。一般IGP最优路径是通过最小cost计算出来的。在部署了流量工程的 MPLS 网络中，你的流量从 A 点到 B 点可以不走最小 IGP cost 的路径，这样的好处是让流量在网络的可用链路中分部的更加均匀，让低利用率链路更为充分被使用。
</p>

![Figure 1-9. Traffic Engineering Example 1](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image09.gif)

<p>
如上图，在部署了 MPLS Traffic Engineering 的网络中，你可以让 A 到 B 的流量走下面的路径而不是 IGP 最短路径。因此你可以更好地利用在正常情况下处于空闲状态的链路。
</p>

![Figure 1-10. Traffic Engineering Example 2](https://learning.oreilly.com/library/view/mpls-fundamentals/1587051974/1587051974_ch01lev1sec3_image10.gif)

<p>
如果上图是一个纯 IP 网络，那么你讲不可能通过配置 A 来让 C 走下面路径到达 B，因为 当 C 收到一个 IP 报文时，永远会自己做出转发选择。但如果你在 A 上开启了 MPLS流量工程，那么问题将会变得有趣，由于 MPLS网络靠顶层标签进行数据转发，所以 A 可以通过流量工程的顶层标签来让 C 强行从下面的路径转发报文。甚至说在 A 上我们可以定义一个端到端完整的路径，这就叫原路由。
</p>
<p>
部署 MPLS 流量工程的另外一个好处就是 FRR（Fast Re-Route 快速重路由），一般 IGP 拓扑中哪个链路或者节点要是断了，IGP 要重新收敛，这会在短时间内丢包，为了把丢包时间降到最短（目前的标准是 50ms） 我们就需要在指定节点中安装备份路径，一点系统检测链路失效，立刻切换FIB 中的转发路径，让流量避开失效链路或节点后继续前进。
</p>

### MPLS 与思科 IOS 系统的前世今生
<p>
这章给你讲讲从 1998 年以来，MPLS 与思科 IOS 系统之间的故事。
</p>

#### MPLS 标签交换
<p>
思科刚刚发明在IP 头部上层打标签时，这个技术还叫做标签交换（tag switching）。思科是 1998 年的时候将这项技术发布在 IOS 11.1(17)CT 中的。这项技术可以通过路由表来给不同的网络打上标签，然后在数据包的顶部压上标签来表示数据想去往那个网络。标签交换构建了一个标签转发表（TFIB），这张表用来存储入标签和出标签之间的映射关系。运行标签交换的路由器收到每一个数据包时，都要根据这张表来匹配相应的入标签，转换成出标签，然后转发。
</p>

#### MPLS 应用程序
<p>
在 带有标签交换的第一个 IOS 版本中，就支持流量工程了，只不过那时候它叫做 RRR（Routing with Resource Reservation）。第一版的流量工程只支持静态显示路径，意思就是你只能在隧道的源点指定隧道的每一跳。在之后的实现中，借力于 IGP 的链路状态数据库，它可以支持动态路径，变得更加灵活，因为 IGP 可以帮忙携带并通告关于流量工程的数据。这样大大地减少了部署流量工程的操作难度，从而使 MPLS 流量工程广为流传。
</p>
<p>
在 MPLS VPN 应用发布之前，MPLS 并不是太受人待见，但在 1999 年，思科在 IOS12.0(5)T 中第一次发布 MPLS VPN 功能后，大家都惊呆了，全世界的运营商如同疯狗一样抢着在自己的网络中部署 MPLS VPN 功能。即使是现在 2021 年，MPLS VPN 也被几乎所有运营商作为核心功能部署在生产网络中。（网络工程师真正学好一项技术可以受用 20 年）
</p>
<p>
MPLS 的下一个重要应用就是 AToM（Anything Transport on MPLS《MPLS啥都能传》），这是思科在 2000 年发布于 IOS12.0(10)的一个应用，当时是为了在 MPLS 骨干网中传输 ATM 数据包。随后，越来越多的二层数据包加入到 AtoM 的支持列表中。现在 AToM 支持的协议有 – 帧中继，ATM，PPP，HDLC，以太网和 802.1Q，尤其是以太网在 AToM 中 取得了巨大的成功，主要因为现在以太网已经占领二层市场了。然而，由于 AToM 本身的限制，它只能以点到点的形式在 MPLS 骨干网中传输二层报文，也就是如今所说的 VPWS。 而 随后VPLS(Virtual Private LAN Service) 实现了以太网报文的点到多点传输。本质上 VPLS 就是在 MPLS 骨干网中模拟了一台以太网交换机。VPLS 是 2004 年思科 IOS12.2(17d)SXB 在 7600 平台上发布的。第 11 章会详细介绍 VPLS。
</p>
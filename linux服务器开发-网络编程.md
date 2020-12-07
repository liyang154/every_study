#  网络基础

## 分层模型

![image-20201026163308840](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026163308840.png)

### OSI七层模型

物、数、网、传、会、表、应

​                               

OSI模型

1. **物理层**：主要定义物理设备标准，如网线的接口类型、光纤的接口类型、各种传输介质的传输速率等。它的主要作用是传输比特流（就是由1、0转化为电流强弱来进行传输，到达目的地后再转化为1、0，也就是我们常说的数模转换与模数转换）。这一层的数据叫做比特。

2. **数据链路层**：定义了如何让格式化数据以帧为单位进行传输，以及如何让控制对物理介质的访问。这一层通常还提供错误检测和纠正，以确保数据的可靠传输。如：串口通信中使用到的115200、8、N、1

3.  **网络层**：在位于不同地理位置的网络中的两个主机系统之间提供连接和路径选择。Internet的发展使得从世界各站点访问信息的用户数大大增加，而网络层正是管理这种连接的层。

4. **传输层**：定义了一些传输数据的协议和端口号（WWW端口80等），如：TCP（传输控制协议，传输效率低，可靠性强，用于传输可靠性要求高，数据量大的数据），UDP（用户数据报协议，与TCP特性恰恰相反，用于传输可靠性要求不高，数据量小的数据，如QQ聊天数据就是通过这种方式传输的）。 主要是将从下层接收的数据进行分段和传输，到达目的地址后再进行重组。常常把这一层数据叫做段。

5. **会话层**：通过传输层(端口号：传输端口与接收端口)建立数据传输的通路。主要在你的系统之间发起会话或者接受会话请求（设备之间需要互相认识可以是IP也可以是MAC或者是主机名）。

6. **表示层**：可确保一个系统的应用层所发送的信息可以被另一个系统的应用层读取。例如，PC程序与另一台计算机进行通信，其中一台计算机使用扩展二一十进制交换码(EBCDIC)，而另一台则使用美国信息交换标准码（ASCII）来表示相同的字符。如有必要，表示层会通过使用一种通格式来实现多种数据格式之间的转换。

7. **应用层**：是最靠近用户的OSI层。这一层为用户的应用程序（例如电子邮件、文件传输和终端仿真）提供网络服务。

### TCP/IP四层模型

TCP/IP网络协议栈分为应用层（Application）、传输层（Transport）、网络层（Network）和链路层（Link）四层。如下图所示：

 <img src="C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026163327743.png" alt="image-20201026163327743" style="zoom:150%;" />

TCP/IP模型

一般在应用开发过程中，讨论最多的是TCP/IP模型。

## 通信过程

**数据在没有封装之前，是不能在网络中传递的。**

两台计算机通过TCP/IP协议通讯的过程如下所示：

![image-20201026163734695](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026163734695.png)

上图对应两台计算机在同一网段中的情况，如果两台计算机在不同的网段中，那么数据从一台计算机到另一台计算机传输过程中要经过一个或多个路由器，如下图所示：

![image-20201026163753188](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026163753188.png)

链路层有以太网、令牌环网等标准，链路层负责网卡设备的驱动、帧同步（即从网线上检测到什么信号算作新帧的开始）、冲突检测（如果检测到冲突就自动重发）、数据差错校验等工作。**交换机**是工作在**链路层**的网络设备，**可以在不同的链路层网络之间转发数据帧**（比如十兆以太网和百兆以太网之间、以太网和令牌环网之间），由于不同链路层的帧格式不同，**交换机要将进来的数据包拆掉链路层首部重新封装之后再转发。**

网络层的IP协议是构成Internet的基础。Internet上的主机通过IP地址来标识，Internet上有大量路由器负责根据IP地址选择合适的路径转发数据包，数据包从Internet上的源主机到目的主机往往要经过十多个路由器。**路由器是工作在第三层的网络设备（网络层）**，**同时兼有交换机的功能**，可以在不同的链路层接口之间转发数据包，因此**路由器需要将进来的数据包拆掉网络层和链路层两层首部并重新封装**。IP协议不保证传输的可靠性，数据包在传输过程中可能丢失，可靠性可以在上层协议或应用程序中提供支持。

网络层负责点到点（ptop，point-to-point）的传输（这里的“点”指主机或路由器），而传输层负责端到端（etoe，end-to-end）的传输（这里的“端”指源主机和目的主机）。传输层可选择TCP或UDP协议。

TCP是一种面向连接的、可靠的协议，有点像打电话，双方拿起电话互通身份之后就建立了连接，然后说话就行了，这边说的话那边保证听得到，并且是按说话的顺序听到的，说完话挂机断开连接。也就是说TCP传输的双方需要首先建立连接，之后由TCP协议保证数据收发的可靠性，丢失的数据包自动重发，上层应用程序收到的总是可靠的数据流，通讯之后关闭连接。

UDP是无连接的传输协议，不保证可靠性，有点像寄信，信写好放到邮筒里，既不能保证信件在邮递过程中不会丢失，也不能保证信件寄送顺序。使用UDP协议的应用程序需要自己完成丢包重发、消息排序等工作。

目的主机收到数据包后，如何经过各层协议栈最后到达应用程序呢？其过程如下图所示：

![image-20201026163823301](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026163823301.png)

以太网驱动程序首先根据以太网首部中的“上层协议”字段确定该数据帧的有效载荷（payload，指除去协议首部之外实际传输的数据）是IP、ARP还是RARP协议的数据报，然后交给相应的协议处理。假如是IP数据报，IP协议再根据IP首部中的“上层协议”字段确定该数据报的有效载荷是TCP、UDP、ICMP还是IGMP，然后交给相应的协议处理。假如是TCP段或UDP段，TCP或UDP协议再根据TCP首部或UDP首部的“端口号”字段确定应该将应用层数据交给哪个用户进程。IP地址是标识网络中不同主机的地址，而端口号就是同一台主机上标识不同进程的地址，IP地址和端口号合起来标识网络中唯一的进程。

虽然IP、ARP和RARP数据报都需要以太网驱动程序来封装成帧，但是从功能上划分，ARP和RARP属于链路层，IP属于网络层。虽然ICMP、IGMP、TCP、UDP的数据都需要IP协议来封装成数据报，但是从功能上划分，ICMP、IGMP与IP同属于网络层，TCP和UDP属于传输层。

## 协议格式

### 数据包封装

**传输层及其以下的机制由内核提供**，应用层由用户进程提供（后面将介绍如何使用socket API编写应用程序），应用程序对通讯数据的含义进行解释，而传输层及其以下处理通讯的细节，将数据从一台计算机通过一定的路径发送到另一台计算机。应用层数据通过协议栈发到网络上时，每层协议都要加上一个数据首部（header），称为封装（Encapsulation），如下图所示：

![image-20201026164856438](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026164856438.png)

不同的协议层对数据包有不同的称谓，在传输层叫做段（segment），在网络层叫做数据报（datagram），在链路层叫做帧（frame）。数据封装成帧后发到传输介质上，到达目的主机后每层协议再剥掉相应的首部，最后将应用层数据交给应用程序处理。

### 以太网帧格式

以太网的帧格式如下所示：

类型：0x0806:ARP 请求

​			0x0800:ip数据包

​			0x0835:RARP

![image-20201026164952537](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026164952537.png)

其中的源地址和目的地址是指网卡的硬件地址（也叫MAC地址），长度是48位（6字节），是在网卡出厂时固化的。可在shell中使用ifconfig命令查看，“HWaddr 00:15:F2:14:9E:3F”部分就是硬件地址。协议字段有三种值，分别对应IP、ARP、RARP。帧尾是CRC校验码。

以太网帧中的数据长度规定最小46字节，最大1500字节，ARP和RARP数据包的长度不够46字节，要在后面补填充位。**最大值1500称为以太网的最大传输单元（MTU）**，不同的网络类型有不同的MTU，如果一个数据包从以太网路由到拨号链路上，数据包长度大于拨号链路的MTU，则需要对数据包进行分片（fragmentation）。ifconfig命令输出中也有“MTU:1500”。注意，**MTU这个概念指数据帧中有效载荷的最大长度，不包括帧头长度**。

### ARP数据报格式

在网络通讯时，源主机的应用程序知道目的主机的IP地址和端口号，却不知道目的主机的硬件地址，而数据包首先是被网卡接收到再去处理上层协议的，如果接收到的数据包的硬件地址与本机不符，则直接丢弃。因此在通讯前必须获得目的主机的硬件地址。ARP协议就起到这个作用。源主机发出ARP请求，询问“IP地址是192.168.0.1的主机的硬件地址是多少”，并将这个请求广播到本地网段（以太网帧首部的硬件地址填FF:FF:FF:FF:FF:FF表示广播（请求mac地址）），目的主机接收到广播的ARP请求，发现其中的IP地址与本机相符，则发送一个ARP应答数据包给源主机，将自己的硬件地址填写在应答包中。

![image-20201026183843546](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026183843546.png)

op字段为1表示ARP请求，op字段为2表示ARP应答。

### RARP

反向地址转换协议（RARP：Reverse Address Resolution Protocol） 反向地址转换协议（RARP）允许局域网的物理机器从[网关](https://baike.baidu.com/item/网关/98992)服务器的 ARP 表或者缓存上请求其 IP 地址。[网络管理员](https://baike.baidu.com/item/网络管理员/595848)在局域网[网关](https://baike.baidu.com/item/网关/98992)[路由器](https://baike.baidu.com/item/路由器/108294)里创建一个表以映射[物理地址](https://baike.baidu.com/item/物理地址/2129)（MAC）和与其对应的 IP 地址。当设置一台新的机器时，其 RARP 客户机程序需要向[路由器](https://baike.baidu.com/item/路由器/108294)上的 RARP 服务器请求相应的 IP 地址。假设在[路由表](https://baike.baidu.com/item/路由表/2707408)中已经设置了一个记录，RARP 服务器将会返回 IP 地址给机器，此机器就会存储起来以便日后使用。 RARP 可以使用于[以太网](https://baike.baidu.com/item/以太网/99684)、[光纤分布式数据接口](https://baike.baidu.com/item/光纤分布式数据接口/3004891)及[令牌环](https://baike.baidu.com/item/令牌环/986104) LAN。

### IP协议

![image-20201026184619055](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026184619055.png)

IP数据报的首部长度和数据长度都是可变长的，**但总是4字节的整数倍**。对于IPv4，4位版本字段是4。4位首部长度的数值是以4字节为单位的，**最小值为****5**，也就是说首部长度最小是4x5=20字节，也就是不带任何选项的IP首部，4位能表示的最大值是15，也就是说首部长度最大是60字节**。8位TOS字段有3个位用来指定IP数据报的优先级（目前已经废弃不用），还有4个位表示可选的服务类型（最小延迟、最大?吐量、最大可靠性、最小成本），还有一个位总是0。总长度是整个数据报（包括IP首部和IP层payload）的字节数。每传一个IP数据报，16位的标识加1，可用于分片和重新组装数据报。3位标志和13位片偏移用于分片。

​		TTL（Time to live)是这样用的：**源主机为数据包设定一个生存时间，比如64，每过一个路由器就把该值减1，如果减到0就表示路由已经太长了仍然找不到目的主机的网络，就丢弃该包，因此这个生存时间的单位不是秒，而是跳（hop）。**协议字段指示上层协议是TCP、UDP、ICMP还是IGMP。然后是**校验和，只校验IP首部，数据的校验由更高层协议负责。**IPv4的IP地址长度为32位。

​		192.168.103.92是点分十进制（string）-------》 二进制（网络中传输）

### UDP数据报格式

​                               ![image-20201026185603174](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026185603174.png)

### TCP协议

#### TCP 报文格式

![image-20201026190609703](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026190609703.png)

#### TCP 通信时序

![image-20201027182609345](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201027182609345.png)

#### 滑动窗口 (TCP流量控制)

介绍UDP时我们描述了这样的问题：如果发送端发送的速度较快，接收端接收到数据后处理的速度较慢，而接收缓冲区的大小是固定的，就会丢失数据。TCP协议通过“滑动窗口（Sliding Window）”机制解决这一问题。看下图的通讯过程：

![image-20201027185041916](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201027185041916.png)

#### 半关闭

当TCP链接中A发送FIN请求关闭，B端回应ACK后（A端进入FIN_WAIT_2状态），B没有立即发送FIN给A时，A方处在半链接状态，此时A可以接收B发送的数据，但是A已不能再向B发送数据。

#### 三次握手

​	主动发起连接请求端，发送SYN标志位，请求建立连接。携带数据包包号、数据字节数（0）、滑动窗口大小。

​	被动接受请求端，发送ACK标志位，同时携带SYN标志位。携带序号、确认序号、数据字节数（0）、滑动窗口大小。

​	主动发起连接请求端，发送ACK 标志位，应答服务器连接请求。

#### 四次挥手

​	主动关闭连接请求端，发送FIN标志位。

​	被动关闭连接请求端，应答ACK标志位。							-------半关闭完成

​	被动关闭连接请求端，发送FIN标志位。

​	主动关闭连接请求端，应答ACK标志位								-------连接全部关闭

## 网络应用程序设计模式

### C/S模式

​     传统的网络应用设计模式，客户机(client)/服务器(server)模式。需要在通讯两端各自部署客户机和服务器来完成数据通信。

### B/S模式

浏览器()/服务器(server)模式。只需在一端部署服务器，而另外一端使用每台PC都默认配置的浏览器即可完成数据的传输。

### 优缺点

C/S   

 优点：缓存大量数据、协议选择灵活、速度快

缺点：安全性低、开发工作量较大

B/S

优点：安全性高、跨平台、开发工作量较小

缺点：不能缓存大量数据、严格遵守HTTP

## 端口复用

在server的TCP连接没有完全断开之前不允许重新监听是不合理的。因为，TCP连接没有完全断开指的是connfd（127.0.0.1:6666）没有完全断开，而我们重新监听的是lis-tenfd（0.0.0.0:6666），虽然是占用同一个端口，但IP地址不同，connfd对应的是与某个客户端通讯的一个具体的IP地址，而listenfd对应的是wildcard address。解决这个问题的方法是使用setsockopt()设置socket描述符的选项SO_REUSEADDR为1，表示允许创建端口号相同但IP地址不同的多个socket描述符。

在server代码的socket()和bind()调用之间插入如下代码：

```c
int opt = 1; 
setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
```

## TCP状态转换

这个图N多人都知道，它排除和定位网络或系统故障时大有帮助，但是怎样牢牢地将这张图刻在脑中呢？那么你就一定要对这张图的每一个状态，及转换的过程有深刻的认识，不能只停留在一知半解之中。下面对这张图的11种状态详细解析一下，以便加强记忆！不过在这之前，先回顾一下TCP建立连接的三次握手过程，以及 关闭连接的四次握手过程。

​                       ![image-20201113212937027](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201113212937027.png)        

TCP状态转换图

**CLOSED****：**表示初始状态。

**LISTEN****：**该状态表示服务器端的某个SOCKET处于监听状态，可以接受连接。

**SYN_SENT****：**这个状态与SYN_RCVD遥相呼应，当客户端SOCKET执行CONNECT连接时，它首先发送SYN报文，随即进入到了SYN_SENT状态，并等待服务端的发送三次握手中的第2个报文。SYN_SENT状态表示客户端已发送SYN报文。

**SYN_RCVD:** 该状态表示接收到SYN报文，在正常情况下，这个状态是服务器端的SOCKET在建立TCP连接时的三次握手会话过程中的一个中间状态，很短暂。此种状态时，当收到客户端的ACK报文后，会进入到ESTABLISHED状态。

**ESTABLISHED****：**表示连接已经建立。

**FIN_WAIT_1:**  FIN_WAIT_1和FIN_WAIT_2状态的真正含义都是表示等待对方的FIN报文。区别是：

FIN_WAIT_1状态是当socket在ESTABLISHED状态时，想主动关闭连接，向对方发送了FIN报文，此时该socket进入到FIN_WAIT_1状态。

FIN_WAIT_2状态是当对方回应ACK后，该socket进入到FIN_WAIT_2状态，正常情况下，对方应马上回应ACK报文，所以FIN_WAIT_1状态一般较难见到，而FIN_WAIT_2状态可用netstat看到。

**FIN_WAIT_2**：主动关闭链接的一方，发出FIN收到ACK以后进入该状态。称之为半连接或半关闭状态。该状态下的socket只能接收数据，不能发。

**TIME_WAIT:** 表示收到了对方的FIN报文，并发送出了ACK报文，等2MSL后即可回到CLOSED可用状态。如果FIN_WAIT_1状态下，收到对方同时带 FIN标志和ACK标志的报文时，可以直接进入到TIME_WAIT状态，而无须经过FIN_WAIT_2状态。

**CLOSING:** 这种状态较特殊，属于一种较罕见的状态。正常情况下，当你发送FIN报文后，按理来说是应该先收到（或同时收到）对方的 ACK报文，再收到对方的FIN报文。但是CLOSING状态表示你发送FIN报文后，并没有收到对方的ACK报文，反而却也收到了对方的FIN报文。什么情况下会出现此种情况呢？如果双方几乎在同时close一个SOCKET的话，那么就出现了双方同时发送FIN报文的情况，也即会出现CLOSING状态，表示双方都正在关闭SOCKET连接。

**CLOSE_WAIT:** 此种状态表示在等待关闭。当对方关闭一个SOCKET后发送FIN报文给自己，系统会回应一个ACK报文给对方，此时则进入到CLOSE_WAIT状态。接下来呢，察看是否还有数据发送给对方，如果没有可以 close这个SOCKET，发送FIN报文给对方，即关闭连接。所以在CLOSE_WAIT状态下，需要关闭连接。

**LAST_ACK:** 该状态是被动关闭一方在发送FIN报文后，最后等待对方的ACK报文。当收到ACK报文后，即可以进入到CLOSED可用状态。

## 半关闭

通信双方中，只有一端关闭通信。--FIN_WAIT_2

当TCP链接中A发送FIN请求关闭，B端回应ACK后（A端进入FIN_WAIT_2状态），B没有立即发送FIN给A时，A方处在半链接状态，此时A可以接收B发送的数据，但是A已不能再向B发送数据。

从程序的角度，可以使用API来控制实现半连接状态。

\#include <sys/socket.h>

int shutdown(int sockfd, int how);

sockfd: 需要关闭的socket的描述符

how:  允许为shutdown操作选择以下几种方式:

  SHUT_RD(0)：  关闭sockfd上的读功能，此选项将不允许sockfd进行读操作。

​          该套接字**不再接受数据**，任何当前在套接字接受缓冲区的数据将被无声的丢弃掉。

  SHUT_WR(1):   关闭sockfd的写功能，此选项将不允许sockfd进行写操作。进程不能在对此套接字发出写操作。

  SHUT_RDWR(2):  关闭sockfd的读写功能。相当于调用shutdown两次：首先是以SHUT_RD,然后以SHUT_WR。

使用close中止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为0时才关闭连接。

**shutdown**不考虑描述符的引用计数，直接关闭描述符。也可选择中止一个方向的连接，只中止读或只中止写。

shutdown在关闭多个文件描述符的应用的文件时，采用全关闭方法。close,只关闭1个。

注意:

1. 如果有多个进程共享一个套接字，close每被调用一次，计数减1，直到计数为0时，也就是所用进程都调用了close，套接字将被释放。

2. 在多进程中如果一个进程调用了shutdown(sfd, SHUT_RDWR)后，其它的进程将无法进行通信。但，如果一个进程close(sfd)将不会影响到其它进程。

# SOCKET编程

## 套接字概念

在TCP/IP协议中，“IP地址+TCP或UDP端口号”唯一标识网络通讯中的一个进程。“IP地址+端口号”就对应一个socket。欲建立连接的两个进程各自有一个socket来标识，那么这两个socket组成的socket pair就唯一标识一个连接。因此可以用Socket来描述网络连接的一对一关系。

套接字通信原理如下图所示：

![image-20201026200039955](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026200039955.png)

**在网络通信中，套接字一定是成对出现的。**一端的发送缓冲区对应对端的接收缓冲区。我们使用同一个文件描述符指向发送缓冲区和接收缓冲区。

## 预备知识

### 网络字节序

小端法（PC本地存储）：      高位存高地址。低位存低地址。

大端法（网络）：	   低位存高地址。高位存低地址。

计算机采用的是小端法，网络采用的是大端法。

TCP/IP协议规定，网络数据流应采用大端字节序，即低地址高字节。为使网络程序具有可移植性，使同样的C代码在大端和小端计算机上编译后都能正常运行，可以调用以下库函数做**网络字节序和主机字节序的转换**。

192.168.1.11（点分十进制string）--->通过atoi转成int--->htonl---->网络字节序

```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);		//主机转网络（IP）
uint16_t htons(uint16_t hostshort);		//主机转网络（port）
uint32_t ntohl(uint32_t netlong);		//网络转主机（IP）
uint16_t ntohs(uint16_t netshort);		//网络转本地（port）
```

### IP地址转换函数

早期：

\#include <sys/socket.h>

\#include <netinet/in.h>

\#include <arpa/inet.h>

int inet_aton(const char *cp, struct in_addr *inp);

in_addr_t inet_addr(const char *cp);

char *inet_ntoa(struct in_addr in);

只能处理IPv4的ip地址

不可重入函数

注意参数是struct in_addr

**现在：**

  \#include <arpa/inet.h>

  int inet_pton(int af, const char *src, void *dst);		//点分十进制转网络字节序

参数：

​	af:AF_INET、AF_INET6

​	src:传入参数，ip地址（点分十进制）

​	dst:传出参数、转换后的网络字节序的IP地址

返回值：

​	成功：1；异常：0，说明src指向的不是一个有效的ip地址；失败：-1

  const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);	//网络字节序（二进制）转点分十进制

参数：

​	af:AF_INET、AF_INET6

​	src:传入参数，网络字节序的IP地址

​	dst:传出参数、主机字节序（string）

​	size：dst的大小

返回值：

​	成功：返回dst;失败：返回NULL

支持IPv4和IPv6

可重入函数

其中inet_pton和inet_ntop不仅可以转换IPv4的in_addr，还可以转换IPv6的in6_addr。

### sockaddr数据结构

头文件

#include<arpa/inet.h>

strcut sockaddr 很多网络编程函数诞生早于IPv4协议，那时候都使用的是sockaddr结构体,为了向前兼容，现在sockaddr退化成了（void *）的作用，传递一个地址给函数，至于这个函数是sockaddr_in还是sockaddr_in6，由地址族确定，然后函数内部再强制类型转化为所需的地址类型。

![image-20201026210029917](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026210029917.png)

struct sockaddr {

  sa_family_t sa_family;   /* address family, AF_xxx */

  char sa_data[14];     /* 14 bytes of protocol address */

};

使用 sudo grep -r "struct sockaddr_in {"  /usr 命令可查看到struct sockaddr_in结构体的定义。一般其默认的存储位置：/usr/include/linux/in.h 文件中。**man 7 ip**

struct sockaddr_in {

  __kernel_sa_family_t sin_family;      /* Address family */  地址结构类型

  __be16 sin_port;               /* Port number */   端口号

  struct in_addr sin_addr;          /* Internet address */ IP地址

  /* Pad to size of `struct sockaddr'. */

  unsigned char __pad[__SOCK_SIZE__ - sizeof(short int) -

  sizeof(unsigned short int) - sizeof(struct in_addr)];

};

 

struct in_addr {            /* Internet address. */

  __be32 s_addr;

};

**用法：**

​	struct sockaddr_in addr;

​	addr.sin_family =AF_INET/AF_INET6

​	addr.sin_port=htons(9527)

​	【*】addr.sin_addr.s_addr=htonl(INADDR_ANY);	//INADDR_ANY取出系统中有效的任意ip地址。二进制类型

​	bind(fd,(struct sockaddr *)&addr,size)   

注【*】也可以用以下方式代替

int dst;

inet_pton(AF_INET,"192.168.0.102",(void *)&dst);

addr.sin_addr.s_addr=dst;

## 网络套接字函数

### socket模型创建流程图

**一对客户端与服务器一共有三个套接字。**

![image-20201026211545208](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201026211545208.png)

### socket函数

创建一个套接字

```html
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
domain:
	AF_INET 这是大多数用来产生socket的协议，使用TCP或UDP来传输，用IPv4的地址
	AF_INET6 与上面类似，不过是来用IPv6的地址
	AF_UNIX 本地协议，使用在Unix和Linux系统上，一般都是当客户端和服务器在同一台及其上的时候使用
type:
	SOCK_STREAM 这个协议是按照顺序的、可靠的、数据完整的基于字节流的连接。这是一个使用最多的socket类型，这个socket是使用TCP来进行传输。
	SOCK_DGRAM 这个协议是无连接的、固定长度的传输调用。该协议是不可靠的，使用UDP来进行它的连接。
	SOCK_SEQPACKET该协议是双线路的、可靠的连接，发送固定长度的数据包进行传输。必须把这个包完整的接受才能进行读取。
	SOCK_RAW socket类型提供单一的网络访问，这个socket类型使用ICMP公共协议。（ping、traceroute使用该协议）
	SOCK_RDM 这个类型是很少使用的，在大部分的操作系统上没有实现，它是提供给数据链路层使用，不保证数据包的顺序
protocol:
	传0 表示使用默认协议。
返回值：成功：新套接字所对应的文件描述符；失败：-1，errorno

```

**fd=socket(AF_INET,SOCK_STREAM,0)**

### bind函数

给socket绑定一个地址结构（IP+port）

```c
#include <sys/types.h> /* See NOTES */

#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

sockfd：

  socket文件描述符；即socket的返回值

addr:
	需要初始化
    struct sockaddr_in addr;

​	addr.sin_family =AF_INET/AF_INET6

​	addr.sin_port=htons(9527)

​	【*】addr.sin_addr.s_addr=htonl(INADDR_ANY);	//INADDR_ANY取出系统中有效的任意ip地址。二进制类型
  构造出IP地址加端口号

addrlen:

  sizeof(addr)长度

返回值：

  成功返回0，失败返回-1, 设置errno
```

bind()的作用是将参数sockfd和addr绑定在一起，使sockfd这个用于网络通讯的文件描述符监听addr所描述的地址和端口号。前面讲过，struct sockaddr *是一个通用指针类型，addr参数实际上可以接受多种协议的sockaddr结构体，而它们的长度各不相同，所以需要第三个参数addrlen指定结构体的长度。如：

struct sockaddr_in servaddr;

bzero(&servaddr, sizeof(servaddr));

servaddr.sin_family = AF_INET;

servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

servaddr.sin_port = htons(6666);

首先将整个结构体清零，然后设置地址类型为AF_INET，**网络地址为INADDR_ANY，这个宏表示本地的任意IP地址**，因为服务器可能有多个网卡，每个网卡也可能绑定多个IP地址，这样设置可以在所有的IP地址上监听，直到与某个客户端建立了连接时才确定下来到底用哪个IP地址，端口号为6666。

### listen函数

设置同时与服务器建立连接的上限数

```c
#include <sys/types.h> /* See NOTES */

#include <sys/socket.h>

int listen(int sockfd, int backlog);
sockfd:socket函数返回值
backlog:上限数值。最大128
返回值：
    成功：0；失败-1，errno
```

查看系统默认backlog

**cat /proc/sys/net/ipv4/tcp_max_syn_backlog**

典型的服务器程序可以同时服务于多个客户端，当有客户端发起连接时，服务器调用的accept()返回并接受这个连接，如果有大量的客户端发起连接而服务器来不及处理，尚未accept的客户端就处于连接等待状态，listen()声明sockfd处于监听状态，并且最多允许有backlog个客户端处于连接待状态，如果接收到更多的连接请求就忽略。listen()成功返回0，失败返回-1。

### accept函数

阻塞等待客户端连接

```c
#include <sys/types.h>   /* See NOTES */

#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);

sockdf:

  socket文件描述符

addr:

  传出参数，返回链接客户端地址信息，含IP地址和端口号

addrlen:

  传入传出参数（值-结果）,传入sizeof(addr)大小，函数返回时返回真正接收到地址结构体的大小

返回值：

  成功返回一个新的socket文件描述符，用于和客户端通信，失败返回-1，设置errno
```

三方握手完成后，服务器调用accept()接受连接，如果服务器调用accept()时还没有客户端的连接请求，就阻塞等待直到有客户端连接上来。**addr是一个传出参数，accept()返回时传出客户端的地址和端口号**。**addrlen参数是一个传入传出参数（value-result argument），传入的是调用者提供的缓冲区addr的长度以避免缓冲区溢出问题，传出的是客户端地址结构体的实际长度（有可能没有占满调用者提供的缓冲区）**。如果给addr参数传NULL，表示不关心客户端的地址。

我们的服务器程序结构是这样的：

```c
while (1) {

  cliaddr_len = sizeof(cliaddr);

  connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);

  n = read(connfd, buf, MAXLINE);

  ......

  close(connfd);

}
```

整个是一个while死循环，每次循环处理一个客户端连接。由于cliaddr_len是传入传出参数，每次调用accept()之前应该重新赋初值。accept()的参数listenfd是先前的监听文件描述符，而accept()的返回值是另外一个文件描述符connfd，之后与客户端之间就通过这个connfd通讯，最后关闭connfd断开连接，而不关闭listenfd，再次回到循环开头listenfd仍然用作accept的参数。accept()成功返回一个文件描述符，出错返回-1。

### connect函数

```c
#include <sys/types.h>         /* See NOTES */

#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

sockdf:

  socket文件描述符

addr:

  传入参数，指定服务器端地址信息，含IP地址和端口号

addrlen:

  传入参数,传入sizeof(addr)大小

返回值：

  成功返回0，失败返回-1，设置errno
```

客户端需要调用connect()连接服务器，connect和bind的参数形式一致，区别在于bind的参数是自己的地址，而connect的参数是对方的地址。connect()成功返回0，出错返回-1。

### TCP通信流程分析

![image-20201027181349847](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20201027181349847.png)

###  client.c

```c
#include "my_head.h"

#define SERV_PORT 9527
char *dst="bye";

/*
                    client
    1.socket
    2.定义初始化要连接的服务器结构地址
    3.connect
    4.write            //向服务器写
    5.read            //读服务器响应

*/

int main(int argc, char *argv[])
{
    int cfd;
    char buf[BUFSIZ];

    //1.建立socket
    cfd = socket(AF_INET, SOCK_STREAM, 0);
    if (cfd == -1)
    {
        sys_err("socket error");
    }

    //建立服务器sockaddr_in
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(SERV_PORT);                                      //跟服务器设定时的port完全一样
    inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr.s_addr);                //跟服务器ip地址一样

    //connect
    int ret = connect(cfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    if (ret != 0)
    {
        sys_err("connect error");
    }
    int count=10;
    while(count--)
    {
        scanf("%s",buf);
        write(cfd,buf,5);
        ret=read(cfd,buf,sizeof(buf));
        printf("%s\n",buf);
        //write(STDOUT_FILENO,buf,ret);
        if(strncmp(buf,dst,strlen(dst))==0)
            break;
    }
    close(cfd);
    return 0;
}
```

### server.c

```c
#include "my_head.h"
#define SERV_PORT 9527
char *dst="end";
/*
                    server
    1.socket
    2.定义初始化服务器结构地址
    3.bind
    4.listen
    5.accept            //传出client地址，与长度
    6.read              //服务器处理client请求

*/
int main(int argc, char *argv[])
{
    int lfd = 0;
    int cfd = 0;      //用于accept返回
    int len;          //存储服务器读取的客户端数据的长度
    char buf[BUFSIZ]; //存储服务器读取的客户端数据
    char client_ip[BUFSIZ];             //存储客户端ip
    lfd = socket(AF_INET, SOCK_STREAM, 0);
    if (lfd == -1)
    {
        sys_err("socket error");
    }

    //定义并初始化服务器地址结构体
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(SERV_PORT);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);

    int ret = bind(lfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    if (ret == -1)
    {
        sys_err("bind error");
    }

    ret = listen(lfd, 128);
    if (ret == -1)
    {
        sys_err("bind error");
    }

    //定义客户端地址
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr_len);
    cfd = accept(lfd, (struct sockaddr *)&client_addr, &client_addr_len);
    if (cfd == -1)
    {
        sys_err("accept error");
    }
    printf("client ip:%s  port:%d\n",
            inet_ntop(AF_INET,&client_addr.sin_addr.s_addr,client_ip,sizeof(client_ip)),
            ntohs(client_addr.sin_port));
    while (1)
    {
        //服务器处理逻辑
        len = read(cfd, buf, sizeof(buf));
        //write(STDOUT_FILENO, buf, len); //终端打印读到的数据
        printf("%s\n",buf);
        if (strncmp(buf, dst,strlen(dst)) == 0)
        {
            strcpy(buf, "bye");
            //write(STDOUT_FILENO, buf, strlen(buf));
            printf("%s\n",buf);
            ret = write(cfd, buf, strlen(buf));
            if (ret == -1)
            {
                sys_err("accept error");
            }
            break;
        }
        //小写转大写
        for (int i = 0; i < len; ++i)
        {
            buf[i] = toupper(buf[i]); //小写转大写
        }
        write(cfd, buf, len);
        if (ret == -1)
        {
            sys_err("accept error");
        }
    }
    close(cfd);
    close(lfd);
}
```

2. 

## 高并发服务器

### 出错处理封装函数

上面的例子不仅功能简单，而且简单到几乎没有什么错误处理，我们知道，系统调用不能保证每次都成功，必须进行出错处理，这样一方面可以保证程序逻辑正常，另一方面可以迅速得到故障信息。

为使错误处理的代码不影响主程序的可读性，我们把与socket相关的一些系统函数加上错误处理代码包装成新的函数，做成一个模块wrap.c：

#### wrap.c

```c
#include "wrap.h"

void perr_exit(const char *s)
{
	perror(s);
	exit(1);
}
int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr)
{
	int n;
	again:
	if ( (n = accept(fd, sa, salenptr)) < 0) {
		if ((errno == ECONNABORTED) || (errno == EINTR))
			goto again;
		else
			perr_exit("accept error");
	}
	return n;
}
int Bind(int fd, const struct sockaddr *sa, socklen_t salen)
{
	int n;
	if ((n = bind(fd, sa, salen)) < 0)
		perr_exit("bind error");
return n;
}
int Connect(int fd, const struct sockaddr *sa, socklen_t salen)
{
	int n;
	if ((n = connect(fd, sa, salen)) < 0)
		perr_exit("connect error");
	return n;
}
int Listen(int fd, int backlog)
{
	int n;
	if ((n = listen(fd, backlog)) < 0)
		perr_exit("listen error");
	return n;
}
int Socket(int family, int type, int protocol)
{
	int n;
	if ( (n = socket(family, type, protocol)) < 0)
		perr_exit("socket error");
	return n;
}
ssize_t Read(int fd, void *ptr, size_t nbytes)
{
	ssize_t n;
again:
	if ( (n = read(fd, ptr, nbytes)) == -1) {
		if (errno == EINTR)
			goto again;
		else
			return -1;
	}
	return n;
}
ssize_t Write(int fd, const void *ptr, size_t nbytes)
{
	ssize_t n;
again:
	if ( (n = write(fd, ptr, nbytes)) == -1) {
		if (errno == EINTR)
			goto again;
		else
			return -1;
	}
	return n;
}
int Close(int fd)
{
int n;
	if ((n = close(fd)) == -1)
		perr_exit("close error");
	return n;
}
ssize_t Readn(int fd, void *vptr, size_t n)
{
	size_t nleft;
	ssize_t nread;
	char *ptr;

	ptr = (char *)vptr;
	nleft = n;

	while (nleft > 0) {
		if ( (nread = read(fd, ptr, nleft)) < 0) {
			if (errno == EINTR)
				nread = 0;
			else
				return -1;
		} else if (nread == 0)
			break;
		nleft -= nread;
		ptr += nread;
	}
	return n - nleft;
}

ssize_t Writen(int fd, const void *vptr, size_t n)
{
	size_t nleft;
	ssize_t nwritten;
	const char *ptr;

	ptr = (const char *)vptr;
	nleft = n;

	while (nleft > 0) {
		if ( (nwritten = write(fd, ptr, nleft)) <= 0) {
			if (nwritten < 0 && errno == EINTR)
				nwritten = 0;
			else
				return -1;
		}
		nleft -= nwritten;
		ptr += nwritten;
	}
	return n;
}
ssize_t my_read(int fd, char *ptr)
{
	static int read_cnt;
	static char *read_ptr;
	static char read_buf[100];

	if (read_cnt <= 0) {
again:
		if ((read_cnt = read(fd, read_buf, sizeof(read_buf))) < 0) {
			if (errno == EINTR)
				goto again;
			return -1;	
		} else if (read_cnt == 0)
			return 0;
		read_ptr = read_buf;
	}
	read_cnt--;
	*ptr = *read_ptr++;
	return 1;
}

ssize_t Readline(int fd, void *vptr, size_t maxlen)
{
	ssize_t n, rc;
	char c, *ptr;
	ptr = (char *)vptr;

	for (n = 1; n < maxlen; n++) {
		if ( (rc = my_read(fd, &c)) == 1) {
			*ptr++ = c;
			if (c == '\n')
				break;
		} else if (rc == 0) {
			*ptr = 0;
			return n - 1;
		} else
			return -1;
	}
	*ptr = 0;
	return n;
}

```



#### wrap.h

``` c
#ifndef __WRAP_H_
#define __WRAP_H_

#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <sys/socket.h>

void perr_exit(const char *s);
int Accept(int fd, struct sockaddr *sa, socklen_t *salenptr);
int Bind(int fd, const struct sockaddr *sa, socklen_t salen);
int Connect(int fd, const struct sockaddr *sa, socklen_t salen);
int Listen(int fd, int backlog);
int Socket(int family, int type, int protocol);
ssize_t Read(int fd, void *ptr, size_t nbytes);
ssize_t Write(int fd, const void *ptr, size_t nbytes);
int Close(int fd);
ssize_t Readn(int fd, void *vptr, size_t n);
ssize_t Writen(int fd, const void *vptr, size_t n);
ssize_t my_read(int fd, char *ptr);
ssize_t Readline(int fd, void *vptr, size_t maxlen);

#endif

```

### 多进程并发服务器

1. socket()							创建套接字lfd

2. bind()                                绑定地址结构 struct sockaddr_in addr

3. listen()

4. while(1){

   ​		cfd=accept();			//接收客户端请求

   ​		pid=fork();

   ​		if(pid==0){

   ​				close(lfd);					//关闭子进程用于建立连接的lfd

   ​				//子进程处理

   ​				read();

   ​				小写转大写

   ​				write();

   ​		}else if(pid>0){

   ​				close(cfd);				//关闭用于与客户端通信的套接字cfd

   ​				注册信号捕捉函数，回收子进程

   ​				continue;					//继续监听

   ​		}								

   }

5. 子进程：close(lfd)

   read()

   小转大

   write()

   父进程：

   close(cfd);

   注册信号捕捉函数：SIGCHLD

   在回调函数中，完成子进程回收

   while(waitpid());

#### **server**.c

```c
/* server.c */
#include <stdio.h>
#include <strings.h>					//bzero()
#include <netinet/in.h>
#include <arpa/inet.h>
#include <signal.h>
#include <sys/wait.h>
#include <sys/types.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 800

//回收子进程
void do_sigchild(int num)
{
	while ((waitpid(0, NULL, WNOHANG)) > 0)
		;
}
int main(void)
{
	struct sockaddr_in servaddr, cliaddr;
	socklen_t cliaddr_len;
	int listenfd, connfd;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int i, n;
	pid_t pid;

	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

    //将指定空间置0
	bzero(&servaddr, sizeof(servaddr));			
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	Bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	Listen(listenfd, 20);

	printf("Accepting connections ...\n");
	while (1) {
		cliaddr_len = sizeof(cliaddr);
		connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);

		pid = fork();
		if (pid == 0) {
            
            //子进程
            
			Close(listenfd);
			while (1) {
				n = Read(connfd, buf, MAXLINE);
				if (n == 0) {
					printf("the other side has been closed.\n");
					break;
				}
				printf("received from %s at PORT %d\n",
						inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
						ntohs(cliaddr.sin_port));
				for (i = 0; i < n; i++)
					buf[i] = toupper(buf[i]);
					Write(connfd, buf, n);
			}
			Close(connfd);
			return 0;
		} else if (pid > 0) {
            //在父进程中回收子进程
            //回收子进程
			struct sigaction newact;
			newact.sa_handler = do_sigchild;
			sigemptyset(&newact.sa_mask);
			newact.sa_flags = 0;
			sigaction(SIGCHLD, &newact, NULL);
            
			Close(connfd);
		} else
			perr_exit("fork");
	}
	Close(listenfd);
	return 0;
}

```

#### **client.c**

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, n;

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	Connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(sockfd, buf, strlen(buf));
		n = Read(sockfd, buf, MAXLINE);
		if (n == 0) {
			printf("the other side has been closed.\n");
			break;
		} else
			Write(STDOUT_FILENO, buf, n);
	}
	Close(sockfd);
	return 0;
}

```

### 多线程并发服务器

1. socket();						创建监听套接字lfd

2. bind()                              绑定地址结构struct sockaddr_in addr;

3. listen()

4. while(1){

   cfd=accept(lfd,);

   pthread_create(&tid,NULL,tfn,NULL);

   pthread_detach(tid);				//父子线程分离,相当于回收子线程

   //或者新建一个线程，回收子线程

   }

5. 子线程
   void *tfn(void *arg){

   ​		close(lfd)

   ​		read();

   ​		小转大

   ​		write()

   }

在使用线程模型开发服务器时需考虑以下问题：

1. 调整进程内最大文件描述符上限

2. 线程如有共享数据，考虑线程同步

3. 服务于客户端线程退出时，退出处理。（退出值，分离态）

4.  系统负载，随着链接客户端增加，导致其它线程不能及时得到CPU

#### server.c

```c
/* server.c */
#include <stdio.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <pthread.h>

#include "wrap.h"
#define MAXLINE 80
#define SERV_PORT 6666

struct s_info {
	struct sockaddr_in cliaddr;
	int connfd;
};
void *do_work(void *arg)
{
	int n,i;
	struct s_info *ts = (struct s_info*)arg;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	/* 可以在创建线程前设置线程创建属性,设为分离态,哪种效率高内？ */
	pthread_detach(pthread_self());		//设置子线程分离，防止僵尸线程
	while (1) {
		n = Read(ts->connfd, buf, MAXLINE);
		if (n == 0) {
			printf("the other side has been closed.\n");
			break;
		}
		printf("received from %s at PORT %d\n",
				inet_ntop(AF_INET, &(*ts).cliaddr.sin_addr, str, sizeof(str)),
				ntohs((*ts).cliaddr.sin_port));
		for (i = 0; i < n; i++)
			buf[i] = toupper(buf[i]);
		Write(ts->connfd, buf, n);			//回写到客户端
	}
	Close(ts->connfd);
}

int main(void)
{
	struct sockaddr_in servaddr, cliaddr;
	socklen_t cliaddr_len;
	int listenfd, connfd;
	int i = 0;
	pthread_t tid;
	struct s_info ts[256];

	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	Bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
	Listen(listenfd, 20);

	printf("Accepting connections ...\n");
	while (1) {
		cliaddr_len = sizeof(cliaddr);
		connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
		ts[i].cliaddr = cliaddr;
		ts[i].connfd = connfd;
		/* 达到线程最大数时，pthread_create出错处理, 增加服务器稳定性 */
		pthread_create(&tid, NULL, do_work, (void*)&ts[i]);
		i++;
	}
	return 0;
}

```

#### client.c

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include "wrap.h"
#define MAXLINE 80
#define SERV_PORT 6666
int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, n;

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	Connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(sockfd, buf, strlen(buf));
		n = Read(sockfd, buf, MAXLINE);
		if (n == 0)
			printf("the other side has been closed.\n");
		else
			Write(STDOUT_FILENO, buf, n);
	}
	Close(sockfd);
	return 0;
}
```

### 多路I/O转接服务器

#### select

1. select能监听的文件描述符个数受限于FD_SETSIZE,一般为1024，单纯改变进程打开的文件描述符个数并不能改变select监听文件个数

2. 解决1024以下客户端时使用select是很合适的，但如果链接客户端过多，select采用的是轮询模型，会大大降低服务器响应效率，不应在select上投入更多精力

```c
#include <sys/select.h>
/* According to earlier standards */
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int nfds, fd_set *readfds, fd_set *writefds,
			fd_set *exceptfds, struct timeval *timeout);

	nfds: 		监控的文件描述符集里最大文件描述符加1，因为此参数会告诉内核检测前多少个文件描述符的状态
	readfds：	监控有读数据到达文件描述符集合，传入传出参数
	writefds：	监控写数据到达文件描述符集合，传入传出参数
	exceptfds：	监控异常发生达文件描述符集合,如带外数据到达异常，传入传出参数
	timeout：	定时阻塞监控时间，3种情况
				1.NULL，永远等下去
				2.设置timeval，等待固定时间
				3.设置timeval里时间均为0，检查描述字后立即返回，轮询
	struct timeval {
		long tv_sec; /* seconds */
		long tv_usec; /* microseconds */
	};
	void FD_CLR(int fd, fd_set *set); 	//把文件描述符集合里fd清0
	int FD_ISSET(int fd, fd_set *set); 	//测试文件描述符集合里fd是否置1
	void FD_SET(int fd, fd_set *set); 	//把文件描述符集合里fd位置1
	void FD_ZERO(fd_set *set); 			//把文件描述符集合里所有位清0

返回值：   >0:所有监听几个（3个）中，满足对应事件的总数
    	 =0：没有满足监听条件的文件描述符
    	 <0:errno
```

思路分析：

​	lfd=socket();							//创建套接字

​	bind();										//绑定地址结构

​	listen();										//设置监听上限

​	fd_set reset,allset;							//创建读监听集合

​	FD_ZERO(&allest)						将读监听集合清空

​	FD_SET(lfd,&allset)					将lfd添加至读集合中

​	while(1){

​	rset=allset;												保存监听集合

​	ret=select(lfd+1,&rset,NULL,NULL,NULL);     监听文件描述符对应事件

​	if(ret>0) {																有监听的描述符满足对应事件

​			if（FD_ISSET（lfd,&rest））{

​					//1在，0不在

​					cfd=accept();								建立连接，返回用于通信的文件描述符

​					FD_SET(cfd,&rset)						添加到监听通信描述符集合中

}		

​		for(i=lfd+1;i<=最大文件描述符;i++){

​		FD_ISSET（lfd,&i）;       有read,write事件

​		read()

​		小转大

​		write();

}

}

}

​	

##### server.c

```c
/* server.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	int i, maxi, maxfd, listenfd, connfd, sockfd;
	int nready, client[FD_SETSIZE]; 	/* FD_SETSIZE 默认为 1024 */
	ssize_t n;
	fd_set rset, allset;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN]; 			/* #define INET_ADDRSTRLEN 16 */
	socklen_t cliaddr_len;
	struct sockaddr_in cliaddr, servaddr;

	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port = htons(SERV_PORT);

Bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

Listen(listenfd, 20); 		/* 默认最大128 */

maxfd = listenfd; 			/* 初始化 */
maxi = -1;					/* client[]的下标 */

for (i = 0; i < FD_SETSIZE; i++)
	client[i] = -1; 		/* 用-1初始化client[] */

FD_ZERO(&allset);
FD_SET(listenfd, &allset); /* 构造select监控文件描述符集 */

for ( ; ; ) {
	rset = allset; 			/* 每次循环时都从新设置select监控信号集 */
	nready = select(maxfd+1, &rset, NULL, NULL, NULL);

	if (nready < 0)
		perr_exit("select error");
	if (FD_ISSET(listenfd, &rset)) { /* new client connection */
		cliaddr_len = sizeof(cliaddr);
		connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
		printf("received from %s at PORT %d\n",
				inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
				ntohs(cliaddr.sin_port));
		for (i = 0; i < FD_SETSIZE; i++) {
			if (client[i] < 0) {
				client[i] = connfd; /* 保存accept返回的文件描述符到client[]里 */
				break;
			}
		}
		/* 达到select能监控的文件个数上限 1024 */
		if (i == FD_SETSIZE) {
			fputs("too many clients\n", stderr);
			exit(1);
		}

		FD_SET(connfd, &allset); 	/* 添加一个新的文件描述符到监控信号集里 */
		if (connfd > maxfd)
			maxfd = connfd; 		/* select第一个参数需要 */
		if (i > maxi)
			maxi = i; 				/* 更新client[]最大下标值 */

		if (--nready == 0)
			continue; 				/* 如果没有更多的就绪文件描述符继续回到上面select阻塞监听,
										负责处理未处理完的就绪文件描述符 */
		}
		for (i = 0; i <= maxi; i++) { 	/* 检测哪个clients 有数据就绪 */
			if ( (sockfd = client[i]) < 0)
				continue;
			if (FD_ISSET(sockfd, &rset)) {
				if ( (n = Read(sockfd, buf, MAXLINE)) == 0) {
					Close(sockfd);		/* 当client关闭链接时，服务器端也关闭对应链接 */
					FD_CLR(sockfd, &allset); /* 解除select监控此文件描述符 */
					client[i] = -1;
				} else {
					int j;
					for (j = 0; j < n; j++)
						buf[j] = toupper(buf[j]);
					Write(sockfd, buf, n);
				}
				if (--nready == 0)
					break;
			}
		}
	}
	close(listenfd);
	return 0;
}
```

##### client.c

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, n;

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	Connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(sockfd, buf, strlen(buf));
		n = Read(sockfd, buf, MAXLINE);
		if (n == 0)
			printf("the other side has been closed.\n");
		else
			Write(STDOUT_FILENO, buf, n);
	}
	Close(sockfd);
	return 0;
}
```

##### 优缺点

​	缺点：监听上限受文件描述符限制。最大是1024.

​				无法直接检测满足条件的文件描述符。提高了编码难度。

​	优点：跨平台。win、linux、unix

#### poll

```c
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
	struct pollfd {
		int fd; /* 文件描述符 */
		short events; /* 监控的事件 */  取值：POLLIN、POLLOUT、POLLERR
		short revents; /* 监控事件中满足条件返回的事件 */传入时给0.如果满足对应事件的话，返回非0--> POLLIN、POLLOUT、POLLERR
	};
	POLLIN			普通或带外优先数据可读,即POLLRDNORM | POLLRDBAND
	POLLRDNORM		数据可读
	POLLRDBAND		优先级带数据可读
	POLLPRI 		高优先级可读数据
	POLLOUT			普通或带外数据可写
	POLLWRNORM		数据可写
	POLLWRBAND		优先级带数据可写
	POLLERR 		发生错误
	POLLHUP 		发生挂起
	POLLNVAL 		描述字不是一个打开的文件

    fds				监听的文件描述符[数组]
	nfds 			监控数组中有多少文件描述符需要被监控

	timeout 		毫秒级等待
		-1：阻塞等，#define INFTIM -1 				Linux中没有定义此宏
		0：立即返回，不阻塞进程
		>0：等待指定毫秒数，如当前系统时间精度不够毫秒，向上取值
       
   返回值：返回满足对应监听事件的文件描述符  总个数

```

##### server.c

```c
/* server.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <poll.h>
#include <errno.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666
#define OPEN_MAX 1024

int main(int argc, char *argv[])
{
	int i, j, maxi, listenfd, connfd, sockfd;
	int nready;
	ssize_t n;
	char buf[MAXLINE], str[INET_ADDRSTRLEN];
	socklen_t clilen;
	struct pollfd client[OPEN_MAX];
	struct sockaddr_in cliaddr, servaddr;
listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	Bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	Listen(listenfd, 20);

	client[0].fd = listenfd;
	client[0].events = POLLRDNORM;/* listenfd监听普通读事件 */

	for (i = 1; i < OPEN_MAX; i++)
		client[i].fd = -1; 	/* 用-1初始化client[]里剩下元素 */
	maxi = 0; 			/* client[]数组有效元素中最大元素下标 */

	for ( ; ; ) {
		nready = poll(client, maxi+1, -1); 		/* 阻塞 */
		if (client[0].revents & POLLRDNORM) {    /* 有客户端链接请求 */
			clilen = sizeof(cliaddr);
			connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &clilen);
			printf("received from %s at PORT %d\n",
					inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
					ntohs(cliaddr.sin_port));
			for (i = 1; i < OPEN_MAX; i++) {
				if (client[i].fd < 0) {
					client[i].fd = connfd; 	/* 找到client[]中空闲的位置，存放accept返回的connfd */
					break;
				}
			}

			if (i == OPEN_MAX)
				perr_exit("too many clients");

			client[i].events = POLLRDNORM; 		/* 设置刚刚返回的connfd，监控读事件 */
			if (i > maxi)
				maxi = i; 						/* 更新client[]中最大元素下标 */
			if (--nready <= 0)
				continue; 						/* 没有更多就绪事件时,继续回到poll阻塞 */
		}
		for (i = 1; i <= maxi; i++) { 			/* 检测client[] */
			if ((sockfd = client[i].fd) < 0)
				continue;
			if (client[i].revents & (POLLRDNORM | POLLERR)) {
				if ((n = Read(sockfd, buf, MAXLINE)) < 0) {
					if (errno == ECONNRESET) { /* 当收到 RST标志时 */
	/* connection reset by client */
						printf("client[%d] aborted connection\n", i);
						Close(sockfd);
						client[i].fd = -1;
					} else {
						perr_exit("read error");
					}
				} else /* server.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <poll.h>
#include <errno.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666
#define OPEN_MAX 1024

int main(int argc, char *argv[])
{
	int i, j, maxi, listenfd, connfd, sockfd;
	int nready;
	ssize_t n;
	char buf[MAXLINE], str[INET_ADDRSTRLEN];
	socklen_t clilen;
	struct pollfd client[OPEN_MAX];
	struct sockaddr_in cliaddr, servaddr;
listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	//servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

    int dst;
    inet_pton(AF_INET, "192.168.102.1", (void *)&dst);
    servaddr.sin_addr.s_addr = dst;
    
	servaddr.sin_port = htons(SERV_PORT);

	Bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	Listen(listenfd, 20);

	client[0].fd = listenfd;
	client[0].events = POLLIN;/* listenfd监听普通读事件 */

	for (i = 1; i < OPEN_MAX; i++)
		client[i].fd = -1; 	/* 用-1初始化client[]里剩下元素 */
	maxi = 0; 			/* client[]数组有效元素中最大元素下标 */

	for ( ; ; ) {
		nready = poll(client, maxi+1, -1); 		/* 阻塞 */
		if (client[0].revents & POLLIN) {    /* 有客户端链接请求 */
			clilen = sizeof(cliaddr);
			connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &clilen);
			printf("received from %s at PORT %d\n",
					inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
					ntohs(cliaddr.sin_port));
			for (i = 1; i < OPEN_MAX; i++) {
				if (client[i].fd < 0) {
					client[i].fd = connfd; 	/* 找到client[]中空闲的位置，存放accept返回的connfd */
					break;
				}
			}

			if (i == OPEN_MAX)
				perr_exit("too many clients");

			client[i].events = POLLIN; 		/* 设置刚刚返回的connfd，监控读事件 */
			if (i > maxi)
				maxi = i; 						/* 更新client[]中最大元素下标 */
			if (--nready <= 0)
				continue; 						/* 没有更多就绪事件时,继续回到poll阻塞 */
		}
		for (i = 1; i <= maxi; i++) { 			/* 检测client[] */
			if ((sockfd = client[i].fd) < 0)
				continue;
			if (client[i].revents & POLLIN) {
				if ((n = Read(sockfd, buf, MAXLINE)) < 0) {
					if (errno == ECONNRESET) { /* 当收到 RST标志时 */
	/* connection reset by client */
						printf("client[%d] aborted connection\n", i);
						Close(sockfd);
						client[i].fd = -1;
					} else {
						perr_exit("read error");
					}
				} else if (n == 0) {
					/* connection closed by client */
					printf("client[%d] closed connection\n", i);
					Close(sockfd);
					client[i].fd = -1;
				} else {
					for (j = 0; j < n; j++)
						buf[j] = toupper(buf[j]);
						Writen(sockfd, buf, n);
				}
				if (--nready <= 0)
					break; 				/* no more readable descriptors */
			}
		}
	}
	return 0;
}if (n == 0) {
					/* connection closed by client */
					printf("client[%d] closed connection\n", i);
					Close(sockfd);
					client[i].fd = -1;
				} else {
					for (j = 0; j < n; j++)
						buf[j] = toupper(buf[j]);
						Writen(sockfd, buf, n);
				}
				if (--nready <= 0)
					break; 				/* no more readable descriptors */
			}
		}
	}
	return 0;
}

```

##### client.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <ctype.h>

#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, n;

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "192.168.102.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	Connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(sockfd, buf, strlen(buf));
		n = Read(sockfd, buf, MAXLINE);
		if (n == 0)
			printf("the other side has been closed.\n");
		else
			Write(STDOUT_FILENO, buf, n);
	}
	Close(sockfd);
	return 0;
}
```

##### read函数的返回值：

​		>0:实际读到的字节数

​		=0：socket中，表示对端关闭。需要close

​		-1：	如果errno  ==  EINTR   被异常中断 。需要重启

​					如果errno  ==  EAGIN 或 EWOULDBLOCK 以非阻塞方式读数据，但是没有数据。需要再次读

​					如果errno  ==  ECONNRESET   说明连接被重置。需要close(),移除监听队列。

##### 优缺点

​	自带数组结构。可以将监听事件集合和返回事件集合分离

​	拓展监听上限。超出1024的限制。

​	不能跨平台，只能在linux。

​	无法直接定位满足监听事件的文件描述符。

##### 突破1024限制

可以使用cat命令查看一个进程可以打开的socket描述符上限。

当前计算机所能打开的最大文件个数。受硬件影响。

> cat /proc/sys/fs/file-max

当前用户下的进程，默认打开文件描述符个数。缺省为1024

> ulimit  -a				查看
>
> ulimit -n 10000      修改

如有需要，也可以通过修改配置文件的方式修改该上限值。

  sudo vi /etc/security/limits.conf

  在文件尾部写入以下配置,soft软限制，hard硬限制。如下图所示。

> soft nofile 65536
>
> hard nofile 100000

注意：soft值不能超过hard值

#### epoll

​		epoll是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率，因为它会复用文件描述符集合来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。

​		目前epoll是linux大规模并发网络程序中的热门首选模型。

​		epoll除了提供select/poll那种IO事件的电平触发（Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

##### 基础API

1.创建一个epoll句柄，参数size用来告诉内核监听的文件描述符的个数，跟内存大小有关。

```c
  #include <sys/epoll.h>

  int epoll_create(int size)  

 	size：监听数目；创建的红黑树的监听结点的数量

	返回值:成功指向新创建的红黑树的根节点的fd。
          失败返回-1，errno
```

2.控制某个epoll监控的文件描述符上的事件：注册、修改、删除。

  

```c
#include <sys/epoll.h>

  int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)

  epfd： 为epoll_creat的句柄，即epoll_create函数的返回值

  op：  对监听红黑树所做的操作，用3个宏来表示：

      		EPOLL_CTL_ADD (注册新的fd到epfd)，

      		EPOLL_CTL_MOD (修改已经注册的fd的监听事件)，

      		EPOLL_CTL_DEL (从epfd删除一个fd)；

 fd:待监听的fd;如：连接事件，读写事件

 event： 告诉内核需要监听的事件

    struct epoll_event {

      __uint32_t events; /* Epoll events */

      epoll_data_t data; /* User data variable */

    };
	
	__uint32_t events:

	EPOLLIN ： 表示对应的文件描述符可以读（包括对端SOCKET正常关闭）

    EPOLLOUT： 表示对应的文件描述符可以写

    EPOLLPRI： 表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）

    EPOLLERR： 表示对应的文件描述符发生错误

    EPOLLHUP： 表示对应的文件描述符被挂断；

    EPOLLET：  将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)而言的

    EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里

    typedef union epoll_data {

      void *ptr;

      int fd;				//对应监听事件的fd

      uint32_t u32;			//不用

      uint64_t u64;			//不用

    } epoll_data_t;

返回值：成功：0;失败：-1，并设置errno

```

3. 等待所监控文件描述符上有事件的产生，类似于select()调用。

```c
  #include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)
    
epfd：epoll_create的返回值
      
events：  用来存内核得到事件的集合，可以简单看作成一个数组；传出参数：传出满足监听条件的那些结构体fd结构体。

maxevents： 告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，

timeout：  是超时时间

  -1： 阻塞

  0： 立即返回，非阻塞

  >0： 指定毫秒

返回值： 成功返回有多少文件描述符就绪，时间到时返回0，出错返回-1
    	>0:返回满足监听的总个数
```

##### server

listenfd:建立连接的读事件

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <errno.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666
#define OPEN_MAX 1024

int main(int argc, char *argv[])
{
	int i, j, maxi, listenfd, connfd, sockfd;
	int nready, efd, res;
	ssize_t n;
	char buf[MAXLINE], str[INET_ADDRSTRLEN];
	socklen_t clilen;
	int client[OPEN_MAX];
	struct sockaddr_in cliaddr, servaddr;
    //tep:epoll_ctl参数
    //ep[]:epoll_wait参数
	struct epoll_event tep, ep[OPEN_MAX];

	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	Bind(listenfd, (struct sockaddr *) &servaddr, sizeof(servaddr));

	Listen(listenfd, 20);

	for (i = 0; i < OPEN_MAX; i++)
		client[i] = -1;
	maxi = -1;

    //创建epoll模型，efd指向红黑树根节点
	efd = epoll_create(OPEN_MAX);
	if (efd == -1)
		perr_exit("epoll_create");

    //指定lfd的监听事件为“读”
	tep.events = EPOLLIN; 
    tep.data.fd = listenfd;

    //操作监听红黑树
    //将lfd及对应的结构体设置到树上，efd可找到该树
	res = epoll_ctl(efd, EPOLL_CTL_ADD, listenfd, &tep);
	if (res == -1)
		perr_exit("epoll_ctl");

	while (1) {
        //epoll为server阻塞监听事件，ep为struct epoll_event类型数组，OPEN_MAX为数组容量，-1代表永久阻塞
		nready = epoll_wait(efd, ep, OPEN_MAX, -1); /* 阻塞监听 */
		if (nready == -1)
			perr_exit("epoll_wait");

		for (i = 0; i < nready; i++) {
            //如果不是读事件，则继续循环
			if (!(ep[i].events & EPOLLIN))
				continue;
            //判断满足的事件的fd是不是lfd
			if (ep[i].data.fd == listenfd) {
				clilen = sizeof(cliaddr);
                //建立连接
				connfd = Accept(listenfd, (struct sockaddr *)&cliaddr, &clilen);
				printf("received from %s at PORT %d\n", 
						inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)), 
						ntohs(cliaddr.sin_port));
                /*
				for (j = 0; j < OPEN_MAX; j++) {
					if (client[j] < 0) {
						client[j] = connfd; // save descriptor 
						break;
					}
				}
				

				if (j == OPEN_MAX)
					perr_exit("too many clients");
				if (j > maxi)
					maxi = j; // max index in client[] array 				*/
				tep.events = EPOLLIN; 
				tep.data.fd = connfd;
res = epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &tep);
				if (res == -1)
					perr_exit("epoll_ctl");
			} else {
                //数据读写事件
				sockfd = ep[i].data.fd;
				n = Read(sockfd, buf, MAXLINE);
                //读到0，说明客户端关闭连接
				if (n == 0) {
                    /*
					for (j = 0; j <= maxi; j++) {
						if (client[j] == sockfd) {
							client[j] = -1;
							break;
						}
					}
					*/
                    //将该文件描述符从红黑树摘除
					res = epoll_ctl(efd, EPOLL_CTL_DEL, sockfd, NULL);
					if (res == -1)
						perr_exit("epoll_ctl");

					Close(sockfd);
					printf("client[%d] closed connection\n", j);
				} else {
					for (j = 0; j < n; j++)
						buf[j] = toupper(buf[j]);
					Writen(sockfd, buf, n);
				}
			}
		}
	}
	close(listenfd);
	close(efd);
	return 0;
}

```

##### client

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include "wrap.h"

#define MAXLINE 80
#define SERV_PORT 6666

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
int sockfd, n;

	sockfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	Connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (fgets(buf, MAXLINE, stdin) != NULL) {
		Write(sockfd, buf, strlen(buf));
		n = Read(sockfd, buf, MAXLINE);
		if (n == 0)
			printf("the other side has been closed.\n");
		else
			Write(STDOUT_FILENO, buf, n);
	}

	Close(sockfd);
	return 0;
}
```

#### epoll进阶

##### 事件模型

EPOLL事件有两种模型：

Edge Triggered (ET) **边缘**触发只有数据到达缓冲区才触发，不管缓存区中是否还有数据。

Level Triggered (LT) **水平**触发只要缓冲区有数据都会触发。

思考如下步骤：

1. 假定我们已经把一个用来从管道中读取数据的文件描述符(RFD)添加到epoll描述符。

2. 管道的另一端写入了2KB的数据

3. 调用epoll_wait，并且它会返回RFD，说明它已经准备好读取操作

4. 读取1KB的数据

5. 调用epoll_wait……

在这个过程中，有两种工作模式：

##### ET模式

ET模式即Edge Triggered工作模式。

如果我们在第1步将RFD添加到epoll描述符的时候使用了EPOLLET标志，那么在第5步调用epoll_wait之后将有可能会挂起，因为剩余的数据还存在于文件的输入缓冲区内，而且数据发出端还在等待一个针对已经发出数据的反馈信息。只有在监视的文件句柄上发生了某个事件的时候 ET 工作模式才会汇报事件。因此在第5步的时候，调用者可能会放弃等待仍在存在于文件输入缓冲区内的剩余数据。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。最好以下面的方式调用ET模式的epoll接口，在后面会介绍避免可能的缺陷。

​	1)   基于非阻塞文件句柄

​	2)   只有当read或者write返回EAGAIN(非阻塞读，暂时无数据)时才需要挂起、等待。但这并不是说每次read时都需要循环读，直到读到产生一个EAGAIN才认为此次事件处理完成，当read返回的读到的数据长度小于请求的数据长度时，就可以确定此时缓冲中已没有数据了，也就可以认为此事读事件已处理完成。

##### LT模式

LT模式即Level Triggered工作模式。

与ET模式不同的是，以LT方式调用epoll接口的时候，它就相当于一个速度比较快的poll，无论后面的数据是否被使用。

LT(level triggered)：LT是默认的工作方式，并且同时支持block和no-block socket。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点。传统的select/poll都是这种模型的代表。

ET(edge-triggered)：ET是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知。请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once).

##### 实例一

基于管道epoll ET触发模式

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/epoll.h>
#include <errno.h>
#include <unistd.h>

#define MAXLINE 10

int main(int argc, char *argv[])
{
	int efd, i;
	int pfd[2];
	pid_t pid;
	char buf[MAXLINE], ch = 'a';

	pipe(pfd);
	pid = fork();
if (pid == 0) {
    	//子进程   写事件
		close(pfd[0]);
		while (1) {
            //写入aaaa\n
			for (i = 0; i < MAXLINE/2; i++)
				buf[i] = ch;
			buf[i-1] = '\n';
			ch++;
            
			//写入bbbb\n
			for (; i < MAXLINE; i++)
				buf[i] = ch;
			buf[i-1] = '\n';
			ch++;
			//buf:aaaa\nbbbb\n
			write(pfd[1], buf, sizeof(buf));
			sleep(2);
		}
		close(pfd[1]);
	} else if (pid > 0) {
    	//父进程  读事件
		struct epoll_event event;	//ctl
		struct epoll_event resevent[10];		//wait
		int res, len;
		close(pfd[1]);

		efd = epoll_create(10);
		/* event.events = EPOLLIN; */	//LT水平触发（默认）
		event.events = EPOLLIN | EPOLLET;		/* ET 边沿触发 ，默认是水平触发 */
		event.data.fd = pfd[0];
		epoll_ctl(efd, EPOLL_CTL_ADD, pfd[0], &event);

		while (1) {
			res = epoll_wait(efd, resevent, 10, -1);
			printf("res %d\n", res);
            //判断是否是读事件
			if (resevent[0].data.fd == pfd[0]) {
				len = read(pfd[0], buf, MAXLINE/2);
				write(STDOUT_FILENO, buf, len);
			}
		}
		close(pfd[0]);
		close(efd);
	} else {
		perror("fork");
		exit(-1);
	}
	return 0;
}
```

##### 实例二

基于网络C/S模型的epoll ET触发模式（一般不用阻塞的ET）

###### server

```c
/* server.c */
#include <stdio.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <signal.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/epoll.h>
#include <unistd.h>

#define MAXLINE 10
#define SERV_PORT 8080

int main(void)
{
	struct sockaddr_in servaddr, cliaddr;
	socklen_t cliaddr_len;
	int listenfd, connfd;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int i, efd;

	listenfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	listen(listenfd, 20);

	struct epoll_event event;
	struct epoll_event resevent[10];
	int res, len;
	efd = epoll_create(10);
	event.events = EPOLLIN | EPOLLET;		/* ET 边沿触发 ，默认是水平触发 */

	printf("Accepting connections ...\n");
cliaddr_len = sizeof(cliaddr);
	connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
	printf("received from %s at PORT %d\n",
			inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
			ntohs(cliaddr.sin_port));

	event.data.fd = connfd;
	epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &event);

	while (1) {
		res = epoll_wait(efd, resevent, 10, -1);
		printf("res %d\n", res);
		if (resevent[0].data.fd == connfd) {
			len = read(connfd, buf, MAXLINE/2);
			write(STDOUT_FILENO, buf, len);
		}
	}
	return 0;
}
```

###### client

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>

#define MAXLINE 10
#define SERV_PORT 8080

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, i;
	char ch = 'a';

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (1) {
for (i = 0; i < MAXLINE/2; i++)
			buf[i] = ch;
		buf[i-1] = '\n';
		ch++;

		for (; i < MAXLINE; i++)
			buf[i] = ch;
		buf[i-1] = '\n';
		ch++;

		write(sockfd, buf, sizeof(buf));
		sleep(10);
	}
	Close(sockfd);
	return 0;
}

```

##### 实例三

基于网络C/S非阻塞模型的epoll ET触发模式

###### server

```c
/* server.c */
#include <stdio.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/epoll.h>
#include <unistd.h>
#include <fcntl.h>

#define MAXLINE 10
#define SERV_PORT 8080

int main(void)
{
	struct sockaddr_in servaddr, cliaddr;
	socklen_t cliaddr_len;
	int listenfd, connfd;
	char buf[MAXLINE];
	char str[INET_ADDRSTRLEN];
	int i, efd, flag;

	listenfd = socket(AF_INET, SOCK_STREAM, 0);
bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);

	bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	listen(listenfd, 20);

	struct epoll_event event;			//epoll_ctl
	struct epoll_event resevent[10];	//epoll_wait
	int res, len;
	efd = epoll_create(10);
	/* event.events = EPOLLIN; */
	event.events = EPOLLIN | EPOLLET;		/* ET 边沿触发 ，默认是水平触发 */

	printf("Accepting connections ...\n");
	cliaddr_len = sizeof(cliaddr);
	connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &cliaddr_len);
	printf("received from %s at PORT %d\n",
			inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
			ntohs(cliaddr.sin_port));

    //设置非阻塞
    //修改connfd为非阻塞
	flag = fcntl(connfd, F_GETFL);
	flag |= O_NONBLOCK;
	fcntl(connfd, F_SETFL, flag);
	event.data.fd = connfd;
	epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &event);

	while (1) {
		printf("epoll_wait begin\n");
		res = epoll_wait(efd, resevent, 10, -1);
		printf("epoll_wait end res %d\n", res);

		if (resevent[0].data.fd == connfd) {
			while ((len = read(connfd, buf, MAXLINE/2)) > 0)
				write(STDOUT_FILENO, buf, len);
		}
	}
	return 0;
}
```

###### client

```c
/* client.c */
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>

#define MAXLINE 10
#define SERV_PORT 8080

int main(int argc, char *argv[])
{
	struct sockaddr_in servaddr;
	char buf[MAXLINE];
	int sockfd, i;
	char ch = 'a';

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
	servaddr.sin_port = htons(SERV_PORT);

	connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));

	while (1) {
		for (i = 0; i < MAXLINE/2; i++)
			buf[i] = ch;
		buf[i-1] = '\n';
		ch++;

		for (; i < MAXLINE; i++)
			buf[i] = ch;
		buf[i-1] = '\n';
		ch++;

		write(sockfd, buf, sizeof(buf));
		sleep(10);
	}
	Close(sockfd);
	return 0;
}
```

epoll的ET模式是高效的模式，只支持非阻塞模式。需要忙轮询，即循环。

##### 优点：

​		高效，能突破1024文件描述符的限制。

##### 缺点：

​		不能跨平台

#### epoll反应堆模型

epoll ET 模式+非阻塞+void *ptr（回调函数）

原来的方法：socket、bind、listen、epoll_create创建监听红黑树 ---返回epfd ----epoll_ctl()向树上添加一个监听fd ---- while(1)----epoll_wait 监听 ---- 对应监听fd有事件产生 ---返回 监听满足数组。 ----判断返回数组元素 ---lfd满足---Accept ---cfd 满足---- read() ---小转大 ---- write回去

反应堆：不但要监听cfd的读事件，还要监听cfd的写事件。

socket、bind、listen、epoll_create创建监听红黑树 ---返回epfd ----epoll_ctl()向树上添加一个监听fd ---- while(1)----epoll_wait 监听 ---- 对应监听fd有事件产生 ---返回 监听满足数组。 ----判断返回数组元素 ---lfd满足---Accept ---cfd 满足---- read() ---小转大 ---cfd从监听红黑树上摘下---EPOLLOUT---- 回调函数----epoll_ctl()----- EPOLL_CTL_ADD重新放到红黑树上监听写事件——----epoll_ctl()监听cfd的写事件----等待epoll_wait返回----说明cfd可写---- write回去---cfd从监听红黑树上摘下---EPOLLIN----epoll_ctl()----- EPOLL_CTL_ADD重新放到红黑树上监听读事件---epoll_wait监听

原来没出错的原因是原来环境简单，一旦对端滑动窗口半关闭或者对端滑动窗口已满，则原来的方法会出错。
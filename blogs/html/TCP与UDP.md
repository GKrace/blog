# TCP与UDP

## 一.TCP/IP网络模型

* 应用层：负责向用户提供应用程序，如HTTP，FTP，Telnet，DNS，SMTP等
* 传输层：负责对报文进行分组和重组，并以TCP或UDP协议格式封装报文
* 网络层：负责路由以及把分组报文发送给网络或者主机
* 链路层：负责封装和解封装IP报文，发送和接受ARP/RARP报文等
* 物理层：CPU，网卡等物料层

## 二.UDP

UDP协议全称是用户数据报协议，在网络中它与TCP协议一样用于处理数据包，是一种无连接的协议。在OSI模型中，在第四层--传输层，处于IP协议的上一层，UDP有不提供数据包分组，组装和不能对数据包进行排序的缺点，也就是说，当报文发送之后，是无法得知是否安全完整送达的。  
它有以下几个特点：

### 1.面向无连接

UDP不需要和TCP一样在发送数据前进行三次握手建立连接。想发送数据就可以发送,并且也只是数据报文的搬运工，不会对数据报文进行任何拆分和拼接操作

* 在发送端：引用层将数据传递给传输层的UDP协议，UDP只会给数据增加一个UDP头标识下是UDP协议，之后就传递给网络层了
* 在接收端：网络层将数据传递给传输层，UDP只去除IP报头就传递给应用层，不做任何拼接操作

### 2.有单播，多播，广播的功能

UDP不止支持一对一的传输方式，同样支持一对多，多对多，多对一的方式，也就是说UDP提供了单播，多播，广播的功能

### 3.UDP是面对报文的

发送方的UDP对应用程序交下来的报文，在添加首部后就向下交付IP层，UDP对引用层交下来的报文既不合并，也不拆分，而是保留这些报文的边界，因此，应用程序必须选择合适大小的报文

### 4.不可靠性

* 首先不可靠性体现在无连接上，通信都不需要建立连接，想发就发  
* 收到什么数据就传递什么数据，并且也不会备份数据，发送数据也不会关心对方是否已经正确接收到数据了  
* 由于网络环境时好时坏，但是UDP因为没有拥塞控制，一直会以恒定的速度发送数据。即使网络条件不好，也不会对发送速率进行调整。这样实现的弊端就是在网络条件不好的情况下可能会导致丢包，但是有点很明显，在某些实时性要求很高的场景（电话会议，直播）就需要使用UDP而不是TCP

### 5.头部开销小，传输数据报文时是很高效的

UDP头部包含了以下几个数据

* 两个十六位的端口号，分别为源端口（可选字段）和目标端口
* 两个数据报文的长度
* 整个数据报文的验证（IPv4可选字段），该字段用于发现头部信息的数据中的错误

因此UDP头部开销小，只有八字节，相比TCP至少二十字节少得多。在传输数据报文时是很高效的

## 三.TCP

TCP协议全称是传输控制协议，是一种面向连接的，可靠的，基于字节流的传输层通信协议，TCP是面向连接的，可靠的流协议，流就是指不间断的数据结构。

### 1.TCP连接过程（三次握手）

* **第一次握手**：客户端向服务端发送连接请求报文段，该报文段中包含自身数据通讯初始序号。请求发送后，客户端便进入SYN-SENT状态  
* **第二次握手**：服务端收到连接请求报文段后，如果同意连接，则会发送一个应答，该应答中也会包含自身的数据通讯初始序号，发送完成后便进入SYN-ECEIVED状态
* **第三次握手**：当客户端收到连接同意的应答后，还要向服务端发送一个确认报文，客户端发完这个报文段后便进入ESTABLISHED状态，服务端收到这个应答后也进入ESTABLISHED状态，此时连接建立成功

TCP建立连接需要三次握手，而不是两次：**为了防止出现失效的连接请求报文段被服务器接收的情况，从而产生错误**

### 2.TCP断开连接（四次挥手）

* **第一次挥手**：若客户端A认为数据发送完成，则它需要向服务端B发送连接释放请求
* **第二次挥手**：B收到连接释放请求后，会告诉应用层要释放TCP连接，然后会发送ACK包，并进入CLOSE_WAIT状态，此时表明A到B的连接已经释放，不再接收A发的数据了，但是因为TCP连接是双向的，所以B仍然可以发送数据给A
* **第三次挥手**：B如果此时还有没有发完的数据会继续发送，完毕后会像A发送连接释放请求，然后B便进入LAST_ACK状态
* **第四次挥手**：A收到释放请求后，向B发送确认应答，此时A进入TIME-WAIT转台，该状态会持续2MSL（最大段生存期，指报文段在网络中的生存时间，超时会被抛弃）时间，若该时间段内没有B的重发请求的话，就进入CLOSED状态，当B收到确认应答后，也进入CLOSED状态

### 3.TCP协议的特点

* 面向连接

面向连接，是指发送数据之前必须在两端建立连接，建立连接的方法就是‘三次握手’，这样能建立可靠的连接，为数据的可靠传输打下基础

* 仅支持单播传输

每天TCP传输连接只能有两个端点，只能进行点对点的数据传输，不支持多播和广播传输方式

* 面向字节流

TCP不像UDP一样一个个报文独立的传输，而是在不保留报文边界的情况下以字节流方式进行传输

* 可靠传输

对于可靠传输，判断丢包，误码靠的是TCP的段编号以及确认号。TCP为了保证报文传输的可靠，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按需接收。然后接收端实体对已成功收到的字节发回一个相应的确认（ACK）；如果发送端实体在合理的往返时延（RTT）内收到确认，那么对应的数据（假设丢失了）将会被重传。

* 提供拥塞控制

当网络出现拥塞时，TCP能够减小向网络注入数据的速率和数量，缓解拥塞

* TCP提供全双工通信

TCP允许通信双方的应用程序在任何时候都能发送数据，因为TCP连接的两端都没有缓存，用来临时存放双向通道的数据。当然TCP可以立即发送一个数据段，也可以缓存一段时间以便一次发送更多的数据段（最大的数据段大小取决于MSS）

## 四.TCP和UDP比较

| |UDP | TCP |
| - | - | - |
|是否连接|无连接|面向连接|
|是否可靠|不可靠传输，不使用流量控制和拥塞控制|可靠传输，使用流量控制和拥塞控制|
|连接对象个数|支持一对一，一对多，多对一和多对多交互通信|只支持一对一通信|
|传输方式|面向报文|面向字节流|
|首部开销|首部开销小，仅8字节|首部最小20字节，最大60字节|
|使用场景|适用实时应用（IP电话，视频会议，直播等）|适用于要求可靠传输的应用，例如文件传输|

* TCP向上层提供面向连接的可靠服务，UDP向上层提供无连接不可靠服务
* 虽然UDP并没有TCP传输来的准确，但是也能在很多实时性要求高的地方有所作为
* 对数据准确性要求高，速度可以相对较慢的，可以选用TCP

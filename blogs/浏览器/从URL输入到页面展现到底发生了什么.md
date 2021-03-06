# 从URL输入到页面展现的步骤

1.在浏览器地址输入URL  
2.浏览器查看**缓存**，如果请求资源在缓存中并且新鲜，跳转到转码步骤

* 如果资源未缓存，发起新请求（协商缓存）
* 如果已缓存(强缓存)，检验是否足够新鲜，足够新鲜直接提供给客户端，否则与服务器进行验证
* 检验新鲜通常有两个HTTP头进行控制Expires（HTTP1.0，值为一个绝对时间）,Cache-Control:max-age(HTTP1.1，值以秒为单位的最大新鲜时间)

3.浏览器**解析URL**获取协议，主机，端口，path  
4.浏览器**组装一个HTTP（GET）请求报文**  
5.浏览器**获取主机ip地址**，过程如下：

* 浏览器缓存
* 本机缓存
* hosts文件
* 路由器缓存
* ISP DNS缓存
* DNS递归查询（可能存在负载均衡导致每次IP不一样）

6.**打开一个socket与目标IP地址，端口建立TCP链接**，三次握手如下：

* 客户端发送一个TCP的**SYN=1，Seq=X**的包到服务器端口（第一次握手，由浏览器发起，告诉服务器我要发送请求了）
* 服务器发回**SYN=1，ACK=X+1，Seq=Y**的响应包（第二次握手，由服务器发起，告诉浏览器我准备接受了，你赶紧发送吧）
* 客户端发送**ACK=Y+1，Swq=Z**（第三次握手，由浏览器发起，告诉服务器，我马上就发了，准备接收）

7.TCP链接建立后**发送HTTP请求**  
8.服务器接收请求并解析，将请求转发到服务程序，如虚拟主机使用HTTP Host头部判断请求的服务程序  
9.服务器检查**HTTP请求头是否包含协商缓存验证信息**如果验证缓存新鲜，返回304等对应状态码  
10.处理程序读取完整请求并准备HTTP响应，可能需要查询数据库等操作  
11.服务器将**响应报文通过TCP链接发送回浏览器**  
12.浏览器接收HTTP响应，然后根据情况选择**关闭TCP链接或者保留重用，关闭TCP链接的四次挥手**如下：

* 主动方发送Fin=1，Ack=Z，Seq=X报文
* 被动方发送ACK=X+1，Seq=Z报文
* 被动方发送Fin=1，ACK=X，Seq=Y报文
* 主动方发送ACK=y，Seq=X报文

13.浏览器检查响应状态码：是否为1xx，3xx,4xx,5xx，这些情况与2xx不同  
14.如果资源可缓存，**进行缓存**  
15.对响应进行**解码**（例如gzip压缩）  
16.根据资源类型决定如何处理（假设资源为HTML文档）  
17.**解析HTML文档，构建DOM树，下载资源，构造CSSOM树，执行js脚本**这些操作没有严格的先后顺序，以下分别解释  
18.**构建DOM树**

* Tokeizing：根据HTML规范将字符串解析为标识
* Lexing：词法分析将标识转换为对象并定义属性和规则
* DOMconstruction：根据HTML标识关系将对象组成DOM树

19.解析过程中遇到图片，样式表，js文件，**启动下载**  
20.构建**CSSOM树**

* Tokenizing：字符流转换为标记流
* Node：根据标记创建节点
* CSSOM：节点创建CSSOM树

21.**根据DOM树和CSSOM树构建渲染树**

* 从DOM树的根节点遍历所有可见节点，不可见节点包括：1.script,meta这样本身不可见的标签。2.被css隐藏的节点，如display：none
* 对每一个可见节点，找到恰当的CSSOM规则并应用
* Layout和Paint最终转换成屏幕上的像素

22.**js解析如下**

* 浏览器创建Document对象并解析HTML，将解析的元素和文本节点添加到文档中，此时document。readystate为loading
* HTML解析器遇到没有async和defer的script时，将他们添加到文档中，然后执行行内和外部脚本。这些脚本会同步执行，并且在脚本下载和执行时解析器会暂停，这样就可以用document.write()把文本插入到输入中，**同步脚本经常简单定义函数和注册事件处理程序，他们可以遍历和操作script和他们之前的文档内容**
* 当解析器遇到了设置了async属性的script时，开始并行下载脚本和解析文档，脚本会在它**下载完成后进行执行(此时解析器停止解析)**，defer属性的script时，也会并行下载脚本和解析文档，但是会**在解析器完成解析之后再去执行**，**异步脚本禁止使用document.write()**
* 当文档解析完成，document.readState变成interactive
* 浏览器在Document对象上触发DOMContentLoaded事件
* 此时文档完全解析完成，浏览器可能还在等待图片等内容加载，等这些**内容完成载入并且所有异步脚本完成载入和执行**，document.readState变为complete,window触发load事件

23.**显示页面**（HTML解析过程中会逐渐显示页面）

## 总共分为6个过程

* DNS解析：将域名解析成ip地址
* TCP链接：TCP三次握手
* 发送HTTP请求
* 服务器处理请求并返回HTTP报文
* 浏览器解析渲染页面
* 断开链接：TCP四次挥手

URL（Uniform Resource Locator） 统一资源定位符  
scheme://host.domain:port/path/filename(https://www.w3school.com.cn/html/index.html)  

* scheme:定义因特网服务的类型。常见的协议有http,https,ftp,file
* host: 定义域主机（http的默认主机为www）
* domain: 定义因特网域名（w3school.com.cn）
* port: 定义主机上的端口号 （http默认80，https默认443）
* path: 定义服务器上的路径
* filename: 定义文档/资源的名称

## 一.域名解析（DNS）

* 浏览器缓存：浏览器回按照一定的频率缓存DNS记录
* 操作系统缓存： 如果浏览器缓存中找不到需要的DNS记录，那就去操作系统中去找
* 路由缓存： 路由器也有DNS缓存
* ISP的DNS服务器：ISP是互联网服务提供商（Internet Service Provider）简称，ISP有专门的DNS服务器应对DNS查询请求
* 根服务器：ISP的DNS服务器还找不到的话，它就会向根服务器发出请求，进行递归查询(DNS服务器会先问根域名服务器.com域名服务器的ip地址，然后再问.baidu域名服务器，以此类推)

小结：**浏览器通过向DNS服务器发送域名，DNS服务器查询到与域名相对于的IP地址，然后返回给浏览器，浏览器再将IP地址打在协议上，同时请求参数也会在协议搭载，然后一并发送给对应的服务器**

## 二.TCP三次握手

三次握手的目的：**为了防止已失效的链接请求报文段突然又传送到了服务端，因而产生错误**

* 客户端发送一个TCP的**SYN=1，Seq=X**的包到服务器端口（第一次握手，由浏览器发起，告诉服务器我要发送请求了）
* 服务器发回**SYN=1，ACK=X+1，Seq=Y**的响应包（第二次握手，由服务器发起，告诉浏览器我准备接受了，你赶紧发送吧）
* 客户端发送**ACK=Y+1，Swq=Z**（第三次握手，由浏览器发起，告诉服务器，我马上就发了，准备接收）

## 三.发送HTTP请求

TCP三次握手结束后，开始发送HTTP请求报文  
请求报文由请求行（request line）,请求头(header),请求体三个部分组成  

### 1.请求行包含请求方法，URL，协议版本

* 请求方法包含8种：GET POST PUT DELETE CONNECT HEAD OPTIONS TRACE
* URL即请求地址，由<协议>://<主机>:<端口>/<路径>?<参数>组成
* 协议版本即http版本号

> POST /chapter17/index.html HTTP/1.1

### 2.请求头包含请求的附加信息，由关键字/值对组成，每行一对，关键字和值用英文冒号“:”分隔

请求头部通知服务器有关于客户端请求的信息。它包含许多有关的客户端环境和请求正文的有用信息。比如：**Host：主机名，虚拟主机；Connection，HTTP/1.1增加的，使用keepalive，即持久链接，一个链接可以发多个请求；User-Agent：请求发出者，兼容性以及定制化需求**

### 3.请求体，可以承载多个请求参数的数据，包含回车符，换行符和请求数据，并不是所有请求都具有请求数据

```html
GET /Protocols/rfc2616/rfc2616-sec5.html HTTP/1.1
Host: www.w3.org
Connection: keep-alive
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36
Referer: https://www.google.com.hk/
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: authorstyle=yes
If-None-Match: "2cc8-3e3073913b100"
If-Modified-Since: Wed, 01 Sep 2004 13:24:52 GMT

name=qiu&age=25
```

> name=nike&password=123&realName='zhangsan'

## 四.服务器处理请求并返回HTTP报文

服务器对于不同用户发送的请求，会结合配置文件，把不同请求委托给服务器上处理相应请求的程序进行处理（例如JSP脚本，service，服务器端JavaScript等），然后返回后台程序处理的结果作为相应

### http响应报文

响应报文由响应行（request line）,响应头部(header),响应主体三个部分组成  
1.响应行包含：协议版本，状态码，状态码描述

* 1xx:提示信息-表示请求已接收，继续处理
* 2xx:成功-表示请求已被成功接收
* 3xx:重定向-要完成请求必须进行进一步操作
* 4xx:客户端错误-请求有语法错误或请求无法实现
* 5xx:服务器错误-服务器未能实现合法的请求

2.响应头部包含响应报文的附加信息，由名/值 对组成  
3.响应主体包含回车符，换行符和响应返回数据，并不是所有响应报文都有响应数据

```html
HTTP/1.1 200 OK
Date: Tue, 08 Jul 2014 05:28:43 GMT
Server: Apache/2
Last-Modified: Wed, 01 Sep 2004 13:24:52 GMT
ETag: "40d7-3e3073913b100"
Accept-Ranges: bytes
Content-Length: 16599
Cache-Control: max-age=21600
Expires: Tue, 08 Jul 2014 11:28:43 GMT
P3P: policyref="http://www.w3.org/2001/05/P3P/p3p.xml"
Content-Type: text/html; charset=iso-8859-1

{"name": "qiu", "age": 25}
```

## 五.浏览器解析页面

五个步骤：

### 1.根据HTML解析DOM树

* 根据HTML的内容，将标签按照结构解析成为DOM树，DOM树解析的过程是一个深度优先遍历的过程
* Tokeizing：根据HTML规范将字符串解析为标识
* Lexing：词法分析将标识转换为对象并定义属性和规则
* DOMconstruction：根据HTML标识关系将对象组成DOM树
* 在读取HTML文档，构建DOM树的过程中，若遇到script标签，则DOM树的构建会暂停，直至脚本执行完毕

### 2.根据CSS解析生成CSS规则树

* Tokeizing：根据CSS规范将字符串解析为标识
* Lexing：词法分析将标识转换为对象并定义属性和规则
* DOMconstruction：根据CSS标识关系将对象组成CSSOM树
* 解析CSS规则树时js执行将暂停，直至CSS规则树就绪
* 浏览器在CSS规则树生成之前不会进行渲染

### 3.结合DOM树和CSS规则树，生成渲染树

* DOM树和CSS规则树全部准备好了后，浏览器才会开始构建渲染树
* 精简CSS并可以加快CSS规则树的构建，从而加快页面响应速度

### 4.根据渲染计算每个节点的信息（布局）

* 布局：通过渲染树中渲染的对象的信息，计算出每一个渲染对象的位置和尺寸
* 回流：在布局完成后，发现了某个部分发生了变化影响了布局，需要重新渲染

### 5.根据计算好的信息绘制页面

* 绘制阶段，系统会遍历呈现树，并调用呈现器的‘paint’方法，将呈现器的内容显示在屏幕上
* 重绘：某个元素的背景颜色，文字颜色，不影响元素周围或内部布局的属性，将只会引起浏览器的重绘
* 回流：某个元素的尺寸发生了变化，则需要重新计算渲染树，重新渲染

## 六.断开连接

* **发起方向被动方发送报文，Fin,Ack,Seq，表示已经没有数据传输了，并进入FIN_WAIT_1状态**（第一次挥手：由浏览器发起的，发送给服务器，我请求的报文发送完了，你准备关闭吧
* **被动方发送报文，Ack，Seq，表示同意关闭请求，此时主机发起方进入FIN_WAIT_2状态**(第二次挥手：由服务器发起，告诉浏览器，我请求报文接受完了，你也准备吧)
* **被动方向发起方发送报文，Fin，Ack，Seq，请求关闭链接，并进入LAST_ACK状态**（第三次挥手：由服务器发起，告诉浏览器，我响应报文发送完了，你准备关闭吧）
* **发起方向被动方发送报文，Ack Seq。然后进入等待TIME_WAIT状态。被动方收到发起方的报文后关闭链接，发起方等待一段时间未收到回复，则正常关闭**（第四次挥手：由浏览器发起，告诉服务器，我响应报文接受完了，我准备关闭，你也准备吧）

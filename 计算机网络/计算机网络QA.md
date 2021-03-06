[toc]

## OSI 七层模型、TCP/IP模型

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220315122533.png)

- 物理层：底层数据传输，如网线；网卡标准。
- 数据链路层：定义数据的基本格式，如何传输，如何标识；如网卡MAC地址。
- 网络层：定义IP编址，定义路由功能；如不同设备的数据转发。
- 传输层：端到端传输数据的基本功能；如 TCP、UDP。
- 会话层：控制应用程序之间会话能力；如不同软件数据分发给不同软件。
- 表示层：数据格式标识，基本压缩加密功能。
- 应用层：各种应用软件，包括 Web 应用。

说明：

- 在四层，既传输层数据被称作**段**（Segments）；
- 三层网络层数据被称做**包**（Packages）；
- 二层数据链路层时数据被称为**帧**（Frames）；
- 一层物理层时数据被称为**比特流**（Bits）。

==TCP协议在传输层，IP协议在网络层，HTTP协议在应用层，DNS协议在应用层==

## TCP

https://www.acwing.com/blog/content/7861/

### TCP 三次握手建立连接

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220309231458.png)

0）初始状态：客户端处于 `closed(关闭)`状态，服务器处于 `listen(监听)` 状态。

1）第一次握手：客户端向服务端发送一个 SYN 报文（SYN = 1）和初始序列号 ISN(x)，即图中的 seq = x，表示本报文段所发送的数据的第一个字节的序号。发送后客户端会进入 `SYN-SENT`状态

> SYN-SENT ：在发送连接请求后等待匹配的连接请求（发送等待状态）

2）第二次握手：服务器收到客户端的 SYN 报文之后，如果同意连接，会发送 SYN 报文（SYN = 1）作为应答，和初始化序列号 ISN(y)，即图中的 seq = y。同时会发送一个 ACK 报文，把客户端的 ISN + 1 作为确认号 ack 的值，表示已经收到了客户端发来的的 SYN 报文，希望收到的下一个数据的第一个字节的序号是 x + 1，此时服务器处于 `SYN_REVD`的状态。

> SYN-RECEIVED：在收到和发送一个连接请求后等待对连接请求的确认（确认接收状态）

3）第三次握手：客户端收到服务器端响应的 SYN 报文之后，会发送一个 ACK 报文，也是一样把服务器的 ISN + 1 作为 ack 的值，表示已经收到了服务端发来的的 SYN 报文，希望收到的下一个数据的第一个字节的序号是 y + 1，并指明此时客户端的序列号 seq = x + 1（初始为 seq = x，所以第二个报文段要 +1），此时客户端处于 `ESTABLISHED`状态。

服务器收到 ACK 报文之后，也处于 `ESTABLISHED`状态，至此，双方建立起了 TCP 连接。

> ESTABLISHED：代表一个打开的连接，数据可以传送给用户（确认连接状态）

### TCP为什么是三次握手？

三次握手最主要的目的就是服务端和客户端双方都==确认自己与对方的发送与接收是正常的==。

- 第一次握手（客户端发送 SYN 报文给服务器，服务器接收该报文）：客户端什么都不能确认；服务器确认了对方发送正常，自己接收正常
- 第二次握手（服务器响应 SYN 报文给客户端，客户端接收该报文）：

  客户端确认了：自己发送、接收正常，对方发送、接收正常；

  服务器确认了：对方发送正常，自己接收正常
- 第三次握手（客户端发送 ACK 报文给服务器）：

  客户端确认了：自己发送、接收正常，对方发送、接收正常；

  服务器确认了：自己发送、接收正常，对方发送、接收正常

### TCP 四次挥手释放连接

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220309234950.png)

0）初始状态：刚开始双方都处于 `ESTABLISHED`状态。假如是客户端先发起关闭请求。

1）第一次挥手：客户端发送一个 FIN 报文（请求连接终止：FIN = 1），报文中会指定一个序列号 seq = u。并**停止再发送数据，主动关闭 TCP 连接**。此时客户端处于 `FIN_WAIT1` 状态，等待服务端的确认。

> FIN-WAIT-1 - 等待远程TCP的连接中断请求，或先前的连接中断请求的确认；

2）第二次挥手：服务器收到FIN报文后知道客户端想断开连接，但此时服务器不一定能做好准备，可能还有未传输完的消息，所以服务器只能先返回一个ACK报文段，且把客户端的序号值 +1 作为 ACK 报文的序列号值，表明已经收到客户端的报文了，并进入 `CLOSE_WAIT` 状态；

> CLOSE-WAIT - 等待从本地用户发来的连接中断请求；

**此时的 TCP 处于半关闭状态，客户端到服务端的连接释放**。客户端收到服务端的确认后，进入 `FIN_WAIT2`（终止等待 2）状态，等待服务端发出的连接释放报文段。

> FIN-WAIT-2 - 从远程TCP等待连接中断请求；

3）服务器已经准备好断开连接，向客户端发送FIN报文段，且指定一个序列号，关闭服务器到客户端的数据传输，服务器进入 `LAST_ACK`状态，等待客户端的确认。

> LAST-ACK - 等待原来发向远程TCP的连接中断请求的确认；

4）第四次挥手：客户端收到 FIN 之后，一样发送一个 ACK 报文作为应答（ack = w+1），且把服务端的序列值 +1 作为自己 ACK 报文的序号值（seq=u+1），然后客户端进入  **`TIME_WAIT` （时间等待）状态**并==等待2MSL时间后进入 `CLOSED`状态==，服务器收到客户端的FIN报文段后会进入 `CLOSED`状态，完成四次挥手。

> TIME-WAIT - 等待足够的时间以确保远程TCP接收到连接中断请求的确认；

### TCP为什么要四次挥手?

由于 TCP 的**半关闭**（half-close）特性，TCP 提供了连接的一端在结束它的发送后还能接收来自另一端数据的能力。

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入**半关闭状态**。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就**完全关闭**了TCP连接。

### TCP 怎么保证可靠传输

==可靠传输就是保证接收方收到的字节流和发送方发出的字节流是完全一样的。==

- **数据校验**：TCP报文头有校验和，用于校验报文是否损坏。
- **确认应答机制**：接收方收到 TCP 报文段后就会返回一个确认应答消息
- **重传机制**：发送方发送一段时间后没有收到确认就会重传。
- **流量控制**：当接收方来不及处理发送方的数据，能通过滑动窗口，提示发送方降低发送的速率，防止包丢失。
- **拥塞控制**：当网络拥塞时，通过拥塞窗口，减少数据的发送，防止包丢失。

### TCP 初始序列号为什么是随机值

这样做主要是出于网络安全的因素着想。==如果 ISN 是固定的，攻击者很容易猜出后续的确认号==，攻击者很容易猜出后续的确认号，然后伪造序列号进行攻击。

## TCP 和 UDP 的区别

| **UDP**                      | **TCP**                            |
| ---------------------------------- | ---------------------------------------- |
| UDP无连接                          | TCP面向连接                              |
| 尽最大努力交付                     | 提供可靠的服务（不错、不丢、不重、有序） |
| 面向报文                           | 面向字节流                               |
| 没有拥塞控制                       | 有拥塞控制                               |
| 支持一对一，一对多，多对一和多对多 | 点到点                                   |
| 首部开销小                         | 首部开销大                               |
| 不可靠信道                         | 全双工的可靠信道                         |

## DNS

### 为什么需要 DNS 协议？(DNS 原理、DNS 协议有啥用)

由于 IP 地址具有不方便记忆并且不能显示地址组织的名称和性质等缺点，人们设计出了域名，并通过域名解析协议（DNS，Domain Name System）来将域名和 IP 地址相互映射，使我们可以更方便地访问互联网，而不用去记住能够被机器直接读取的 IP 地址数串。

DNS 工作过程:

1）首先搜索浏览器的 DNS 缓存，缓存中维护一张域名与 IP 地址的对应表；

2）若没有命中，则继续搜索操作系统的 DNS 缓存；

3）若仍然没有命中，则操作系统将域名发送至本地域名服务器，本地域名服务器查询自己的 DNS 缓存，查找成功则返回结果（主机和本地域名服务器之间的查询方式是递归查询）；

4）若本地域名服务器的 DNS 缓存没有命中，则本地域名服务器向上级域名服务器进行查询，通过以下方式进行迭代查询（本地域名服务器和其他域名服务器之间的查询方式是迭代查询，防止根域名服务器压力过大）：

- 首先本地域名服务器向根域名服务器发起请求，根域名服务器是最高层次的，它并不会直接指明这个域名对应的 IP 地址，而是返回顶级域名服务器的地址，也就是说给本地域名服务器指明一条道路，让他去这里寻找答案
- 本地域名服务器拿到这个顶级域名服务器的地址后，就向其发起请求，获取权限域名服务器的地址
- 本地域名服务器根据权限域名服务器的地址向其发起请求，最终得到该域名对应的 IP 地址

4）本地域名服务器将得到的 IP 地址返回给操作系统，同时自己将 IP 地址缓存起来

5）操作系统将 IP 地址返回给浏览器，同时自己也将 IP 地址缓存起来

6）至此，浏览器就得到了域名对应的 IP 地址，并将 IP 地址缓存起来

==总结： 浏览器缓存，系统缓存，路由器缓存，IPS服务器缓存，根域名服务器缓存，顶级域名服务器缓存，主域名服务器缓存。==

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220310222046.png)

==主机和本地域名服务器之间的查询方式是递归查询==

### DNS 协议基于 TCP 还是 UDP？

- ==域名解析使用UDP==：

  UDP快，UDP的DNS协议只要一个请求、一个应答就好了。

  而使用基于TCP的DNS协议要三次握手、发送数据以及应答、四次挥手，但是UDP协议传输内容不能超过512字节。

  不过客户端向DNS服务器查询域名，一般返回的内容都不超过512字节，用UDP传输即可。
- ==区域传输使用TCP==：（区域传输就是辅助域名服务器与主域名服务器通信，并同步数据信息的过程。）

  1. TCP协议可靠性好
  2. 数据同步传送的数据量比一个 DNS 请求和响应报文的数据量要多得多。

## Cookie和Session

Cookie 是**服务器发送到用户浏览器并保存在本地的一小块数据**，它会在浏览器之后向同一服务器再次发起请求时被携带上，用于告知服务端两个请求是否来自同一浏览器。

session 是另一种记录服务器和客户端会话状态的机制。session 是基于 cookie 实现的，session 存储在服务器端，sessionId 会被存储到客户端的cookie 中

抽象地概括一下：一个 cookie 可以认为是一个「变量」，形如 name=value，存储在浏览器；一个 session 可以理解为一种数据结构，多数情况是「映射」（键值对），存储在服务器上。

### Cookie和Session的区别

1、存储的位置不同。Cookie 在客户端（浏览器），Session 在服务器端。

2、存储的数据类型不同。Cookie 只支持存字符串数据，想要设置其他类型的数据，需要将其转换成字符串，Session 可以存任意数据类型。

3、安全性不同。Session 比 Cookie 安全。

4、存储大小不同。Cookie大小受浏览器的限制，一般单个Cookie保存的数据不能超过4K。Session 可存储数据远高于 Cookie，但是当访问量过多，会占用过多的服务器资源。

5、有效期不同。 Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭（默认情况下）或者 Session 超时都会失效。

## 在浏览器中输入url地址后显示主页的过程

- 根据域名，进行DNS域名解析；
- 拿到解析的IP地址，建立TCP连接；
- 向IP地址，发送HTTP请求；
- 服务器处理请求并返回HTTP报文；
- 关闭TCP连接；
- 浏览器解析HTML渲染页面；

## HTTP、HTTPS

### HTTP、HTTPS的区别

1. http是明文传输，https是加密的安全传输
2. http的端口默认是80，https的端口默认是443
3. http是无状态的连接，https是ssl加密的传输，身份认证的网络协议。
4. http的 URL 前缀是 `http://`，https的 URL 前缀是 `https://`
5. https比http耗费更多服务器资源
6. https协议要CA证书申请

### HTTP方法

- GET：获取资源
- POST：传输实体主体
- PUT：传输文件
- HEAD：获得报文首部，与GET方法一样，只是不返回报文主体内容。用于确认URI的有效性及资源更新的日期时间等。
- DELETE：删除文件，与PUT相反（响应返回204 No Content）
- OPTIONS：询问支持的方法，查询针对请求URI指定的资源支持的方法（Allow:GET、POST、HEAD、OPTIONS）
- TRACE：追踪路径
- CONNECT：要求用隧道协议连接代理，主要使用SSL（Secure Sockets Layer，安全套接层）和TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输

### HTTP状态码

| 状态码 | 类别                             | 含义                       |
| :----- | :------------------------------- | :------------------------- |
| 1XX    | Informational（信息性状态码）    | 接收的请求正在处理         |
| 2XX    | Success（成功状态码）            | 请求正常处理完毕           |
| 3XX    | Redirection（重定向状态码）      | 需要进行附加操作以完成请求 |
| 4XX    | Client Error（客户端错误状态码） | 服务器无法处理请求         |
| 5XX    | Server Error（服务器错误状态码） | 服务器处理请求出           |

##### 1xx 信息

- **100 Continue** ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

##### 2xx 请求正常处理完毕

- **200 OK**：客户端请求成功。
- **204 No Content** ：无内容。服务器成功处理，但未返回内容。一般用在只是客户端向服务器发送信息，而服务器不用向客户端返回什么信息的情况。不会刷新页面。
- **206 Partial Content** ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。

##### 3xx 重定向

- **301 Moved Permanently** ：永久重定向，表示请求的资源已经永久的搬到了其他位置。
- **302 Found** ：临时重定向，表示请求的资源临时搬到了其他位置
- **303 See Other** ：临时重定向，应使用GET定向获取请求资源。303功能与302一样，区别只是303明确客户端应该使用GET访问。
- **304 Not Modified** ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。
- **307 Temporary Redirect** ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

##### 4xx 客户端错误

- **400 Bad Request** ：请求报文中存在语法错误。
- **401 Unauthorized** ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。
- **403 Forbidden** ：请求被拒绝。
- **404 Not Found**：请求资源不存在。比如，输入了错误的 URL。
- **415 Unsupported media type**：不支持的媒体类型。

##### 5xx 服务器错误

- **500 Internal Server Error** ：服务器正在执行请求时发生错误。
- **503 Service Unavailable** ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

### 对称加密和非对称加密

对称加密：一份密钥
![](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220731005859.png)

非对称加密：公钥+密钥
![](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220731011207.png)

### HTTPS是如何保证数据传输的安全，整体的流程是什么？（SSL是怎么工作保证安全的）

（1）客户端向服务器端发起SSL连接请求

（2）服务器把公钥发送给客户端，并且服务器端保存着唯一的私钥

（3）客户端用公钥对双方通信的对称密钥进行加密，并发送给服务器端

（4）服务器利用自己唯一的私钥对客户端发来的对称密钥进行解密

进行数据传输，服务器和客户端双方用公有的相同的对称密钥对数据进行加密解密，可以保证在数据收发过程中的安全，即是第三方获得数据包，也无法对其进行加密，解密和篡改。

### HTTPS中数字认证流程

https://www.cnblogs.com/fengf233/p/11775415.html

由于公钥在下发的时候也容易被替换劫持，所以需要个第三方认证机构确认公钥的正确性

CA：数字证书认证机构，是客户端服务端都认可的第三方机构，负责数字签名服务端公钥

数字签名：签名就是一种证明身份的机制，是一种校验机制（简单说是用私钥加密内容的hash,公钥解密对比hash判断内容是否完整）

数字证书：由一个可信的组织验证和签发的识别信息

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220329123703.png)

## ICMP协议 Ping Traceroute

https://zhuanlan.zhihu.com/p/116902722

### Ping 原理

Ping 的原理是 ICMP 协议。

- 客户端：向服务端发送ICMP回显请求报文（echo message）。
- 服务端：向客户端返回ICMP回西显响应报文（echo reply message）。

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220317130056.jpg)

### ICMP 协议的格式

ICMP 包头的**类型**字段，大致可以分为两大类：

- 一类是用于诊断的查询消息，也就是「**查询报文类型**」
- 另一类是通知出错原因的错误消息，也就是「**差错报文类型**」

![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220317130156)

### Traceroute（路由追踪） 原理

https://zhuanlan.zhihu.com/p/404043710

traceroute的实现原理，有两种方法：1、基于UDP报文实现；2、基于ICMP报文实现。

### 基于UDP报文实现

让你在客户端输入 traceroute 命令+ip时， 客户端就发起一个UDP报文，使用一个大于30000的端口号（选这么端口号，目的端一般都是未使用，所以待会就收到一个端口不可达信息。）这样子，服务器端收到这个UDP报文后就会返回ICMP端口不可达的错误信息。同时，第一个数据包，TTL=1，这样第一跳路由器收到后,要转发出去时，会将TTL减一，即TTL=0, 就丢弃，然后第一跳路由器就返回一个ICMP超时的错误信息，所以，客户端通过判断收到ICMP端口不可达报文来确定数据包已到达目的地，如果是收到ICMP超时错误信息报文，说明还没到达目的地，就会将TTL加1，以此类推。

<video id="video" controls=""src="https://vdn.vzuu.com/SD/439fed4a-06d7-11ec-9f4b-ba0fc80291c2.mp4?disable_local_cache=1&auth_key=1647497018-0-0-18cb4276de00ae3f857b3c65b509551f&f=mp4&bu=pico&expiration=1647497018&v=ali" preload="none">

<video id="video" controls=""src="https://vdn1.vzuu.com/SD/80dda4c2-06d7-11ec-8c9c-3e117ea31dbf.mp4?disable_local_cache=1&auth_key=1647497052-0-0-3a2d3738c9569e5793a879f0261c42d6&f=mp4&bu=pico&expiration=1647497052&v=hw" preload="none">
<video id="video" controls=""src="https://vdn1.vzuu.com/SD/9bdfbdb4-06d7-11ec-862e-06c1ba80f579.mp4?disable_local_cache=1&auth_key=1647497255-0-0-d82ab6a5e1d17137650baf8ad3256b07&f=mp4&bu=pico&expiration=1647497255&v=hw" preload="none">

### 基于ICMP报文实现

使用ICMP的**回显请求**和**回显应答**这两种报文

让你在客户端输入 traceroute 命令+ip时， 客户端就发起一个ICMP回显请求报文，第一个数据包，TTL=1，这样第一跳路由器收到后,要转发出去时，会将TTL减一，即TTL=0, 就丢弃，然后第一跳路由器就返回一个ICMP超时的错误信息，客户端收到后，会判断是否收到ICMP 回显应答 报文？ 如果还没收到，就会继续发送回显请求报文，TTL加1进行尝试，当到底服务器后，服务器就会发送ICMP 回显应答报文。

<video id="video" controls=""src="https://vdn1.vzuu.com/SD/b9a54260-06d7-11ec-8986-3623b0d475ed.mp4?disable_local_cache=1&auth_key=1648531851-0-0-af6d184918e4437b935908540e9237dd&f=mp4&bu=pico&expiration=1648531851&v=hw" preload="none">

<video id="video" controls=""src="https://vdn1.vzuu.com/SD/f2ae763a-06d7-11ec-be9c-124d99edaad9.mp4?disable_local_cache=1&auth_key=1648440770-0-0-4cd8b67bb99ba8c272c7c06b74c073d8&f=mp4&bu=pico&expiration=1648440770&v=hw" preload="none">
<video id="video" controls=""src="https://vdn.vzuu.com/SD/0cad36b6-06d8-11ec-882a-2aaab6ff7f5f.mp4?disable_local_cache=1&auth_key=1648531851-0-0-7286ff326badfc1c1b66f60a6480d9f3&f=mp4&bu=pico&expiration=1648531851&v=ali" preload="none">

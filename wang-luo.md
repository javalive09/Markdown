# 网络

## URI（uniformresource identifier）和URL（universal resource Loactor）的区别

一个URI定义了一种资源，它是一个定位符。 一个URI由两部分组成： scheme （URI协议名）：scheme-specific-part（URI协议对应的内容） 一个URL由三部分组成：scheme（协议）：// 主机IP地址 / 资源具体地址 一个URI是一个URL或URN URL、URN是URI的子集 当对某种资源是URL还是URI产生疑惑时，用URI来定义它

## HTTP协议

超文本传输协议，用于网络间传输数据。传输的内容不仅是文本文字，还有图片，程序包，文件等各种格式的数据。 HTTP是一个应用层协议，由请求和响应构成，是一个标准的客户端服务器模型。HTTP是一个无状态的协议。

### 请求

请求行（负责请求的方式），消息头（负责请求的协议），正文（负责请求的内容）

* request-line : Method  Request-URI  http-version
* Request-Header-Fields : General-Header  Request-Header  Entity-Header
* Entity-Body

### 响应

状态行（负责响应的状态提示），消息头（负责相应的协议），正文（负责响应的内容）

* Status-Line : http-version  Status-Code  Reason-Phrase
* Response Header Fields : General-Header  Response-Header  Entity-Header
* Entity-Body

#### 状态码

| 状态码 | 类别 | 详情 |
| :--- | :--- | :--- |
| 1XX | informational\(信息性\) | 接收的请求正在处理 |
| 2XX | success\(成功\) | 请求正常处理完毕 |
| 3XX | redirection\(重定向\) | 需要进行附加操作以完成请求 |
| 4XX | client error\(客户端错误\) | 服务器无法处理请求 |
| 5XX | server error\(服务器端错误\) | 服务器处理请求出错 |

| 状态码 | 详情 | 场景 |
| :--- | :--- | :--- |
| 204 | 请求处理成功，但没有资源可返回。返回的响应报文中不含实体，也不允许含实体，从浏览器发出请求，返回204，则浏览器显示的页面不更新 | 一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用 |
| 206 | 表示客户端进行了范围请求，响应报文中包含content-Range指定范围的实体内容 |  |
| 301 | 永久重定向。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI 保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存。 |  |
| 302 | 临时性重定向。该状态码表示请求的资源已被分配了新的 URI，希望用户（本次）能使用新的 URI 访问。 | 已移动的资源对应的 URI 将来还有可能发生改变。比如，用户把 URI 保存成书签，但不会像 301 状态码出现时那样去更新书签，而是仍旧保留返回 302 状态码的页面对应的 URI。 |
| 303 | 该状态码表示由于请求对应的资源存在着另一个 URI，应使用 GET 方法定向获取请求的资源。303 状态码和 302 Found 状态码有着相同的功能，但 303 状态码明确表示客户端应当采用 GET 方法获取资源，这点与 302 状态码有区别。 | 当 301、302、303 响应状态码返回时，几乎所有的浏览器都会把 POST 改成 GET，并删除请求报文内的主体，之后请求会自动再次发送。301、302 标准是禁止将 POST 方法改变成 GET 方法的，但实际使用时大家都会这么做。 |
| 304 | 该状态码表示客户端发送附带条件的请求 时，服务器端允许请求访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应的主体部分。 | 条件请求，缓存无需更新的返回码 |
| 307 | 临时重定向。该状态码与 302 Found 有着相同的含义。尽管 302 标准禁止 POST 变换成 GET，但实际使用时大家并不遵守。 | 307 会遵照浏览器标准，不会从 POST 变成 GET。但是，对于处理响应时的行为，每种浏览器有可能出现不同的情况。 |
| 400 | 该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。另外，浏览器会像 200 OK 一样对待该状态码。 |  |
| 401 | 该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证、DIGEST 认证）的认证信息。另外若之前已进行过 1 次请求，则表示用 户认证失败。 | 返回含有 401 的响应必须包含一个适用于被请求资源的 WWW-Authenticate 首部用以质询（challenge）用户信息。当浏览器初次接收到 401 响应，会弹出认证用的对话窗口。 |
| 403 | 该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。 | 未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源 IP 地址试图访问）等列举的情况都可能是发生 403 的原因。 |
| 404 | 该状态码表明服务器上无法找到请求的资源。除此之外，也可以在服务器端拒绝请求且不想说明理由时使用。 |  |
| 412 | 服务器会比对 If-Match 的字段值和资源的 ETag 值，仅当两者一致时，才会执行请求。反之，则返回状态码 412 Precondition Failed 的响应。 | 还可以使用星号（\*）指定 If-Match 的字段值。针对这种情况，服务器将会忽略 ETag 的值，只要资源存在就处理请求。 |
| 500 | 该状态码表明服务器端在执行请求时发生了错误。也有可能是 Web 应用存在的 bug 或某些临时的故障。 | 服务器内部资源出故障 |
| 503 | 该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。如果事先得知解除以上状况需要的时间，最好写入 RetryAfter 首部字段再返回给客户端。 | 服务器正在忙 |

### http header fileds

#### General Header Fields

* Cache-Control
* Connection
* Date
* Pragma
* Trailer
* Transfer-Encoding
* Upgrade
* Via
* Warning

#### Request Header Fields

* Accept
* Accept-Charset
* Accept-Encoding
* Accept-Language
* Authorization
* Expect
* From
* Host
* If-Match
* If-Modified-Since
* If-None-Match
* If-Range
* If-Unmodified-Since
* Max-Forwards
* Proxy-Authorization
* Range
* Referer
* TE
* User-Agent

#### Response Header Fields

* Accept-Ranges
* Age
* ETag
* Location
* Proxy-Authenticate
* Retry-After
* Server
* Vary
* WWW-Authenticate

#### Entity Header Fields

* Content-Encoding
* Content-Language
* Content-Length
* Content-Location
* Content-MD5
* Content-Range
* Content-Type
* Expires
* Last-Modified
* Set-Cookie
* Cookie

#### other Header Fields

* X-Frame-Options
* X-XSS-Protection
* DNT
* P3P

## tcp连接 为什么要进行3次握手，断开为什么要4次握手

![](.gitbook/assets/tcp-on-off.jpg)

### 建立连接

 [3次握手具体抓包分析](http://www.omnisecu.com/tcpip/tcp-three-way-handshake.php)

《计算机网络》中是这样解释的：

```text
“已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞
留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要
server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server
发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。采用“
三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知
道client并没有要求建立连接。”
```

### 断开连接

[4次挥手具体抓包分析](https://wiki.wireshark.org/TCP%204-times%20close) TCP是全双工模式，双方确定彼此都要挂断电话后，正式断开连接。 因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

注：

```text
TIME_WAIT作用在于防止TCP client的ack丢失。
TIME_WAIT状态中，如果TCP client端最后一次发送的ACK丢失了，它将重新发送。
TIME_WAIT状态中所需要的时间是依赖于实现方法的。典型的值为30秒、1分钟和2分钟。
等待之后连接正式关闭，并且所有的资源(包括端口号)都被释放。
```

### 什么是MIME

用于标记传输的数据的文件类型。 最早应用于电子邮件系统，但后来也应用到浏览器，以及所有的cs 交互模型中。在HTTP协议中用Content-type描述。 MIME\(Multipurpose Internet Mail Extensions\) 多用途互联网邮件扩展类型，是一个互联网标准，标准定义在 [RFC](http://www.cnpaf.net/class/rfcall/) 文档中。 http post提交表单格式数据 content-type ：application/x-www-form-urlencoded http post提交json格式数据 content-type ：application/json

## UDP协议

### 概念

UDP: User Datagram Protocol 用户数据报协议 是一种无连接的协议。UDP有不提供数据包分组、组装和不能对数据包进行排序的缺点，也就是说，当报文发送之后，是无法得知其是否安全完整到达的。UDP用来支持那些需要在计算机之间传输数据的网络应用。包括网络视频会议系统在内的众多的客户/服务器模式的网络应用都需要使用UDP协议。[参考](http://colobu.com/2014/10/21/udp-and-unicast-multicast-broadcast-anycast/) [Netty UDP 官方 demo](https://github.com/netty/netty/tree/4.0/example/src/main/java/io/netty/example/qotm)

### 使用

在选择使用协议的时候，选择UDP必须要谨慎。在网络质量令人十分不满意的环境下，UDP协议数据包丢失会比较严重。但是由于UDP的特性：它不属于连接型协议，因而具有资源消耗小，处理速度快的优点，所以通常音频、视频和普通数据在传送时使用UDP较多。 UDP广播：广播使用广播地址255.255.255.255，将消息发送到在同一广播网络上的每个主机。值得强调的是：本地广播信息是不会被路由器转发。当然这是十分容易理解的，因为如果路由器转发了广播信息，那么势必会引起网络瘫痪。这也是为什么IP协议的设计者故意没有定义互联网范围的广播机制。其实广播顾名思义，就是想局域网内所有的人说话，但是广播还是要指明接收者的端口号的，因为不可能接受者的所有端口都来收听广播。

### UDP广播

广播使用广播地址255.255.255.255，将消息发送到在同一广播网络上的每个主机。值得强调的是：本地广播信息是不会被路由器转发。当然这是十分容易理解的，因为如果路由器转发了广播信息，那么势必会引起网络瘫痪。这也是为什么IP协议的设计者故意没有定义互联网范围的广播机制。其实广播顾名思义，就是想局域网内所有的人说话，但是广播还是要指明接收者的端口号的，因为不可能接受者的所有端口都来收听广播。

#### 发送接收代码

```text
    public static void sendBroadcast(byte[] data, int port) throws Exception{
        DatagramSocket socket = new DatagramSocket();
        socket.setBroadcast(true);
        DatagramPacket packet = new DatagramPacket(data, data.length, InetAddress.getByName("255.255.255.255"), port);
        socket.send(packet);
    }

    public static void receiveBoradcast(int port) {
        new Thread(new Runnable() {

            DatagramSocket socket;

            @Override
            public void run() {
                while (true) {
                    try {
                        receive(port);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }

            public void receive(int port) throws Exception{
                if (socket == null || socket.isClosed()) {
                    socket = new DatagramSocket(port);
                    socket.setBroadcast(true);
                }

                byte[] buffer = new byte[1024 * 100];
                DatagramPacket packet = new DatagramPacket(buffer , buffer.length);
                socket.receive(packet);
                String data = new String(packet.getData(), 0, packet.getLength());
                Log.i("receiver", data);
                socket.close();
            }

        }).start();
    }
```

#### Netty 实现代码

```text
public class Broadcast {

    Channel udpChannel;

    public void send(String data) throws Exception{
        if(udpChannel != null) {
            udpChannel.writeAndFlush(data);
        }
    }

    public void init(int port) {
        NioEventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(group)
                .channel(NioDatagramChannel.class)
                .option(ChannelOption.SO_BROADCAST, true)
                .handler(new MessageToMessageEncoder<String>() {
                    @Override
                    protected void encode(ChannelHandlerContext ctx, String msg, List<Object> out) throws Exception {
                        InetSocketAddress remoteAddress = new InetSocketAddress("255.255.255.255", port);
                        out.add(new DatagramPacket(Unpooled.copiedBuffer(msg, CharsetUtil.UTF_8), remoteAddress));
                    }
                });
        udpChannel = bootstrap.bind(0).syncUninterruptibly().channel();
    }

}

public class BroadcastReceiver {

    public void start(int port){
        Bootstrap b = new Bootstrap();
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            b.group(group)
                    .channel(NioDatagramChannel.class)
                    .option(ChannelOption.SO_BROADCAST, true)
                    .handler(new SimpleChannelInboundHandler<DatagramPacket>() {

                        @Override
                        protected void messageReceived(ChannelHandlerContext ctx, DatagramPacket packet) throws Exception {
                            String req = packet.content().toString(CharsetUtil.UTF_8);
                            System.out.println(req);
                        }
                    });
            b.bind(port).sync().channel().closeFuture().await();
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            group.shutdownGracefully();
        }
    }

}
```

### UDP多播

多播数据报套接字类用于发送和接收 IP 多播包。MulticastSocket 是一种 \(UDP\) DatagramSocket，它具有加入 Internet 上其他多播主机的“组”的附加功能。 多播组通过 D 类 IP 地址和标准 UDP 端口号指定。D 类 IP 地址在 224.0.0.0 和 239.255.255.255 的范围内（包括两者）。地址 224.0.0.0 被保留，不应使用。

#### 多播代码

```text
public class MultiBroadcast {

    public void startReveive(String host, int port) {

        new Thread(new Runnable() {

            MulticastSocket ms;

            private void receive(String host, int port) throws Exception{
                byte[] buf = new byte[1024 * 100];
                if(ms == null || ms.isClosed()) {
                    ms = new MulticastSocket(port);
                    DatagramPacket dp = new DatagramPacket(buf, buf.length);
                    InetAddress group = InetAddress.getByName(host);
                    ms.joinGroup(group);
                    ms.receive(dp);
                }

                Log.i("receiver", new String(buf));
                ms.close();
            }

            @Override
            public void run() {
                while (true) {
                    try {
                        receive(host, port);
                        Thread.sleep(100);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }

    public void send(String host, int port, String message) throws Exception{
        InetAddress group = InetAddress.getByName(host);
        MulticastSocket s = new MulticastSocket();
        s.joinGroup(group);
        DatagramPacket dp = new DatagramPacket(message.getBytes(),message.length(), group,port);
        s.send(dp);
        s.close();
    }

}
```

### UDP单播

```text
public class UnicastBroadcast {

    public void send(String host, int port, byte[] message) throws Exception{
        //sending data to host and its port
        InetAddress address = InetAddress.getByName(host);
        // Initialize a datagram packet with data and address
        DatagramPacket packet = new DatagramPacket(message, message.length, address, port);
        // Create a datagram socket, send the packet through it, close it.
        DatagramSocket dsocket = new DatagramSocket();
        dsocket.send(packet);
        dsocket.close();
    }

    public void startReceive() {
        new Thread(new Runnable() {

            DatagramSocket dsocket;

            @Override
            public void run() {

            }

            private void receive(int port) throws Exception{

                byte[] buffer = new byte[2048];
                if(dsocket == null || dsocket.isClosed()) {
                    //receiving data from its port
                    dsocket = new DatagramSocket(port);
                    // Create a buffer to read datagrams into. If a
                    // packet is larger than this buffer, the
                    // excess will simply be discarded!

                    // Create a packet to receive data into the buffer
                    DatagramPacket packet = new DatagramPacket(buffer, buffer.length);

                    // Wait to receive a datagram
                    dsocket.receive(packet);
                    // Convert the contents to a string, and display them
                    String msg = new String(buffer);
                    System.out.println(packet.getAddress().getHostName() + ": " + msg);
                }

                dsocket.close();

            }

        }).start();
    }

}
```

## url编码问题

url 标准为美国制定，只支持asc编码，所以 1. get 请求等需要传递汉子等非asc编码支持的字符时，违背的这个标准，需要url encode 2. 参数包含& = 等这些标准连接的特殊符号时，也需要url encode

## http（android api）

HttpURLConnection [代码demo](http://stackoverflow.com/questions/2793150/using-java-net-urlconnection-to-fire-and-handle-http-requests/2793153#2793153)

### 发送：

#### 1.设置协议：

con.setRequestMethod\("post"\), timeout 等。

#### 2.发送内容：

os = con.getOutPutStream\(\)。 os.writeBytes\(json data 数据\).toString\(\)

#### 3.发送流结构：

请求行（自动） 消息头（协议） 内容（数据）

### 接收：

1.con.getInputStream\(\); read\(\); 2.接收流结构 状态行（状态码） 消息头（协议） 内容（数据）

> 注：https 在上面的设置协议中加入： 1. setHostnameVerifier\( \); 2.构造器中加入 setDefaultSSLSocketFactory

## http（apache api）

### post发送：

```text
1.pos = new HttpPost（url）
2.pos.setHeader()   协议
3.pos.setEntity（） json数据
4.re = new HttpClient（）.execute（pos）
5.EntityUtils.toString(re.getEntity（）)
```

### get发送：

```text
1.get = new HttpGet（url）
2.get.setHeader()
3.re = new HttpClient().execute(get)
4. status = re.getStatusline().getStatusCode()
re.getEntity().getContent().
```

## https

![](.gitbook/assets/ssl_handshake_rsa.jpg)

## TCP/IP 网络模型

[TCP/IP Protocol Architecture Model](https://docs.oracle.com/cd/E19683-01/806-4075/ipov-10/index.html)

| OSI Ref. Layer No. | OSI Layer Equivalent | TCP/IP Layer | TCP/IP Protocol Examples |
| :--- | ---: | :---: | :---: |
| 5,6,7 | Application, session, presentation | Application | HTTP, NFS, NIS+, DNS, telnet, ftp, rlogin, rsh, rcp, RIP, RDISC, SNMP, and others |
| 4 | Transport | TransportTransport | TCP, UDP |
| 3 | Network | Internet | IP, ARP, ICMP |
| 2 | Data link | Data link | PPP, IEEE 802.2 |
| 1 | Physical | Physical network | Ethernet \(IEEE 802.3\) Token Ring, RS-232, others |


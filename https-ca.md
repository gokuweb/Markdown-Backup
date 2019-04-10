---
title: HTTPS、CA 以及自签名证书
categories:
  - Web安全
tags:
  - HTTPS
  - CA
dabbrlink: 2a108c2c
abbrlink: 873e23ee
ate: 2019-04-08 01:47:37
---
&#8195;&#8195;HTTPS 可以通过对数据加密来保障通信过程中的安全，这个过程看似简单实际涉及到了 CA 机构，根证书，中间证书，证书申请，证书签名等很多步骤，理解了这个过程我们就可以自签名证书，方便在内网中做 HTTPS 测试。

<!-- more -->

## HTTP 和 HTTPS 
### HTTP 的缺点
&#8195;&#8195;我们知道 HTTP 协议是以明文传输信息的，信息很容易被劫持和篡改，为了防止信息被劫持，我们决定对传输的信息用一个密码加密，为了保证这个密码的安全，每次连接中我们让客户端生成一个随机数作为密码，但是考虑到这个密码在发送给服务端的时候也可能被劫持，我们还得对这个随机数进行加密，但是如何才能做到只有服务端才能解密呢？最安全的莫过于非对称加密，用服务端公钥加密，私钥只在服务端手里，方案有了，只需要服务端把公钥发过来就行了，流程如下：
> 1. 客户端向服务端请求公钥；
> 2. 服务端把自己公钥发给客户端；
> 3. 客户端生成随机值，用服务端公钥加密发给服务端；
> 4. 服务端用自己私钥解密获取该随机值；
> 5. 服务端客户端的通信用该随机值加密。

但是这种方案并不完美，如图：
![MAN-IN-MIDDLE](/imgs/man-in-middle.png)

&#8195;&#8195;Client 把随机值发给 Server 时候是经过 Server 公钥加密的，但是 Client 获取 Server 公钥的时候， Server 是以明文将公钥发送给 Client 的，有可能被劫持，如上图 Server 发送公钥的过程中被劫持，公钥被劫持就无安全可言了，我们 **无法确保首次公钥交换过程的安全**，为了解决这个问题，就有了 HTTPS 协议。
&#8195;&#8195;HTTPS 基于 SSL/TLS 协议，引入 SSL 后主要对上面第二步产生影响，这一步服务端会发送一个证书到客户端，证书里有服务端的公钥，客户端会先验证证书，验证可信后才会使用其公钥，中间人无法伪造可信的证书，也就无法劫持公钥（浏览器发现不可信的证书，会提示“您的链接不安全”，如果用户主动添加信任的话，还是有证书被劫持的风险，在用 Burp Suite 抓 HTTPS 包时候需要事先把 Burp 的证书导入浏览器，也是这个原因），为什么伪造的证书无法通过验证？这就是 CA 的功劳，CA 是一个权威的第三方机构，经过 CA 认证的证书才会通过客户端浏览器的验证，浏览器怎么知道某个证书是否被 CA 认证过？浏览器和操作系统都会内置一些可信的根证书颁发机构，也就是说这些机构的权威性是由浏览器或操作系统保证的。

### HTTPS 协议
&#8195;&#8195; HTTPS = HTTP + SSL，简单说运行在 SSL/TLS 上的 HTTP 就是 HTTPS，**<font color="Red">SSL 协议依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密</font>**，所以 SSL 验证过程中会有证书校验的步骤，在下一章节会详细介绍证书，先来看看这几个协议在计算机网络的 OSI 七层模型中的位置：

| 层级 |    层名    | 常用协议 |
| ---- | ---------- | ----------- |
|  7   |   应用层   | **<font color="Red">HTTP/HTTPS</font>**、FTP、Socket、Telnet、SSH、SMTP、POP3、DHCP、DNS、NFS、SNMP |
|  6   |   表示层   | XDR、LPP |
|  5   |   会话层   | **<font color="Red">SSL/TLS</font>**、LDAP/DAP、RPC |
|  4   |   传输层   | **<font color="Red">TCP</font>**、UDP |
|  3   |   网络层   | **<font color="Red">IP</font>**、OSPF、ICMP |
|  2   | 数据链路层 | 以太网、令牌环、PPP、PPTP、L2TP、ARP、ATMP |
|  1   |   物理层   | 物理线路、光纤、无线电 |

> &#8195;&#8195;对于 SSL 属于哪一层网上争议挺多，有人说 SSL 协议并没有很好的对应 OSI 模型，所以具体属于哪一层没有严格的说法，会话层似乎更合适一些。

&#8195;&#8195;客户端执行 HTTPS 请求时，需要由 TCP 协议建立和释放连接，这就涉及 TCP 协议的三次握手和四次挥手：
**TCP 三次握手建立连接**
![Three-way Handshake](/imgs/three-way-handshake.png)

> 1. 客户端发送一个唯一的数据包（SYNJ）给服务器，请求建立连接；
> 2. 服务器收到客户端的请求后，生成一个 SYN J 的回应包（ack J+1）和一个新的数据包（SYN K），发给客户端；
> 3. 客户端收到服务器返回的两个包后，针对服务器的 SYN K 包生成一个回应包（ack K+1）,并发送给服务器。至此完成三次握手。

**TCP 四次挥手断开连接**
![Three-way Handshake](/imgs/four-way-handshake.png)

> 1. 客户端发送 FIN（M）报文给服务器，告诉对方，“我的数据发完了”；
> 2. 服务器收到 FIN（M）报文后，回给客户端一个 ack（M+1）报文，告诉客户端，“好，我知道了”；
> 3. 服务器发一个 FIN（N）报文给客户端，告诉对方，“我的数据也发完了”；
> 4. 客户端回应 ack（N+1）,告诉服务器，“好，我知道了”，至此，四次挥手断开连接。

### HTTPS 单向验证
&#8195;&#8195;对于 HTTP 而言，TCP 连接建立好后服务器就可以发数据给客户端了。但是对于 HTTPS，它还要运行 SSL/TLS 协议，SSL/TLS 协议分两层，第一层是记录协议，主要用于传输数据的加密压缩；第二层是握手协议，它建立在第一层协议之上，主要用于数据传输前的双方身份认证、协商加密算法、交换密钥。
&#8195;&#8195;HTTPS 验证过程就是 SSL 握手协议的交互过程。“HTTPS 验证”这个说法其实不准确的，应该是“SSL 验证”，过程如下：
**1. 客户端发起 ClientHello**
客户端向指定域名的服务器发起 HTTPS 请求，请求内容包括:
> 1. 客户端支持的 SSL/TLS 协议版本列表；
> 2. 支持的对称加密算法列表；
> 3. 客户端生成的随机数 A。

**2. 服务端回应 SeverHello**
服务器收到请求后，回应客户端，回应的内容主要有：
> 1. SSL/TLS 版本。服务器会在客户端支持的协议和服务器自己支持的协议中，选择双方都支持的 SSL/TLS 的最高版本，作为双方使用的 SSL/TLS 版本。如果客户端的 SSL/TLS 版本服务器都不支持，则不允许访问；
> 2. 与1类似，选择双方都支持的最安全的加密算法；
> 3. 从服务器密钥库中取出的证书；
> 4. 服务器端生成的随机数 B。

**3. 客户端回应**
客户端收到后，检查证书是否合法，主要检查下面 4 点：
> 1. 检查证书是否过期
> 2. 检查证书是否已经被吊销。有 CRL 和 OCSP 两种检查方法。CRL 即证书吊销列表，证书的属性里面会有一个 CRL 分发点属性，这个属性会包含了一个 url 地址，证书的签发机构会将被吊销的证书列表展现在这个 url 地址中；OCSP 是在线证书状态检查协议，客户端直接向证书签发机构发起查询请求以确认该证书是否有效。
> 3. 证书是否可信。浏览器会有一个信任库，里面保存了该客户端信任的 CA（证书签发机构）的证书，如果收到的证书签发机构不在信任库中，则浏览器会提示用户证书不可信。假如是 Java 程序，需要程序配置信任库文件，以判断证书是否可信，如果没设置，则默认使用 jdk 自带的证书库（jre\lib\security\cacerts，默认密码 changeit）。如果证书或签发机构的证书不在信任库中，则认为不安全，程序会报错。（你可以在程序中设置信任所有证书，不过这样并不安全）。
> 4. 检查收到的证书中的域名与请求的域名是否一致。若客户端是程序，这一项可配置不检查。若为浏览器，则会出现警告，用户也可以跳过。证书验证通过后，客户端使用特定的方法又生成一个随机数 c，这个随机数有专门的名称“pre-master key”。接着客户端会用证书的公钥对“pre-master key”加密，然后发给服务器。

**4. 服务器的最后回应**
&#8195;&#8195;服务器使用密钥库中的私钥解密后，得到这个随机数 c。此时，服务端和客户端都拿到了随机数 a、b、c ，双方通过这 3 个随机数使用相同的 **DH 密钥交换算法** 计算得到了相同的 **对称加密的密钥**。这个密钥就作为后续数据传输时对称加密使用的密钥。 
&#8195;&#8195;服务器回应客户端，握手结束，可以采用对称加密传输数据了。

**这里注意几点：**
1. 整个验证过程，折腾了半天，其实是为了安全地得到一个双方约定的对称加密密钥，当然，过程中也涉及一些身份认证过程。既然刚开始时，客户端已经拿到了证书，里面包含了非对称加密的公钥，为什么不直接使用非对称加密方案呢，这是因为非对称加密计算量大、比较耗时，而对称加密耗时少。

2. 为什么要用到 3 个随机数，1 个不行吗？这是因为客户端和服务端都不能保证自己的随机数是真正随机生成的，这样会导致数据传输使用的密钥就不是随机的，时间长了就很容易被破解。如果使用客户端随机数、服务端随机数、pre-master key 随机数这 3 个组合，就能十分接近随机。

### HTTPS 双向验证
&#8195;&#8195;单向验证过程中，客户端会验证自己访问的服务器，服务器对来访的客户端身份不做任何限制。如果服务器需要限制客户端的身份，则可以选择开启服务端验证，这就是双向验证。从这个过程中我们不难发现，**使用单向验证还是双向验证，是服务器决定的。**
&#8195;&#8195;一般而言，我们的服务器都是对所有客户端开放的，所以服务器默认都是使用单向验证。如果你使用的是 Tomcat 服务器，在配置文件 `server.xml` 中，配置 Connector 节点的 clientAuth 属性即可。若为 true 则使用双向验证，若为 false 则使用单向验证。如果你的服务只允许特定的客户端访问，那就需要使用双向验证了。

双向验证基本过程与单向验证相同，不同在于：

1. 第二步服务器第一次回应客户端的 SeverHello 消息中，会要求客户端提供“客户端的证书”;
2. 第三步客户端验证完服务器证书后的回应内容中，会增加两个信息：
> 1. 客户端的证书；
> 2. 客户端证书验证消息（CertificateVerify message）：客户端将之前所有收到的和发送的消息组合起来，并用 hash 算法得到一个 hash 值，然后用客户端密钥库的私钥对这个 hash 进行签名，这个签名就是 CertificateVerify message。 

3. 服务器收到客户端证书后：
> 1. 确认这个证书是否在自己的信任库中（当然也会校验是否过期等信息），如果验证不通过则会拒绝连接；
> 2. 用客户端证书中的公钥去验证收到的证书验证消息中的签名。这一步的作用是为了确认证书确实是客户端的。

**说明：**
&#8195;&#8195;关于第二步中客户端私钥的使用，网上有很多文章认为：在协商对称加密方案时，服务端先用客户端公钥加密服务器选定的对称加密方案，客户端收到后使用私钥解密得到。首先，对称加密方案就那么几种，逐个试试就能试出来，没必要为了这个增加一个客户端和服务端的交互过程。而这里关于 CertificateVerify message 的说法参考了维基百科关于“Transport Layer Security”一文中["Client-authenticated TLS handshake"](https://en.wikipedia.org/wiki/Transport_Layer_Security#Client-authenticated_TLS_handshake)的描述。

&#8195;&#8195;所以，在双向验证中，客户端需要用到密钥库，保存自己的私钥和证书，并且证书需要提前发给服务器，由务器放到它的信任库中。

## CA 和证书
### 什么是 CA 证书
　　CA（Certificate Authority），也叫“证书授权中心”，它是负责管理和签发证书的第三方机构，CA 证书顾名思义就是 CA 颁发的证书，平常说的 CA 证书应该是指 CA 根证书，但 CA 根证书并不会直接对申请者签名，而是通过中间证书进行签名，经过签名认证的证书，通常也叫 SSL 证书，当然具体怎么称呼还是看场景，CA 自签名的证书叫根证书；针对客户端/服务端分别有客户端证书、服务端证书；但是对于 CA 来说无论客户端/服务端都是其用户，可以统称用户证书。

### 证书的签发过程
1. 服务方 S 向第三方机构 CA 提交公钥、组织信息、个人信息(域名)等信息并申请认证；

2. CA 通过线上、线下等多种手段验证申请者提供信息的真实性，如组织是否存在、企业是否合法，是否拥有域名的所有权等；

3. 如信息审核通过，CA 会向申请者签发认证文件-证书。证书包含以下信息：申请者公钥、申请者的组织信息和个人信息、签发机构 CA 的信息、有效时间、证书序列号等信息的明文，同时包含一个签名；
签名的产生算法：首先，使用散列函数计算公开的明文信息的信息摘要，然后该散列函数和信息摘要经过 CA 的私钥进行加密，密文即是签名；

4. 客户端 C 向服务器 S 发出请求时，S 返回证书文件；

5. 客户端 C 读取证书中的相关的明文信息，然后利用对应 CA 的公钥解密签名数据，获取散列函数后用该函数计算证书中明文信息得到信息摘要，对比证书的信息摘要，如果一致，则可以确认证书的合法性，即公钥合法；

6. 客户端然后验证证书相关的域名信息、有效时间等信息；

7. 客户端会内置信任 CA 的证书信息(包含公钥)，如果CA不被信任，则找不到对应 CA 的证书，证书也会被判定非法。

证书格式如下：
![Certificate frame](/imgs/certificate-frame.png)

**在这个过程注意几点：**
1. 申请证书不需要提供私钥，确保私钥永远只能服务器掌握；

2. 证书的合法性仍然依赖于非对称加密算法，证书主要是增加了服务器信息以及签名；

3. 内置 CA 对应的证书称为根证书，颁发者和使用者相同，自己为自己签名，即自签名证书；

4. 证书 = 公钥 + 申请者与颁发者信息 + 签名。

### 证书链
证书是以证书链的形式存在的：
> 最上层为 root，也就是通常所说的 CA ，用来颁发证书； 
> 最下层为 end-user，对应每个网站购买使用的证书；
> 中间一层为 intermediates，是二级 CA ，这一层可以继续划分为多层，用来帮助 root 给 end-user 颁发证书，这样 root 只需向 intermediates 颁发证书；
> 只有当整个证书链上的证书都有效时，才会认定当前证书合法 。


### 中间证书
&#8195;&#8195;中间层 intermediates 实际上是一个中间证书，其实也叫中间 CA（中间证书颁发机构，Intermediate certificate authority, Intermedia CA），对应的是根证书颁发机构（Root certificate authority ，Root CA）。为了验证证书是否可信，必须确保证书的颁发机构在设备的可信 CA 中。如果证书不是由可信 CA 签发，则会检查颁发这个 CA 证书的上层 CA 证书是否是可信 CA ，客户端将重复这个步骤，直到证明找到了可信 CA（将允许建立可信连接）或者证明没有可信 CA（将提示错误）。

&#8195;&#8195;为了构建信任链，每个证书都包括字段：“使用者”和“颁发者”。 中间 CA 将在这两个字段中显示不同的信息，显示设备如何获得下一个 CA 证书，重复检查是否是可信 CA 。

&#8195;&#8195;根证书，必然是一个自签名的证书，“使用者”和“颁发者”都是相同的，所以不会进一步向下检查，如果根 CA 不是可信 CA ，则将不允许建立可信连接，并提示错误。

例如：一个服务器证书 domain.com，是由 Intermedia CA 签发，而 Intermedia CA 的颁发者 Root CA 在 WEB 浏览器可信 CA 列表中，则证书的信任链如下：
> 证书 1 - 使用者：domain.com；颁发者：Intermedia CA
> 证书 2 - 使用者：Intermedia CA；颁发者： Root CA
> 证书 3 - 使用者：Root CA ； 颁发者： Root CA

&#8195;&#8195;当 Web 浏览器验证到 Root CA 时，发现是一个可信 CA ，则完成验证，允许建立可信连接。当然有些情况下，Intermedia CA 也在可信 CA 列表中，这个时候就可以直接完成验证，建立可信连接。

**为何需要中间证书**
> 1. 保护根证书。如果直接采用根证书签发证书，一旦发生根证书泄露，将造成极大的安全问题。所以目前根证书都要求离线保存，如果需要用根证书签名，则必须通过人手工方式，直接用根证书在线签发证书是不允许的。
> 2. 区分不同类型的产品。针对 DV、OV、EV 等不同类型，不同安全级别的证书，CA 会采用不同的根证书，一来便于区分，二来一旦出现问题，也便于区别处理，降低影响。中间 CA 证书一般都是支持在线签发证书的。
> 3. 交叉验证。为了获得更好的兼容性，支持一些很古老的浏览器，有些根证书本身，也会被另外一个很古老的根证书签名，这样根据浏览器的版本，可能会看到三层或者是四层的证书链结构，如果能看到四层的证书链结构，则说明浏览器的版本很老，只能通过最早的根证书来识别。


### 证书验证流程
如图：
![certificate-verify](/imgs/certificate-verify.png)

> 1. 客户端获取到了站点证书，拿到了站点的公钥；
> 2. 要验证站点可信后，才能使用其公钥，因此客户端找到其站点证书颁发者的信息；
> 3. 站点证书的颁发者验证了服务端站点是可信的，但客户端依然不清楚该颁发者是否可信；
> 4. 再往上回溯，找到了认证了中间证书商的源头证书颁发者。由于源头的证书颁发者非常少，我们浏览器之前就认识了，因此可以认为根证书颁发者是可信的；
> 5. 一路倒推，证书颁发者可信，那么它所颁发的所有站点也是可信的，最终确定了我们所访问的服务端是可信的；
> 6. 客户端使用证书中的公钥，继续完成TLS的握手过程。

&#8195;&#8195;那么，客户端是是如何验证某个证书的有效性，或者验证策略是怎样的?证书颁发者一般提供两种方式来验证证书的有效性：
**CRL**
&#8195;&#8195;CRL（Certificate Revocation List）即 证书撤销名单。证书颁发者会提供一份已经失效证书的名单，供浏览器验证证书使用。当然这份名单是巨长无比的，浏览器不可能每次TLS都去下载，所以常用的做法是浏览器会缓存这份名单，定期做后台更新。这样虽然后台更新存在时间间隔，证书失效不实时，但一般也OK。

**OCSP**
&#8195;&#8195;OCSP(Online Certificate StatusProtocol)即 在线证书状态协议。除了离线文件，证书颁发者也会提供实时的查询接口，查询某个特定证书目前是否有效。实时查询的问题在于浏览器需要等待这个查询结束才能继续TLS握手，延迟会更大。

### 总结
**为什么需要 CA **
&#8195;&#8195; HTTPS 要使客户端与服务器端的通信过程得到安全保证，必须使用的对称加密算法，但是协商对称加密算法的过程，需要使用非对称加密算法来保证安全，然而直接使用非对称加密的过程本身也不安全，会有中间人篡改公钥的可能性，所以客户端与服务器不直接使用公钥，而是使用数字证书签发机构颁发的证书来保证非对称加密过程本身的安全。这样通过这些机制协商出一个对称加密算法，就此双方使用该算法进行加密解密，从而解决了客户端与服务器端之间的通信安全问题。

**浏览器证书校验大致过程：**
> 1. 首先证书包括证书机构、有效期、服务器公钥等一些详细信息，以及一个签名。把前面详细信息经过 hash 得到一个字串，这个字串和 hash 算法经过 CA 私钥加密就是前面提到的签名了；
> 2. 浏览器读取证书中的证书所有者、有效期等信息进行一一校验；
> 3. 浏览器开始查找操作系统中已内置的受信任的证书发布机构CA，与服务器发来的证书中的颁发者CA比对，用于校验证书是否为合法机构颁发；
> 4. 如果找不到，浏览器就会报错，说明服务器发来的证书是不可信任的；
> 5. 如果找到，那么浏览器就会从操作系统中取出颁发者CA  的公钥，然后对服务器发来的证书里面的签名进行解密，解密后得到一个 hash 算法，浏览器使用该算法对证书中详细信息计算 hash 值，将这个 hash 值与签名中的 hash 值做对比；
> 6. 对比结果一致，则证明服务器发来的证书合法，没有被冒充；
> 7. 此时浏览器就可以读取证书中的公钥，用于后续加密了。

**用于对称加密的 key 的生成过程：**
> 1. 客户端发送 ClientHello 消息、支持的加密算法和生成的一个随机数 random1 到服务器端（Say Hello)；
> 2. 服务端会在客户端加密算法中选择一个自己也支持的加密算法作为后面通信的加密算法、生成一个随机数 random2，还有服务器证书（包含公钥）一起回馈给客户端（I got it）；
> 3. 证书验证通过后，客户端生成随机数 random3 ，通过 server 的公钥加密并传递给服务端；
> 4. 服务器端使用私钥解密，拿到 random3；
> 5. 这三个随机数通过某种算法算出 session key。

## 自签名证书
&#8195;&#8195;比如在内网的测试环境中，为了验证 HTTPS 下的一些问题，我们没必要申请昂贵的证书，这个时候就可以自签名一个证书，**自签名证书步骤如下：**
> 1. 创建 根证书（Root CA，用来给中间证书签名） ；
> 2. 创建 中间证书（Intermediate CA，用来给用户签名）；
> 3. 创建 服务端/客户端证书，使用 Intermediate CA 签名。

**生成证书步骤如下：**
> 1. 生成私钥（.key）。要保证私钥只有客户端自己拥有；
> 2. 生成证书请求文件（.csr）。以客户端的密钥和客户端自身的信息(国家、机构、域名、邮箱等)为输入，生成证书请求文件。其中客户端的公钥和客户端信息是明文保存在证书请求文件中的，而客户端私钥的作用是对客户端公钥及客户端信息做签名，自身是不包含在证书请求中的，然后把证书请求文件发送给 CA 机构。
> 3. 生成客户端证书（.crt）。CA机构接收到客户端的证书请求文件后，首先校验其签名，然后审核客户端的信息，最后 CA 机构使用自己的私钥为证书请求文件签名，生成证书文件，下发给客户端，此证书就是客户端的身份证，来表明用户的身份。中间证书需要根证书的私钥签名，其他网站的证书一般需要中间证书的签名。

### 最偷懒的方式
临时测试 HTTPS 的话可以省略很多步骤，比如根 CA 和中间证书都可以不要：
```
#生成私钥
$ openssl genrsa -out server.key 2048

#生成证书请求文件
$ openssl req -new -key server.key -out server.csr

#生成自签名证书
$ openssl x509 -req -in server.csr -signkey server.key -out server.crt
```
在 nginx 里面配置好 `server.csr` 和 `server.crt` 就可以使用了。

### 稍微标准点的方式
没用中间证书而是直接用根证书对用户签名认证。

**生成 CA 根证书**
生成 CA 私钥（.key）-->生成 CA 证书请求（.csr）-->自签名得到根证书（.crt）（ CA 给自已颁发的证书）。
```
# Generate CA private key 
$ openssl genrsa -out ca.key 2048 

# Generate CSR 
$ openssl req -new -key ca.key -out ca.csr

# Generate Self Signed certificate（CA 根证书）
$ openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```

**生成用户证书**
生成私钥（.key）-->生成证书请求（.csr）-->用CA根证书签名得到证书（.crt）

服务端用户证书
```
# private key
$ $openssl genrsa -des3 -out server.key 2048 

# generate csr
$ openssl req -new -key server.key -out server.csr

# generate certificate
$ openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key 
```

客户端用户证书
```
$ openssl genrsa -des3 -out client.key 1024 

$ openssl req -new -key client.key -out client.csr

$ openssl ca -in client.csr -out client.crt -cert ca.crt -keyfile ca.key
```

**生成 pem 格式证书（可选）**
有时需要用到 pem 格式的证书，可以用以下方式合并证书文件（crt）和私钥文件（key）来生成： 
```
$cat client.crt client.key> client.pem 
$cat server.crt server.key > server.pem
```

结果：
服务端证书：ca.crt，server.key，server.crt，server.pem
客户端证书：ca.crt，client.key，client.crt，client.pem

### 标准方式
**创建 CA 根证书（root pair）**
&#8195;&#8195;扮演 CA 角色就意味着要管理大量的 pair 对，而原始的一对 pair 对叫做 root pair，它包含了 root key（ca.key.pen）和 root certificate（ca.cert.pem）。通常情况下，root CA 不会直接为服务器或者客户端签证，它们会先为自己生成几个中间 CA（intermediate CAs），这几个中间 CA 作为 root CA 的代表为服务器和客户端签证。
> 注意：一定要在绝对安全的环境下创建 root pair，可以断开网络、拔掉网线和网卡，当然，如果是测试玩一玩就不用这么认真了。

1. 设定文件夹结构，并且配置好 openssl 设置：
```
$ cd /root/ca
$ mkdir certs crl newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
$ wget -O /root/ca/openssl.cnf \ 
    //raw.githubusercontent.com/barretlee/autocreate-ca/master/cnf/root-ca
```
&#8195;&#8195;签名证书时候（也就是生成最终证书时候）`openssl` 的 `ca` 参数会读取 `index.txt` 和 `serial` 文件，它们分别是证书缩影文件数据库和证书序列号文件，这里预选创建好以防报错，具体配置文件详见引申部分 [/root/ca/openssl.cnf](#ca-openssl)

2. 生成 CA 私钥（root key），密码可为空，设定权限为只可读：
```
$ cd /root/ca
$ openssl genrsa -aes256 -out private/ca.key.pem 4096

Enter pass phrase for ca.key.pem: secretpassword
Verifying - Enter pass phrase for ca.key.pem: secretpassword

$ chmod 400 private/ca.key.pem
```

3. 生成 CA 自签名证书（root cert），权限设置为可读：
```
$ cd /root/ca
$ openssl req -config openssl.cnf \
      -key private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out certs/ca.cert.pem
$ chmod 444 certs/ca.cert.pem	  
  
```
4. 验证证书：
```
$ openssl x509 -noout -text -in certs/ca.cert.pem
```

正确的输出应该是这样的：
```
 Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            87:e8:c0:a0:4b:e2:12:5d
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=Zhejiang, O=Barret Lee, OU=Barret Lee Certificate Authority, CN=Barret Lee Root CA
        Validity
            Not Before: Apr 23 05:46:36 2016 GMT
            Not After : Apr 18 05:46:36 2036 GMT
        Subject: C=CN, ST=Zhejiang, O=Barret Lee, OU=Barret Lee Certificate Authority, CN=Barret Lee Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (4096 bit)
                Modulus (4096 bit):
                    // ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                E5:2D:B8:2B:DC:88:FE:CE:DA:93:D8:6F:2E:74:04:D2:39:E7:C8:03
            X509v3 Authority Key Identifier:
                keyid:E5:2D:B8:2B:DC:88:FE:CE:DA:93:D8:6F:2E:74:04:D2:39:E7:C8:03

            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
    Signature Algorithm: sha256WithRSAEncryption
        // ...
```
包含：
> 数字签名（Signature Algorithm）
> 有效时间（Validity）
> 主体（Issuer）
> 公钥（Public Key）
> X509v3 扩展，openssl config 中配置了 v3_ca，所以会生成此项

**创建 Intermediate 证书**
&#8195;&#8195;目前我们已经拥有了 Root Pair，事实上已经可以用于证书的发放了，但是由于根证书很干净，特别容易被污染，所以我们需要创建中间 pair 作为 root pair 的代理，生成过程同上，只是细节略微不一样。

1. 生成目录结构和 openssl 的配置，这里的配置是针对 intermediate pair 的：
```
$ mkdir /root/ca/intermediate
$ cd /root/ca/intermediate
$ mkdir certs crl csr newcerts private
$ chmod 700 private
$ touch index.txt
$ echo 1000 > serial
$ echo 1000 > /root/ca/intermediate/crlnumber
$ wget -O /root/ca/intermediate/openssl.cnf \
    //raw.githubusercontent.com/barretlee/autocreate-ca/master/cnf/intermediate-ca
```
配置文件详见引申部分 [/root/ca/intermediate/openssl.cnf](#intermediate-openssl)

2. 生成 intermediate 秘钥（intermediate key），密码可为空，设定权限为只可读：
```
$ cd /root/ca
$ openssl genrsa -aes256 \
      -out intermediate/private/intermediate.key.pem 4096

Enter pass phrase for intermediate.key.pem: secretpassword
Verifying - Enter pass phrase for intermediate.key.pem: secretpassword

$ chmod 400 intermediate/private/intermediate.key.pem
```

3. 生成 intermediate 证书请求文件（intermediate csr），设定权限为只可读，这里需要特别注意的一点是 Common Name 不要与 root pair 的一样 ：
```
$ cd /root/ca
$ openssl req -config intermediate/openssl.cnf -new -sha256 \
      -key intermediate/private/intermediate.key.pem \
      -out intermediate/csr/intermediate.csr.pem

Enter pass phrase for intermediate.key.pem: secretpassword
You are about to be asked to enter information that will be incorporated
into your certificate request.

Country Name (2 letter code) [XX]:CN
State or Province Name []:Zhejiang
Locality Name []:
Organization Name []:Barret Lee
Organizational Unit Name []:Barret Lee Certificate Authority
Common Name []:Barret Lee Intermediate CA
Email Address []:
```

4. 生成 intermediate 证书，用 CA 根证书签名得到 intermediate 证书，使用 v3_intermediate_ca 扩展签名，密码可为空，中间 pair 的有效时间一定要为 root pair 的子集：
```
$ cd /root/ca
$ openssl ca -config openssl.cnf -extensions v3_intermediate_ca \
      -days 3650 -notext -md sha256 \
      -in intermediate/csr/intermediate.csr.pem \
      -out intermediate/certs/intermediate.cert.pem

Enter pass phrase for ca.key.pem: secretpassword
Sign the certificate? [y/n]: y

$ chmod 444 intermediate/certs/intermediate.cert.pem
```
此时 root 的 index.txt 中将会多出这么一条记录：

> V 260421055318Z   1000  unknown .../CN=Barret Lee Intermediate CA

4. 验证中间 pair 的正确性：
```
$ openssl x509 -noout -text \
      -in intermediate/certs/intermediate.cert.pem
$ openssl verify -CAfile certs/ca.cert.pem \
      intermediate/certs/intermediate.cert.pem

intermediate.cert.pem: OK
```

5. 打包证书，浏览器在验证中间证书的时候，同时也会去验证它的上一级证书是否靠谱，创建证书链，将 root cert 和 intermediate cert 合并到一起，可以让浏览器一并验证：
```
$ cat intermediate/certs/intermediate.cert.pem \
      certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
$ chmod 444 intermediate/certs/ca-chain.cert.pem	  
```

**创建服务器/客户端证书**
&#8195;&#8195;终于到了这一步，生成我们服务器上需要部署的内容，上面已经解释了为啥需要创建中间证书。root pair 和 intermediate pair 使用的都是 4096 位的加密方式，一般情况下服务器/客户端证书的过期时间为一年，所以可以安全地使用 2048 位的加密方式。

1. 生成 www.barretlee.com 的私钥：
```
$ cd /root/ca
$ openssl genrsa -aes256 \
      -out intermediate/private/www.barretlee.com.key.pem 2048
$ chmod 400 intermediate/private/www.barretlee.com.key.pem
```

2. 生成 www.barretlee.com 的证书请求文件：
```
$ cd /root/ca
$ openssl req -config intermediate/openssl.cnf \
      -key intermediate/private/www.barretlee.com.key.pem \
      -new -sha256 -out intermediate/csr/www.barretlee.com.csr.pem

Enter pass phrase for www.barretlee.com.key.pem: secretpassword
You are about to be asked to enter information that will be incorporated
into your certificate request.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name []:Zhejiang
Locality Name []:Hangzhou
Organization Name []:Barret Lee
Organizational Unit Name []:Barret Lee's Personal Website
Common Name []:www.barretlee.com
Email Address []:barret.china@gmail.com
```

3. 生成 www.barretlee.com 的证书，使用 intermediate pair 签名上面证书：
```
$ cd /root/ca
$ openssl ca -config intermediate/openssl.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in intermediate/csr/www.barretlee.com.csr.pem \
      -out intermediate/certs/www.barretlee.com.cert.pem
$ chmod 444 intermediate/certs/www.barretlee.com.cert.pem
```

可以看到 /root/ca/intermediate/index.txt 中多了一条记录：
> V 170503055941Z   1000  unknown .../emailAddress=barret.china@gmail.com

4. 验证证书：
```
$ openssl x509 -noout -text \
      -in intermediate/certs/www.barretlee.com.cert.pem
$ openssl verify -CAfile intermediate/certs/ca-chain.cert.pem \
      intermediate/certs/www.barretlee.com.cert.pem

www.barretlee.com.cert.pem: OK
```

此时我们已经拿到了几个用于部署的文件，在 nginx 中配好路径就可以进行测试了：
```
ca-chain.cert.pem
www.barretlee.com.key.pem
www.barretlee.com.cert.pem
```

上面我们在生成证书的时候调用了 `openssl.cnf` 配置文件，我们也可以手动指定 CA 私钥和证书来进行签名，形如：
```
# 向自己的 CA 机构申请证书，签名过程需要 CA 的证书和私钥参与，最终颁发一个带有 CA 签名的证书
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt
```


## 引申
### req 主要参数说明
req 的基本功能主要有两个：生成证书请求和生成自签名证书。其他还有一些校验、查看请求文件等功能，示例会简单说明下。参数说明如下：
**[new/x509]**
&#8195;&#8195;当使用 `-new` 选取的时候，说明是要生成证书请求，当使用 `x509` 选项的时候，说明是要生成自签名证书。

**[key/newkey/keyout]**
&#8195;&#8195;`key` 和 `newkey` 是互斥的，`key` 是指定已有的密钥文件，而 `newkey` 是指在生成证书请求或者自签名证书的时候自动生成密钥，然后生成的密钥名称有 `keyout` 参数指定。
&#8195;&#8195;当指定 `newkey` 选项时，后面指定 `rsa:bits` 说明产生 rsa 密钥，位数由 bits 指定。指定 `dsa:file` 说明产生 dsa 密钥，file 是指生成 dsa 密钥的参数文件(由dsaparam生成)

**[nodes/days]**
&#8195;&#8195;`nodes` 告诉 OpenSSL 生产证书时忽略密码环节，(因为我们需要 Nginx 自动读取这个文件，而不是以用户交互的形式)。`days` 指定证书有效期。

**[in/out/inform/outform/keyform]**
&#8195;&#8195;`in` 选项指定证书请求文件，当查看证书请求内容或者生成自签名证书的时候使用 `out` 选项指定证书请求或者自签名证书文件名，或者公钥文件名（当使用 pubkey 选项时用到），以及其他一些输出信息。
&#8195;&#8195;`inform`、`outform`、`keyform` 分别指定了 `in`、`out`、`key` 选项指定的文件格式，默认是 PEM 格式。

**[config]**
&#8195;&#8195;参数文件，默认是 `/etc/ssl/openssl.cnf(ubuntu12.04)`，根据系统不同位置不同。该文件包含生成 req 时的参数，当在命令行没有指定时，则采用该文件中的默认值。

除上述主要参数外，还有许多其他的参数，不在一一叙述，有兴趣的读者可以查看 req 的 man 手册

### 什么是x509证书
X509 其实就是证书的一种规范，严格说与文件扩展名也没有直接关系，详细参考[维基百科 x.509](https://zh.wikipedia.org/wiki/X.509)：
> &#8195;&#8195;X.509 是密码学里公钥证书的格式标准。 X.509 证书己应用在包括 TLS/SSL 在内的众多 Intenet 协议里，同时它也用在很多非在线应用场景里，比如电子签名服务。X.509 证书里含有公钥、身份信息（比如网络主机名，组织的名称或个体名称等）和签名信息（可以是证书签发机构 CA 的签名，也可以是自签名）。对于一份经由可信的证书签发机构签名或者可以通过其它方式验证的证书，证书的拥有者就可以用证书及相应的私钥来创建安全的通信，对文档进行数字签名.
> &#8195;&#8195;另外除了证书本身功能，X.509 还附带了证书吊销列表和用于从最终对证书进行签名的证书签发机构直到最终可信点为止的证书合法性验证算法。

x509 证书一般会用到三种类型的文件：key、csr、crt。
`key` 是服务器上的私钥文件，用于对发送给客户端数据的加密，以及对从客户端接收到数据的解密。
`csr` 是证书签名请求文件，用于提交给证书颁发机构（CA）对证书签名。
`crt` 是证书文件，由证书颁发机构（CA）签名或者自签名的证书，包含证书持有人的信息，持有人的公钥，以及签署者的签名等信息，签署人用自己的 key 给你签署的凭证。

### 证书格式和编码方式
**证书格式：**
> x509 ，这种证书只有公钥，不包含私钥。
> pcks#7 ，这种主要是用于签名或者加密。
> pcks#12 ，这种含有私钥，同时也含有公钥，但是有口令保护。

**编码方式：**
> .pem 后缀的证书都是 base64 编码。
> .der 后缀的证书都是二进制格式。

&#8195;&#8195;x509 是数字证书的规范。PKCS#7 和 PKCS#12 是两种封装形式。
&#8195;&#8195;P7 一般是把证书分成两个文件，一个公钥一个私钥，有 PEM 和 DER 两种编码方式。**PEM 是纯文本的 base64 编码**，P7 一般是分发公钥用，看到的就是一串可见字符串，扩展名经常是 `.crt`、`.cer`、`.key` 等。**DER 是二进制编码**。
&#8195;&#8195;P12是把证书压成一个文件，`.pfx` 。主要是考虑分发证书，私钥是要绝对保密的，不能随便以文本方式散播。所以 P7 格式不适合分发。`.pfx` 中可以加密码保护，所以相对安全些。
&#8195;&#8195;在实践中要中，用户证书都是放在 USB Key 中分发，服务器证书经常还是以文件方式分发。服务器证书和用户证书都是X509证书，就是里面的属性有区别。

**格式转换**
1. crt + key 转 pfx【pfx是证书安装包，方便在电脑上直接双击按向导安装证书】
```
$ openssl pkcs12 -export -in client.crt -inkey client.key -out client.pfx
```

2. pfx 转化为 pem【curl 需要 pem 格式文件】
```
$ openssl pkcs12 -in client.pfx -nodes -out client.pem
```
3. crt + key转 p12【apache 的 cxf 客户端支持 jks 和 p12 证书】
```
$ openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
```

4. crt 转 jks【jks 支持存放信任证书，而 pkcs12 不支持，所以 tomcat 环境下配置 ca 只能使用 jks 才能保证 ca 密钥不泄露】
```
$ keytool -import -v -trustcacerts -storepass defaultpwd -keypass defaultpwd -file ca.crt -keystore ca_only.jks
```

### TSL 和 SSL
&#8195;&#8195;其实 TLS 就是从 SSL 发展而来的，只是 SSL 发展到 3.0 版本后改成了 TLS，版本演进大体为 SSL 2.0 -> SSL 3.0 -> TLS 1.0（可以看做是SSL 3.1版）。TLS 主要提供三个基本服务：
> 加密
> 身份验证，也可以叫证书验证吧
> 消息完整性校验

### 证书类型
&#8195;&#8195;能够受浏览器默认信任的 CA 大厂商有很多，Verisign 是第一家 CA 厂商，创办于 1995 年，当时得到了 RSA 算法的使用授权，是全世界最大的 CA 厂商，在 2010 年以 12.8 亿美元卖给了赛门铁克。当前 TOP5 的 CA 厂商分别是 Symantec、Comodo、Godaddy、GlobalSign 和 Digicert，占据了 90% 以上的市场份额。

![dv-ov-ev](/imgs/dv-ov-ev.png)

&#8195;&#8195;需要强调的是，不论是 DV、OV 还是 EV 证书，其加密效果都是一样的！ 它们的区别在于：
&#8195;&#8195;DV（Domain Validation），面向个体用户，安全体系相对较弱，验证方式就是向 whois 信息中的邮箱发送邮件，按照邮件内容进行验证即可通过；
&#8195;&#8195;OV（Organization Validation），面向企业用户，证书在 DV 证书验证的基础上，还需要公司的授权，CA 通过拨打信息库中公司的电话来确认；
&#8195;&#8195;EV（Extended Validation），打开 Github 的网页，你会看到 URL 地址栏展示了注册公司的信息，这会让用户产生更大的信任，这类证书的申请除了以上两个确认外，还需要公司提供金融机构的开户许可证，要求十分严格。
&#8195;&#8195;OV 和 EV 证书相当昂贵，使用方可以为这些颁发出来的证书买保险，一旦 CA 提供的证书出现问题，一张证书的赔偿金可以达到 100w 刀以上。

### openssl.cnf
补充一下生成 CA 和 Intermediate 证书的两个配置文件：
<span id="ca-openssl"><B>CA openssl.cnf</B></span>
```
# OpenSSL root CA configuration file.
# Copy to `/root/ca/openssl.cnf`.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = GB
stateOrProvinceName_default     = England
localityName_default            =
0.organizationName_default      = Alice Ltd
organizationalUnitName_default  =
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

<span id="intermediate-openssl"><B>intermediate openssl.cnf</B></span>
```
# OpenSSL intermediate CA configuration file.
# Copy to `/root/ca/intermediate/openssl.cnf`.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca/intermediate
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = GB
stateOrProvinceName_default     = England
localityName_default            =
0.organizationName_default      = Alice Ltd
organizationalUnitName_default  =
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

### 部分参考图片
生成了 pem 格式的根证书，自签名证书，所以无需生成证书请求文件：
![root-ca](/imgs/root-ca.png)

生成了客户端证书请求文件：
![clint-csr](/imgs/client-csr.png)

生成了客户端证书，使用了默认的配置文件 `openssl.cnf`：
![client-crt](/imgs/client-crt.png)

## 参考
[HTTPS原理和CA证书申请（满满的干货）](https://blog.51cto.com/11883699/2160032?source=dra)
[HTTPS实战之单向验证和双向验证](https://mp.weixin.qq.com/s/UiGEzXoCn3F66NRz_T9crA)
[Https单向认证和双向认证](https://blog.csdn.net/duanbokan/article/details/50847612)

[CA证书扫盲，https讲解。](http://www.cnblogs.com/handsomeBoys/p/6556336.html)
[浅析HTTPS与SSL原理](https://blog.csdn.net/tengxy_cloud/article/details/52808163)
[https之证书验证](https://blog.csdn.net/u012852986/article/details/78873387)
[中间证书的使用](https://www.myssl.cn/home/article-0406-42.html)
[HTTPS加密过程和TLS证书验证](https://juejin.im/post/5a4f4884518825732b19a3ce)
[HTTPS 通信过程详解（含CA证书验证）](https://www.jianshu.com/p/031fb34a05bd)

[给nginx创建个自签名SSL证书](http://blog.harrisonxi.com/2017/02/%E7%BB%99nginx%E5%88%9B%E5%BB%BA%E4%B8%AA%E8%87%AA%E7%AD%BE%E5%90%8DSSL%E8%AF%81%E4%B9%A6.html)
[细说 CA 和证书](https://www.barretlee.com/blog/2016/04/24/detail-about-ca-and-certs/)
[openssl生成SSL证书的流程](https://blog.csdn.net/liuchunming033/article/details/48470575)
[基于OpenSSL自建CA和颁发SSL证书](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)

[openssl 证书请求和自签名命令 req 详解](https://linux.cn/article-7248-1.html)
[x509和pkcs12以及pkcs7的关系](https://bbs.csdn.net/topics/190044123)
[总结之：CentOS6.5下openssl加密解密及CA自签颁发证书详解](https://blog.51cto.com/tanxw/1379417)



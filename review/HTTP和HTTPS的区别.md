#iOS笔记/复习笔记
---

## 一、HTTP和HTTPS的基本概念
* HTTP：超文本传输协议；是互联网上应用最为广泛的一种网络协议，是一个客户端和服务端请求和应答的标准（TCP），用于WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使用网络传输减少。
* HTTPS：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。
HTTPS协议的主要作用分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的真实性。
## HTTP与HTTPS有什么区别
HTTP协议传输的数据都是未加密的，也就是明文的，因此使用HTTP协议传输隐私信息非常不安全，为了保证这些隐私数据能加密传输，于是网景公司设计了SSL（Secure Sockets Layer）协议用于对HTTP协议传输的数据进行加密，从而就诞生了HTTPS。

HTTP与HTTPS有什么区别主要如下：
1. https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用；
2. http是超文本传输协议，信息是铭文传输，https则是具有安全性的ssl加密传输协议。
3. http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443.
4. http的连接很简单，是无状态的；https协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比https协议安全。

## HTTPS的工作原理
1. Client发起一个HTTPS（比如https://juejin.im/user/4283353031252967）的请求，根据RFC2818的规定，Client知道需要连接Server的443（默认）端口。
2. Server把事先配置好的公钥证书（public key certificate）返回给客户端。
3. Client验证公钥证书：比如是否在有效期内，证书的用途是不是匹配Client请求的站点，是不是在CRL吊销列表里面，它的上一级证书是否有效，这是一个递归的过程，直到验证到根证书（操作系统内置的Root证书或者Client内置的Root证书）。如果验证通过则继续，不通过则显示警告信息。
4. Client使用伪随机数生成器生成加密所使用的对称密钥，然后用证书的公钥加密这个对称密钥，发给Server。
5. Server使用自己的私钥（private key）解密这个消息，得到对称密钥。至此，Client和Server双方都持有了相同的对称密钥。
6. Server使用对称密钥加密“明文内容A”，发送给Client。
7. Client使用对称密钥解密响应的密文，得到“明文内容A”。
8. Client再次发起HTTPS的请求，使用对称密钥加密请求的“明文内容B”，然后Server使用对称密钥解密密文，得到“明文内容B”。


![5880561C-06E9-423F-8AF5-EBC293223306.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba9d0375ce3a4fac9e5c10c8d10f81d3~tplv-k3u1fbpfcp-watermark.image?)





[深入理解HTTPS工作原理](https://juejin.cn/post/6844903830916694030)
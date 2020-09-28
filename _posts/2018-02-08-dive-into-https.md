---
layout: post
title: "深入浅出 HTTPS"
tags: 
- 编程技术
---

## Why HTTPS?

以访问 google.com 为例，用上 HTTPS 后：

* Identity：你能确定你是在跟 google.com 在对话，而不是其他伪装成 google 的恶意网站
* Confidentiality：通信过程中，双方发送的内容都不会被人窃取、偷窥（eavesdropping）
* Integrity：中间人无法修改你与 google.com 互相发送的数据（tampering）

HTTPS 可以解决的一些常见问题：

* 保证用户的隐私数据（如密码、信用卡等）不被中间人窃取
* 避免 ISP 向你的网站注入广告等

浏览器中一些高级的 API，只能在 HTTPS 网站上使用，如 Geography、push notifications。HTTP2 要求有 HTTPS。

<!--more-->

## 历史

HTTPS 虽然叫 HTTP over SSL (Secure Sockets Layer)，但是现在看到的都是 TLS 了。SSL 是 Netscape 在 90 年代中期发明的，但是 Netscape 衰落后 SSL 交给了 IETF，IETF 在 1999 年发布了 TLS，作为 SSL 的继任者。现在（2018 年 1 月），TLS 1.2 是主流的版本，而 1.3 版本还在起草阶段。

## HTTPS 101

对于 HTTPS 的入门知识，HTTP: The Definitive Guide 的第 14 章提供了非常好的文字和插图。

由于计算非对称加密所消耗的 CPU 比对称加密多得多，因此 HTTPS 选择使用非对称加密来建立通信密钥，使用对称加密来加密传输的数据。

简单的描述即是：

1. 客户端跟服务器，先用非对称加密协商一个通信密钥
   * 非对称加密过程，客户端用服务器证书中的公钥来进行加解密
2. 客户端跟服务器再用之前协商的密钥，以对称加密的形式传输 HTTP 数据

下面描述相对完整的过程。一个完整握手（full handshake）的时序图如下：

![full-tls-handshake]({{ site.image_cdn }}/images/2018/02/full-tls-handshake.png)

1. ClientHello 时，client 告诉服务器它支持的 TLS 版本、Cipher Suites、SNI（Server Name Indication，后面详述）等信息
2. ServerHello 时，server 选择一个 TLS 版本和 chiper suite，并发送证书给 client
3. ClientKeyExchange 时，client 校验证书（看 hash、域名、过期时间等），然后随机生成一个 pre-master key，用证书中的公钥进行加密，发给 server。client 和 server 都可以根据这个 pre-master key 来生成一个 session key，用于加密 HTTP 流量
4. ChangeCipherSpec 时，client 和 server 互相表示，后面数据用 session key 进行对称加密后传输
5. 最后，两边开始传输加密的数据

值得注意的是，ChangeCipherSpec 之前的数据均是明文传输（比如 ClientHello 和 ServerHello）。

P.S. 我对密码学没有研究，关于各种 key 的描述可能有不准确。

## TLS handshake

TLS 主要有两种类型的 handshake：

| |!Key establishment |!Authentication |
|!RSA handshake |RSA |RSA |
|!DH handshake |DH |RSA/DSA |

### 性能

由快至慢：

1. DH 握手，使用非 RSA 证书
2. RSA 握手（只能使用 RSA 证书）
3. DH 握手，使用 RSA 证书

例如 CluodFlare 用的就是非 RSA 证书，公钥使用的是 ECDSA 算法：

![cloudflare-certificate]({{ site.image_cdn }}/images/2018/02/cloudflare-certificate.png)

而 GitHub 使用的就是 RSA 证书：

![github-certificate]({{ site.image_cdn }}/images/2018/02/github-certificate.png)

### Forward Secrecy

RSA 握手，会使用同一对公私钥来做 authentication 和加密对称密钥（key establishment）。这使得一旦有攻击者，在中间把两端通信的流量截取下来，同时又获得了服务器的私钥，则通信双方的信息会全部被破解。这种加密方式无法保证 forward secrecy。

而 DH（Diffie-Hellman）握手时，可以实现 authentication 用 server 的公私钥，但是 key establishment 用另外生成的一个 key。它能保证 forward secrecy。

### Cipher Suites

RSA 握手的 cipher suites，如 TLS_RSA_WITH_AES_128_GCM_SHA256，表示：

* RSA for key establishment
* RSA for authentication
* 128-bit Advanced Encryption Standard in Cipher Block Chaining (CBC) mode for confidentiality
* 256-bit Secure Hashing Algorithm (SHA) for integrity

DH 握手的有 ECDHE-ECDSA-AES256-GCM-SHA384，表示：

* Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) key exchange for key establishment
* Elliptic Curve Digital Signature Algorithms (ECDSA) for authentication
* 256-bit Advanced Encryption Standard in Galois/Counter mode (GCM) for confidentiality
* 384-bit Secure Hashing Algorithm for integrity

目前浏览器和主流的 server 端配置，会更倾向于使用 DH 握手。

用 WireShark 观察浏览器发出的 TLS 握手信息，可以它支持的 cipher suites 及其优先级。

## Chain of Trust

Client 发起 TLS handshake 时，怎样判断 server 不是中间人扮演的呢？比如你访问 google.com，怎样判断你是在跟真的 google.com 对话，而不是一个伪装的呢？答案在证书中。

证书由 CA（Certificate Authority，数字证书认证机构）颁发。而 CA 又分 intermediate CA 和 root CA，一般网站的证书由 intermediate CA 颁发，而 intermediate CA 的证书则由根证书颁发。root CA 的证书则是自已颁发给自己（self-sign）：

![chain-of-trust]({{ site.image_cdn }}/images/2018/02/chain-of-trust.png)

举个例子，google.com.hk 的证书路径：

![certificate-path]({{ site.image_cdn }}/images/2018/02/certificate-path.png)

其中 Google Internet Authority 是 intermediate CA，GlobalSign 则是 root CA。

「颁发」这个动作，用的是电子签名技术。比如 CA 颁发证书给你的网站，即是用 CA 的私钥加密一段信息放进证书中。客户端拿到证书后用 CA 的公钥进行解密和校验，则可以判断证书是否真的是 CA 所颁发。而 root CA 的公钥会被各操作系统、浏览器内置并且信任。如果你信任你的操作系统、浏览器中绑定的 root CA 信息，那么你也会信任其签名的 intermediate CA 证书和随后的网站证书。这就是信任链的运作模式。

为了提升性能，server 发送证书给 client 时，会把证书路径下的多个证书一起发给客户端做校验。

证书所使用的标准是 X.509 Version 3。

## ALPN extension

TLS 只是一层加密层，它承载的应用数据可以是各种各样，不仅限于 HTTP。ALPN (Application-Layer Protocol Negotiation) 是 client 用来与 server 协商通信层协议的。Client 在 ClientHello 发送它所支持的应用层协议（目前主要是 HTTP2 和 HTTP 1.1），server 从中选择一个，通过 ServerHello 给回。

下面是用 WireShark 抓包的一个例子。ClientHello 中，Chrome 表示它接受 HTTP2 和 HTTP 1.1：

![client-alpn]({{ site.image_cdn }}/images/2018/02/client-alpn.png)

ServerHello 中，server 表示它选择了 HTTP2：

![server-alpn]({{ site.image_cdn }}/images/2018/02/server-alpn.png)

## SNI

SNI (Server Name Indication) ，是一个 TLS Extension，在 ClientHello 中可以看到它的身影：

![sni]({{ site.image_cdn }}/images/2018/02/sni.png)

由于 TLS 握手发生在实际的 HTTP 请求之前，web server 还无法通过 HTTP 头中的 Host 字段来判断用户访问的是哪个网站，因此需要通过 TLS 协议带过去。对于一个 IP 部署多个网站的情况，不同的网站往往使用不同的证书，web server 需要通过 SNI 它才可以判断给哪个证书。

但是 SNI 和证书信息都是明文的，会导致中间人知道你访问的是哪个网站。但是它无法知道你访问网站里面的什么路径、内容。

## TLS Performance Tuning

TLS 性能调优这块，强烈推荐看看 High Performance Browser Networking 的 [Transport Layer Security (TLS)](https://hpbn.co/transport-layer-security-tls/#enable-tls-false-start)，写得深入浅出、形象生动。我这里是消化后的二手内容。

### TLS Session Resumption

除去 TCP 三次握手，一个完整的 TLS 握手需要耗 2 个 RTT。TLS 握手主要是 client 跟 server 互相交换加密相关信息，而前一次协商后的结果往往可以用到下一次连接中，因此 TLS 提供了一些机制来降低这里的延时。

第一种机制是，服务端提供一个 session ID 给客户端，并缓存相应的握手信息。客户端下一次连接握手时带上 session ID，如果服务端查到该 ID 的相关握手信息，则可以省去一个完整的 session negotiation，直接用上次的对称密钥做通信。这样节省掉 1 个 RTT：

![abbreviated-tls-handshake]({{ site.image_cdn }}/images/2018/02/abbreviated-tls-handshake.png)

Session ID 的问题是，现代大规模的网站往往前面有多机做负载均衡，比如多台 nginx 在前端做 TLS 相关的工作，再把请求转发到后端逻辑 server。这样导致 Session ID 的缓存需要能被多台 nginx 访问，带来了额外的运维成本。于是 TLS 给出第二种机制，session ticket，用来解决这个问题。

Session ticket 其实很简单，就是把服务端缓存挪到客户端。Session negotiation 的结果通过服务器下发，然后客户端缓存起来。下一次请求时客户端又带给服务器，如果服务器无异议，则可以直接开始数据传输。

### TLS False Start

TLS False Start 是 Google 提出来的一项优化，用来减少 TLS 握手的时延。原理是 client 发送 CipherChangeSpec 后立刻发送应用层数据：

![tls-false-start]({{ site.image_cdn }}/images/2018/02/tls-false-start.png)

TLS False Start 需要浏览器跟 server 都支持，比如：

* 协商的 cipher suite 需要支持 forward secrecy（因为不需要交换 pre-master key？）
* 双方需要支持 ALPN Extention 而不是 NPN（参考：[为什么我们应该尽快支持 ALPN？](https://imququ.com/post/enable-alpn-asap.html)）

Session Resumption 使得第二次 TLS 握手可以只用 1 个 RTT，但是第一次握手还是需要 2 个 RTT。而 TLS False Start 则使第一次 TLS 握手也只需要占用 1 个 TLS。

### Certificate Revocation

证书在一些情况下会被 CA 吊销，比如私钥泄漏，证书更换主体等。这个时候需要有一种机制，让客户端不再信任已被吊销的证书。

有几种机制：

* Certificate Revocation List (CRL)，即 CA 维护一个庞大的已被回收的证书列表，由客户端定时更新
* Online Certificate Status Protocol (OCSP)，即 CA 提供一个服务，让客户端可以实时查某个证书的吊销情况
* OCSP Stapling，下面详述

前两种存在的问题，可以自行考虑，[这里](https://hpbn.co/transport-layer-security-tls/#certificate-revocation-list-crl) 也给出了一些看法。

OCSP Stapling 是由 server 定时向 CA 发起 OCSP 请求，并附在 ServerHello 回给客户端，而不是由客户端发起。CA 给的 OCSP 回包会用它的私钥加密，这样 client 拿到这份 OCSP 回包时，可以校验出是 CA 给的。这样有几个好处 client 不直接发 OCSP 给 CA，不会暴露访问历史，不需要自己等待 CA 给回复。

在 WireShark 观察的相关数据如下：

![ocsp-stamping]({{ site.image_cdn }}/images/2018/02/ocsp-stamping.png)

## 前端相关

Jerry Qu 的 [博客](https://imququ.com) 上提到很多跟前端相关的 HTTPS 内容。前端内容多且繁杂，不同浏览器支持能力不一，在这里就不详述了，只简单讲讲 HSTS 头。

### HTTP to HTTPS redirect

这个重定向带来了额外的网络延时。可以用 `Strict-Transport-Security` (HSTS) 头，告诉浏览器应该只访问这个网站的 HTTPS 版本。浏览器会忽略在 HTTP 页面下收到的这个文件头，因为攻击者可以窜改这些数据。

HSTS 头同样可以解决一些安全问题。比如你在公共 Wi-Fi 中连接到一个攻击者的热点，你打开 HTTP 版本的网银页面时，攻击者会把页面 302 重定向到一个假的克隆的站点，以盗取密码。如果网银页面启用了 HSTS 头，而且你之前访问过它，那么你在公共 Wi-Fi 中连接时，也会默认走 HTTPS，避开攻击者的攻击。

Chrome 维护了一个 HSTS preload list，即在这个名单中的域名，Chrome 会默认使用 HTTPS。Firefox 和 IE 似乎也使用这个列表。注意这个 preload list 并不在 HSTS 规范中，只是个事实上（de-facto）的规范。

## 安全

这块我没有深入了解，参考下 [Mozilla Wiki](https://wiki.mozilla.org/Security/Server_Side_TLS#Attacks_on_SSL_and_TLS) 和 [RFC 7457 - Summarizing Known Attacks on Transport Layer Security (TLS) and Datagram TLS (DTLS)](https://tools.ietf.org/html/rfc7457)。

## 工具

[Qualys SSL Labs](https://www.ssllabs.com/) 能检测不同浏览器 / Server 对于 SSL 的支持能力、安全性等，信息非常详细

Mozilla 提供了非常好的工具，用来检查一个网站支持的 cipher suites、证书路径等：

* [mozilla/tls-observatory](https://github.com/mozilla/tls-observatory)
* [mozilla/cipherscan](https://github.com/mozilla/cipherscan)

Mozilla 同时提供了一个 [SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)，用来为不同的 web server 生成安全可靠的配置文件。

[Is TLS Fast Yet?](https://istlsfastyet.com/) 网站，提供了各服务器程序（如 nginx）和 CDN 服务（如 Akamai）对 TLS 各优化项的支持能力。

[Let's Encrypt](https://letsencrypt.org/) 提供了免费的证书申请渠道和 [工具](https://github.com/certbot/certbot)，工具可以帮你自动申请证书及修改 web server 配置，非常赞。

## 参考信息

* [HTTP: The Definitive Guide](http://shop.oreilly.com/product/9781565925090.do), Chapter 14 Secure HTTP
* [High Performance Browser Networking - Chapter 4 Transport Layer Security (TLS)](https://hpbn.co/transport-layer-security-tls/) (highly recommend)
* [Keyless SSL: The Nitty Gritty Technical Details](https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/)
* [TLS Handshake Protocol - MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/aa380513(v=vs.85).aspx)
* [RFC 5246 - TLS 1.2 规范](https://tools.ietf.org/html/rfc5246)
* [Security/Server Side TLS - Mozilla Wiki](https://wiki.mozilla.org/Security/Server_Side_TLS)
* [Mythbusting HTTPS: Squashing security's urban legends - Google I/O 2016](https://www.youtube.com/watch?v=YMfW1bfyGSY)
* [Jerry Qu 的博客](https://imququ.com/)
* [The First Few Milliseconds of an HTTPS Connection](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)

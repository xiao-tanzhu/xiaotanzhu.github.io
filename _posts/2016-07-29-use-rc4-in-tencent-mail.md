---
layout: post
title: RC4被JDK8默认禁用导致腾讯QQ邮箱无法访问
tags:
- JavaMail
- JDK
categories: Java
description: RC4被JDK8默认禁用导致腾讯QQ邮箱无法访问
---
7月29日开始，腾讯修改了邮箱的加密方式，导致我们线上的所有的腾讯代收、代发邮件的功能全部失效。解决方法在最后，如果需要可直接跳转至[解决方法](#解决方法)一节

### 问题出现

7月29日开始，线上的所有的腾讯代收、代发邮件的功能全部失效，报`handshake_error`:
```
javax.mail.MessagingException: Connect failed;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
	at com.sun.mail.pop3.POP3Store.protocolConnect(POP3Store.java:209)
	at javax.mail.Service.connect(Service.java:295)
	at javax.mail.Service.connect(Service.java:176)
	at cn.irenshi.biz.recruit.service.impl.RecruitReceiveMailServiceImpl.testConnect(RecruitReceiveMailServiceImpl.java:507)
	at cn.irenshi.biz.recruit.service.impl.RecruitReceiveMailServiceImpl.testMailConnect(RecruitReceiveMailServiceImpl.java:534)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:302)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	... more

Caused by: javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
	at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
	at sun.security.ssl.Alerts.getSSLException(Alerts.java:154)
	at sun.security.ssl.SSLSocketImpl.recvAlert(SSLSocketImpl.java:2023)
	at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1125)
	at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1375)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1403)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1387)
	at com.sun.mail.util.SocketFetcher.configureSSLSocket(SocketFetcher.java:549)
	at com.sun.mail.util.SocketFetcher.createSocket(SocketFetcher.java:354)
	at com.sun.mail.util.SocketFetcher.getSocket(SocketFetcher.java:237)
	at com.sun.mail.pop3.Protocol.<init>(Protocol.java:112)
	at com.sun.mail.pop3.POP3Store.getPort(POP3Store.java:260)
	at com.sun.mail.pop3.POP3Store.protocolConnect(POP3Store.java:205)
	... 132 more
```
记得当时用客户端链接腾讯的企业邮箱时，报证书警告，警告原因是`pop.exmail.qq.com`这个域名使用了`pop.qq.com`这个证书，应该是为了省钱吧。但是当用QQ个人邮箱连接的时候，确实使用了正确的证书，那错误原因应该不同。

### 问题定位
把问题交给做邮箱连接的同事，结果同事很快告诉我，他在windows上运行代码完全没有任何问题，并且JDK都用了相同的版本：1.8.0_u60。难道这个错误和系统相关？
打开Thunderbird尝试连接腾讯邮箱，发现一切也正常。可以断定这个问题不是系统相关的，问题一定出在我们的代码中。网上搜了一些`handshake_failure`相关的原因如下（[StackOverflow](http://stackoverflow.com/a/6353956 "StackOverflow")）：

> - Incompatible cipher suites in use by the client and the server. This would require the client to use (or enable) a cipher suite that is supported by the server.
> - Incompatible versions of SSL in use (the server might accept only TLS v1, while the client is capable of only using SSL v3). Again, the client might have to ensure that it uses a compatible version of the SSL/TLS protocol.
> - Incomplete trust path for the server certificate; the server's certificate is probably not trusted by the client. This would usually result in a more verbose error, but it is quite possible. Usually the fix is to import the server's CA certificate into the client's trust store.
> - The cerificate is issued for a different domain. Again, this would have resulted in a more verbose message, but I'll state the fix here in case this is the cause. The resolution in this case would be get the server (it does not appear to be yours) to use the correct certificate.

问题来了，如果以上是问题所在的话，一定会有一些错误信息输出，但为什么在我的系统中没有任何输出？

想到telnet，因为一般测试邮箱连接都直接使用telnet明文测试一下。当然这个想法是不可行的，如下：
```bash
fify@fify-PC:~$ telnet pop.qq.com 995
Trying 163.177.72.198...
Connected to pop.qq.com.
Escape character is '^]'.
```
没有办法输入任何东西，原因也很简单，995端口使用了加密。（这里犯迷糊了，现在握手错误就是因为加密问题...）

换一个方式连接POP邮箱：
```bash
openssl s_client -connect pop.qq.com:995
```
输出如下：
```
CONNECTED(00000003)
depth=2 C = US, O = GeoTrust Inc., CN = GeoTrust Global CA
verify return:1
depth=1 C = US, O = GeoTrust Inc., CN = GeoTrust SSL CA - G3
verify return:1
depth=0 C = CN, ST = Guangdong, L = Shenzhen, O = Shenzhen Tencent Computer Systems Company Limited, OU = R&D, CN = pop.qq.com
verify return:1
---
Certificate chain
 0 s:/C=CN/ST=Guangdong/L=Shenzhen/O=Shenzhen Tencent Computer Systems Company Limited/OU=R&D/CN=pop.qq.com
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust SSL CA - G3
 1 s:/C=US/O=GeoTrust Inc./CN=GeoTrust SSL CA - G3
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIGbzCCBVegAwIBAgIQZlTnxqFc/rVo50RzuVnejDANBgkqhkiG9w0BAQsFADBE
MQswCQYDVQQGEwJVUzEWMBQGA1UEChMNR2VvVHJ1c3QgSW5jLjEdMBsGA1UEAxMU
R2VvVHJ1c3QgU1NMIENBIC0gRzMwHhcNMTYwMTI3MDAwMDAwWhcNMTYxMDIzMjM1
OTU5WjCBkzELMAkGA1UEBhMCQ04xEjAQBgNVBAgTCUd1YW5nZG9uZzERMA8GA1UE
BxQIU2hlbnpoZW4xOjA4BgNVBAoUMVNoZW56aGVuIFRlbmNlbnQgQ29tcHV0ZXIg
U3lzdGVtcyBDb21wYW55IExpbWl0ZWQxDDAKBgNVBAsUA1ImRDETMBEGA1UEAxQK
cG9wLnFxLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALeSY7Vb
60Cvv7P2O+zhaZnqlz/KFs//DH4It3xmyMPFOPUFopzN1h8n3/4FPqGBtqEEuWBE
/o7soZT30E8bw30Tl07VOcYm/fPKi1pyro3hNEdLi5Wlta9fKxDAvw0U3clSq39R
qihYIDAA3QrDuqI54gULa5IZnqM16A9VBULPfIDaXbdgaAIJ5Ak92nC13YcdQYuv
egL6jOWSKzCRTqeRAg+6dWkfce1+gAOCuCUDgAso2EJ+k9nFe/LAMMGdGbe4KI9H
CwpDCMo+2k2u4SQtXOmuYke7nNmRnpJeL3qZnGWsqT7l3N0mYCc/+3zcMfAcmyuo
H90stoWF/G2T2rcCAwEAAaOCAwswggMHMIIBggYDVR0RBIIBeTCCAXWCCm14Mi5x
cS5jb22CEmltYXAuZXhtYWlsLnFxLmNvbYISdXBsb2FkLm1haWwucXEuY29tgg90
ZWwubWFpbC5xcS5jb22CFGh3c210cC5leG1haWwucXEuY29tgg9tb2IubWFpbC5x
cS5jb22CEXJ0eC5leG1haWwucXEuY29tgg1teGJpejIucXEuY29tgg1teGJpejEu
cXEuY29tgg5oay5tYWlsLnFxLmNvbYIOY2xvdWRteC5xcS5jb22CFGh3aW1hcC5l
eG1haWwucXEuY29tggpteDEucXEuY29tghJzbXRwLmV4bWFpbC5xcS5jb22CEXBv
cC5leG1haWwucXEuY29tghNod3BvcC5leG1haWwucXEuY29tggpteDMucXEuY29t
ggtzbXRwLnFxLmNvbYIKZGF2LnFxLmNvbYIJZXgucXEuY29tgg9jbmMubWFpbC5x
cS5jb22CC2ltYXAucXEuY29tggpwb3AucXEuY29tMAkGA1UdEwQCMAAwDgYDVR0P
AQH/BAQDAgWgMCsGA1UdHwQkMCIwIKAeoByGGmh0dHA6Ly9nbi5zeW1jYi5jb20v
Z24uY3JsMIGdBgNVHSAEgZUwgZIwgY8GBmeBDAECAjCBhDA/BggrBgEFBQcCARYz
aHR0cHM6Ly93d3cuZ2VvdHJ1c3QuY29tL3Jlc291cmNlcy9yZXBvc2l0b3J5L2xl
Z2FsMEEGCCsGAQUFBwICMDUMM2h0dHBzOi8vd3d3Lmdlb3RydXN0LmNvbS9yZXNv
dXJjZXMvcmVwb3NpdG9yeS9sZWdhbDAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYB
BQUHAwIwHwYDVR0jBBgwFoAU0m/3lvSFP3I8MH0j2oV4m6N8WnwwVwYIKwYBBQUH
AQEESzBJMB8GCCsGAQUFBzABhhNodHRwOi8vZ24uc3ltY2QuY29tMCYGCCsGAQUF
BzAChhpodHRwOi8vZ24uc3ltY2IuY29tL2duLmNydDANBgkqhkiG9w0BAQsFAAOC
AQEAvta4aGvK5qe31ZnLbmtblhgLD11dAdSom3sEnkF8UHtoi+gPiHBmHy1t39Du
2w+5aeriqwsetdDNuAhh6ckKJhGjc9ochWw2lvyuHPko8sSDdBd/oUYBh60lREwB
DoAi7x37QIjia4yprFCNs/+bV+bee+2nijeNYibgwLQ+5jZL89jC6BVXxLSTenVw
B2bzQPauNo+DOsB6ubY/i5r9p2E1DHAO9AluN/epJZ1gwZhYlOey71s59341w/ql
ZJImDrWch+Gj1ZgnXWnttgOSafqynPA6VtiFyYGF4zLboxIkNiyuwj+ZzuugV97z
IurYVE9FA7vTlfeJhAkG2gIwsA==
-----END CERTIFICATE-----
subject=/C=CN/ST=Guangdong/L=Shenzhen/O=Shenzhen Tencent Computer Systems Company Limited/OU=R&D/CN=pop.qq.com
issuer=/C=US/O=GeoTrust Inc./CN=GeoTrust SSL CA - G3
---
No client certificate CA names sent
---
SSL handshake has read 3070 bytes and written 619 bytes
---
New, TLSv1/SSLv3, Cipher is RC4-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : RC4-SHA
    Session-ID: E278833690D2364F44B8E2B6D3F3708888411AD55298F02A0710C73FE229BBE9
    Session-ID-ctx: 
    Master-Key: AF8A9394C87F52872A31DCC5ED62FF5F97B6F621CC9337E151D5F6229E9F231626F10B392A1938669EE72911ABD860D6
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 6c 30 25 d9 92 71 25 7c-3e bd 7b e6 e2 a5 13 d1   l0%..q%|>.{.....
    0010 - 9b f9 61 e6 3d dc e6 ea-96 9d 04 02 ea 6f 68 0f   ..a.=........oh.
    0020 - 18 a3 a3 e6 39 02 b9 d2-dd d1 2c 18 6d 9c 87 e5   ....9.....,.m...
    0030 - 31 a9 53 a0 6c 2d 4c b6-d4 d6 35 ef d9 04 b0 b9   1.S.l-L...5.....
    0040 - 70 af 82 74 1e 1d 26 9a-00 00 6b 90 2e eb 56 e9   p..t..&...k...V.
    0050 - d8 f4 cd 56 d5 c2 02 80-0e d9 15 e5 2a b9 1f f3   ...V........*...
    0060 - 8a 90 7b c0 72 6e c5 2a-04 2c 91 1c 11 fd 40 ba   ..{.rn.*.,....@.
    0070 - 38 fb db fb eb b7 65 e1-e1 51 1a e3 b2 f3 64 4e   8.....e..Q....dN
    0080 - 54 6b 5f 0e 9d be 40 60-dd 68 8f 52 5d f3 48 36   Tk_...@`.h.R].H6
    0090 - 40 7b 11 68 10 7f 7d e2-d6 93 19 48 42 f0 da bc   @{.h..}....HB...

    Start Time: 1470455760
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
+OK QQMail POP3 Server v1.0 Service Ready(QQMail v2.0)
```
注意到以下片段

>SSL-Session:
>    Protocol  : TLSv1.2
>    Cipher    : RC4-SHA

可以看到，加密使用的协议是`TLSv1.2`，Cipher使用的是`RC4-SHA`。

那么问题是出在这两个地方吗？

### JavaMail握手时的Protocol和Cipher

在Java启动中增加参数`-Djavax.net.debug=all`可以开启加密协议的调试模式。详情可以参考[Oracle提供的文档](http://docs.oracle.com/javase/6/docs/technotes/guides/security/jsse/ReadDebug.html)。

开启之后，连接邮箱握手的过程中打出了以下日志：
```
trigger seeding of SecureRandom
done seeding SecureRandom
Ignoring unavailable cipher suite: TLS_DHE_DSS_WITH_AES_256_GCM_SHA384
Ignoring unavailable cipher suite: TLS_RSA_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
Ignoring unavailable cipher suite: TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
Ignoring unavailable cipher suite: TLS_RSA_WITH_AES_256_CBC_SHA256
Ignoring unavailable cipher suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384
Ignoring unavailable cipher suite: TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384
Ignoring unavailable cipher suite: TLS_RSA_WITH_AES_256_GCM_SHA384
Ignoring unavailable cipher suite: TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384
Ignoring unavailable cipher suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
Ignoring unavailable cipher suite: TLS_ECDH_RSA_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384
Ignoring unavailable cipher suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
Ignoring unavailable cipher suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA256
Ignoring unavailable cipher suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_DHE_DSS_WITH_AES_256_CBC_SHA256
Ignoring unavailable cipher suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
Ignoring unavailable cipher suite: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
Allow unsafe renegotiation: false
Allow legacy hello messages: true
Is initial handshake: true
Is secure renegotiation: false
Ignoring unsupported cipher suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_RSA_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA256 for TLSv1
Ignoring unsupported cipher suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 for TLSv1.1
Ignoring unsupported cipher suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 for TLSv1.1
Ignoring unsupported cipher suite: TLS_RSA_WITH_AES_128_CBC_SHA256 for TLSv1.1
Ignoring unsupported cipher suite: TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256 for TLSv1.1
Ignoring unsupported cipher suite: TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256 for TLSv1.1
Ignoring unsupported cipher suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA256 for TLSv1.1
Ignoring unsupported cipher suite: TLS_DHE_DSS_WITH_AES_128_CBC_SHA256 for TLSv1.1
%% No cached client session
*** ClientHello, TLSv1.2
RandomCookie:  GMT: 1453331365 bytes = { 172, 246, 197, 31, 208, 63, 31, 30, 107, 70, 211, 242, 90, 243, 100, 108, 44, 192, 70, 4, 238, 84, 176, 59, 5, 75, 162, 127 }
Session ID:  {}
Cipher Suites: [TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_DSS_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_DSS_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]
Compression Methods:  { 0 }
Extension elliptic_curves, curve names: {secp256r1, sect163k1, sect163r2, secp192r1, secp224r1, sect233k1, sect233r1, sect283k1, sect283r1, secp384r1, sect409k1, sect409r1, secp521r1, sect571k1, sect571r1, secp160k1, secp160r1, secp160r2, sect163r1, secp192k1, sect193r1, sect193r2, secp224k1, sect239k1, secp256k1}
Extension ec_point_formats, formats: [uncompressed]
Extension signature_algorithms, signature_algorithms: SHA512withECDSA, SHA512withRSA, SHA384withECDSA, SHA384withRSA, SHA256withECDSA, SHA256withRSA, SHA224withECDSA, SHA224withRSA, SHA1withECDSA, SHA1withRSA, SHA1withDSA, MD5withRSA
Extension server_name, server_name: [type=host_name (0), value=pop.qq.com]
***
[write] MD5 and SHA1 hashes:  len = 214
0000: 01 00 00 D2 03 03 57 A0   14 A5 AC F6 C5 1F D0 3F  ......W........?
0010: 1F 1E 6B 46 D3 F2 5A F3   64 6C 2C C0 46 04 EE 54  ..kF..Z.dl,.F..T
0020: B0 3B 05 4B A2 7F 00 00   3A C0 23 C0 27 00 3C C0  .;.K....:.#.'.<.
0030: 25 C0 29 00 67 00 40 C0   09 C0 13 00 2F C0 04 C0  %.).g.@...../...
0040: 0E 00 33 00 32 C0 2B C0   2F 00 9C C0 2D C0 31 00  ..3.2.+./...-.1.
0050: 9E 00 A2 C0 08 C0 12 00   0A C0 03 C0 0D 00 16 00  ................
0060: 13 00 FF 01 00 00 6F 00   0A 00 34 00 32 00 17 00  ......o...4.2...
0070: 01 00 03 00 13 00 15 00   06 00 07 00 09 00 0A 00  ................
0080: 18 00 0B 00 0C 00 19 00   0D 00 0E 00 0F 00 10 00  ................
0090: 11 00 02 00 12 00 04 00   05 00 14 00 08 00 16 00  ................
00A0: 0B 00 02 01 00 00 0D 00   1A 00 18 06 03 06 01 05  ................
00B0: 03 05 01 04 03 04 01 03   03 03 01 02 03 02 01 02  ................
00C0: 02 01 01 00 00 00 0F 00   0D 00 00 0A 70 6F 70 2E  ............pop.
00D0: 71 71 2E 63 6F 6D                                  qq.com
http-nio-8080-exec-2, WRITE: TLSv1.2 Handshake, length = 214
[Raw write]: length = 219
0000: 16 03 03 00 D6 01 00 00   D2 03 03 57 A0 14 A5 AC  ...........W....
0010: F6 C5 1F D0 3F 1F 1E 6B   46 D3 F2 5A F3 64 6C 2C  ....?..kF..Z.dl,
0020: C0 46 04 EE 54 B0 3B 05   4B A2 7F 00 00 3A C0 23  .F..T.;.K....:.#
0030: C0 27 00 3C C0 25 C0 29   00 67 00 40 C0 09 C0 13  .'.<.%.).g.@....
0040: 00 2F C0 04 C0 0E 00 33   00 32 C0 2B C0 2F 00 9C  ./.....3.2.+./..
0050: C0 2D C0 31 00 9E 00 A2   C0 08 C0 12 00 0A C0 03  .-.1............
0060: C0 0D 00 16 00 13 00 FF   01 00 00 6F 00 0A 00 34  ...........o...4
0070: 00 32 00 17 00 01 00 03   00 13 00 15 00 06 00 07  .2..............
0080: 00 09 00 0A 00 18 00 0B   00 0C 00 19 00 0D 00 0E  ................
0090: 00 0F 00 10 00 11 00 02   00 12 00 04 00 05 00 14  ................
00A0: 00 08 00 16 00 0B 00 02   01 00 00 0D 00 1A 00 18  ................
00B0: 06 03 06 01 05 03 05 01   04 03 04 01 03 03 03 01  ................
00C0: 02 03 02 01 02 02 01 01   00 00 00 0F 00 0D 00 00  ................
00D0: 0A 70 6F 70 2E 71 71 2E   63 6F 6D                 .pop.qq.com
javax.mail.MessagingException: Connect failed;
  nested exception is:
	javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
	at com.sun.mail.pop3.POP3Store.protocolConnect(POP3Store.java:209)
	at javax.mail.Service.connect(Service.java:295)
	at javax.mail.Service.connect(Service.java:176)
	at cn.irenshi.biz.recruit.service.impl.RecruitReceiveMailServiceImpl.testConnect(RecruitReceiveMailServiceImpl.java:507)
	at cn.irenshi.biz.recruit.service.impl.RecruitReceiveMailServiceImpl.testMailConnect(RecruitReceiveMailServiceImpl.java:534)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:302)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:281)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:207)
	at com.sun.proxy.$Proxy1083.testMailConnect(Unknown Source)
	at cn.irenshi.web.controller.recruit.RecruitReceiveMailController.testMailConnect(RecruitReceiveMailController.java:116)
	at cn.irenshi.web.controller.recruit.RecruitReceiveMailController$$FastClassBySpringCGLIB$$4d626811.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:717)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.security.access.intercept.aopalliance.MethodSecurityInterceptor.invoke(MethodSecurityInterceptor.java:68)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:653)
    at ...
Caused by: javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
	at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)
	at sun.security.ssl.Alerts.getSSLException(Alerts.java:154)
	at sun.security.ssl.SSLSocketImpl.recvAlert(SSLSocketImpl.java:2023)
	at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:1125)
	at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1375)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1403)
	at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1387)
	at com.sun.mail.util.SocketFetcher.configureSSLSocket(SocketFetcher.java:549)
	at com.sun.mail.util.SocketFetcher.createSocket(SocketFetcher.java:354)
	at com.sun.mail.util.SocketFetcher.getSocket(SocketFetcher.java:237)
	at com.sun.mail.pop3.Protocol.<init>(Protocol.java:112)
	at com.sun.mail.pop3.POP3Store.getPort(POP3Store.java:260)
	at com.sun.mail.pop3.POP3Store.protocolConnect(POP3Store.java:205)
	... 132 more
[Raw read]: length = 5
0000: 15 03 03 00 02                                     .....
[Raw read]: length = 2
0000: 02 28                                              .(
http-nio-8080-exec-2, READ: TLSv1.2 Alert, length = 2
http-nio-8080-exec-2, RECV TLSv1.2 ALERT:  fatal, handshake_failure
http-nio-8080-exec-2, called closeSocket()
http-nio-8080-exec-2, handling exception: javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
```
注意到：

>Cipher Suites: [TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_RSA_WITH_AES_128_CBC_SHA256, TLS_DHE_DSS_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_RSA_WITH_AES_128_GCM_SHA256, TLS_DHE_DSS_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]

发现其中果然没有`RC4`相关的Cipher Suites。

在同事Windows机器上试了以下，输入果然不同，如下：

> Cipher Suites: [TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDHE_ECDSA_WITH_RC4_128_SHA, TLS_ECDHE_RSA_WITH_RC4_128_SHA, SSL_RSA_WITH_RC4_128_SHA, TLS_ECDH_ECDSA_WITH_RC4_128_SHA, TLS_ECDH_RSA_WITH_RC4_128_SHA, TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_RC4_128_MD5, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]

赫然发现：SSL_RSA_WITH_RC4_128_SHA。这正是我们想要的东西。

### 解决问题

问题找到了，解决办法应该也比较容易找到。查看POP协议相关的[所有参数](https://javamail.java.net/nonav/docs/api/com/sun/mail/pop3/package-summary.html)，找到了`mail.pop3.ssl.ciphersuites`。在JavaMail的Properties中增加相关的`SSL_RSA_WITH_RC4_128_SHA`：
```java
prop.setProperties("mail.pop3s.ssl.ciphersuites", "SSL_RSA_WITH_RC4_128_SHA");
```
重启服务器，测试，一气呵成，结果：`handshake_failure`

### 还是系统问题？

以`SSL_RSA_WITH_RC4_128_SHA`为关键字搜索，搜到了这篇文章：[Java 8 update 60 disables "RC4" cipher suites: Causes issues with Blancco erasure software and MC 3 communication](https://support.blancco.com/index.php?/Knowledgebase/Article/View/497/117/java-8-update-60-disables-rc4-cipher-suites-causes-issues-with-blancco-erasure-software-and-mc-3-communication)。这是别的软件遇到的问题，但原因是一样的。

原来从JDK 1.8.0_u60开始，默认禁止了RC4这个算法。可以在`{JRE_HOME}/lib/security/java.security`找到相关配置：

>534 jdk.tls.disabledAlgorithms=SSLv3, RC4, MD5withRSA, DH keySize < 768

以及

>586 jdk.tls.legacyAlgorithms= \
>587         K_NULL, C_NULL, M_NULL, \
>588         DHE_DSS_EXPORT, DHE_RSA_EXPORT, DH_anon_EXPORT, DH_DSS_EXPORT, \
>589         DH_RSA_EXPORT, RSA_EXPORT, \
>590         DH_anon, ECDH_anon, \
>591         RC4_128, RC4_40, DES_CBC, DES40_CBC

那Windows中同样的JDK版本，为什么却可以正常使用呢？比较发现在Windows安装之后的Java目录中，有两个部分，一个是JDK目录（其中包含一个JRE目录），一个是直接的JRE目录。打开两个`java.security`文件，惊奇的发现两个文件并不相同，JRE目录中的`java.security`并不包含禁用RC4算法这些配置。

### 解决方法

#### 启用Java的RC4算法

没有去深研究为什么JDK会默认禁止这个算法，也不知道腾讯为什么会重新选择了一个JDK默认禁用的算法（7月29日之前是没有问题的）。但由于需要用到QQ邮箱，所以必须得开启这个算法。开启步骤如下：

1. Go to the Java JRE installation folder: {JRE_HOME}\lib\security\
1. Locate java.security file.
1. Make a backup copy of the file.
1. Edit the java.security file with a text editor software (for example Notepad) according to the example further below.

```diff
534c534
- jdk.tls.disabledAlgorithms=SSLv3, RC4, MD5withRSA, DH keySize < 768
+ jdk.tls.disabledAlgorithms=SSLv3, MD5withRSA, DH keySize < 768
591c591
-         RC4_128, RC4_40, DES_CBC, DES40_CBC
+         DES_CBC, DES40_CBC
```

#### 启用协议

在POP/SMTP的协议中增加最新的TLSv1.2协议：
```java
prop.setProperties("mail.pop3s.ssl.protocols", "TSLv1 TSLv1.1 TLSv1.2");
```
或
```java
prop.setProperties("mail.smtps.ssl.protocols", "TSLv1 TSLv1.1 TLSv1.2");
```

#### 启用Cipher Scites

因为我需要兼容的不只是RC4这个算法，所以我启用了大部分的Cipher Suites
```java
prop.setProperties("mail.pop3s.ssl.ciphersuites", "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA TLS_RSA_WITH_AES_128_CBC_SHA TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA TLS_ECDH_RSA_WITH_AES_128_CBC_SHA TLS_DHE_RSA_WITH_AES_128_CBC_SHA TLS_DHE_DSS_WITH_AES_128_CBC_SHA TLS_ECDHE_ECDSA_WITH_RC4_128_SHA TLS_ECDHE_RSA_WITH_RC4_128_SHA SSL_RSA_WITH_RC4_128_SHA TLS_ECDH_ECDSA_WITH_RC4_128_SHA TLS_ECDH_RSA_WITH_RC4_128_SHA TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA SSL_RSA_WITH_3DES_EDE_CBC_SHA TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA SSL_RSA_WITH_RC4_128_MD5 TLS_EMPTY_RENEGOTIATION_INFO_SCSV");
```
或
```java
prop.setProperties("mail.smtps.ssl.ciphersuites", "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA TLS_RSA_WITH_AES_128_CBC_SHA TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA TLS_ECDH_RSA_WITH_AES_128_CBC_SHA TLS_DHE_RSA_WITH_AES_128_CBC_SHA TLS_DHE_DSS_WITH_AES_128_CBC_SHA TLS_ECDHE_ECDSA_WITH_RC4_128_SHA TLS_ECDHE_RSA_WITH_RC4_128_SHA SSL_RSA_WITH_RC4_128_SHA TLS_ECDH_ECDSA_WITH_RC4_128_SHA TLS_ECDH_RSA_WITH_RC4_128_SHA TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA SSL_RSA_WITH_3DES_EDE_CBC_SHA TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA SSL_RSA_WITH_RC4_128_MD5 TLS_EMPTY_RENEGOTIATION_INFO_SCSV");
```
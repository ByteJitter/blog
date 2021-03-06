---
title: TLS连接过程
date: 2020-05-18
next: false
---

Https连接过程需要在正式收发数据前建立TLS连接，确保安全性。

TLS建立在TCP之上，建立TLS连接前需要TCP4次握手。然后进行TLS连接。在连接中要完成秘钥交换，交换算法不同，握手过程细节也会不同。
## ECDHE握手过程
![ECDHE握手过程（图片来自透视Http协议课程）](~@blogImg/tls-connection.png)
### 浏览器发送Client Hello消息
- 客户端向服务器打招呼，消息中包含客户端生成的随机数C，客户单的TLS版本号，可使用的密码套件列表及扩展列表。
- 后续需要对比TLS版本号，用随机数计算秘钥。
### 服务器发送Server Hello消息
- 服务器向客户端打招呼，消息包含服务器生成的随机数S，确认TLS版本号，从客户端可用密码套件列表中选用的密码套件。
- 还需包含数字证书，用于验证。
- 以及秘钥交换算法的参数（也就是公钥），需包含签名认证。
- 并确认已收到了Client Hello信息。
### 客户端验证并计算主密钥
- 对收到的证书和签名进行验证。
- 验证成功后向服务器发送送秘钥交换算法的参数。
- 服务器接收到客户端的秘钥交换算法参数（也就是私钥），开始计算master secret。先生成pre-master，根据pre-master和两个随机数算出master secret（主秘钥）。
- 客户端也进行主秘钥的计算。之所以称为主密钥，是因为根据主密钥可生成多个会话秘钥由于不同的具体加密，如：客户端发送用的会话秘钥、服务端用的会话秘钥。
- 目前都是明文通信，所以秘钥的生成必须很讲究，各部计算都需随机性极强的算法，保证安全性。
### 验证加密通信并验证
至此加密通信所用的秘钥都已生成，在此之前都是明文发送。客户端通知服务器启用加密通信并加密发送之前所发信息的摘要，服务器也做一样的事，用于验证加密通信是否可行。发送的两个消息分别是“Change Cipher Spec”和“Finished”。之后就可使用加密的Http请求和响应。
## 双向认证
上述方式验证了服务器的安全性，而服务器验证客户端可用更多方式，比如账号密码，此时已经达到了加密通信的标准。

更深层次的客户端验证，也可Client Hello中出示客户端证书，用以验证。最终实现更为安全的双向验证。比如银行的U盾。

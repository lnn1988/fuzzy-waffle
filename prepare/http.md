# 理解 HTTP 协议

## 输入 URL 到显示页面的过程
1. DNS 解析：根据域名查 IP
    1. 递归查询
2. TCP 链接（3 次握手）
3. HTTP 请求
4. 服务器处理请求：接收到TCP报文开始，它会对TCP连接进行处理，对HTTP协议进行解析，并按照报文格式进一步封装成HTTP Request对象，供上层使用。 
5. 代理服务器（nginx）
6. 接受 HTTP response
7. 浏览器解析渲染页面

## HTTP 协议特点
**客户端、服务端模式**
**简单快速**
只传送 Method 和 URL
**灵活**
可以传输多种类型的数据，在 Content-type 加以标记


**TCP**
HTTP协议(超文本传输协议HyperText Transfer Protocol)，它是基于TCP协议的应用层传输协议，简单来说就是客户端和服务端进行数据传输的一种规则。

HTTP协议属于应用层，建立在传输层协议TCP之上。客户端通过与服务器建立TCP连接，之后发送HTTP请求与接收HTTP响应都是通过访问Socket接口来调用TCP协议实现。

**无状态**
HTTP 是一种无状态 (stateless) 协议, HTTP协议本身不会对发送过的请求和相应的通信状态进行持久化处理。这样做的目的是为了保持HTTP协议的简单性，从而能够快速处理大量的事务, 提高效率。

然而，在许多应用场景中，我们需要保持用户登录的状态或记录用户购物车中的商品。由于HTTP是无状态协议，所以必须引入一些技术来记录管理状态，例如Cookie。

## HTTP 请求

* HTTP请求状态行
    * Method
    * URL
    * HTTP VERSION
* HTTP请求头
    * COOKIE
    * Accept
    * Content-Length	
    * Content-Type
* HTTP请求正文(GET 方法没有)

## HTTP 响应
* 响应状态行
    * HTTP version
    * 状态码
    * 状态描述
* 响应头
* 响应报文


## 持久连接
![H81HHF](https://gitee.com/lnn1988/upic1988/raw/master/uPic/H81HHF.jpg)
非持久连接在每次请求|响应之后都要断开连接，下次再建立新的TCP连接，这样就造成了大量的通信开销。例如前面提到的往返时间(RTT) 就是在建立TCP连接的过程中的代价。

## GET、POST、PUT
|          | 幂等  | 非幂等  |
|----------|-----|------|
| 改变服务器状态  | PUT | POST |
| 不改变服务器状态 | GET |      |

* GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
* GET参数通过URL传递，POST放在Request body中。
* 对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；
* 而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

## HTTP和HTTPS
**HTTP的不足**
* 通信使用明文(不加密),内容可能会被窃听
* 不验证通信方的身份,因此有可能遭遇伪装
* 无法证明报文的完整性,所以有可能已遭篡改

HTTP 协议中没有加密机制,但可以通 过和 SSL(Secure Socket Layer, 安全套接层 )或 TLS(Transport Layer Security, 安全层传输协议)的组合使用,加密 HTTP 的通信内容。属于通信加密，即在整个通信线路中加密。

![BpW3M0](https://gitee.com/lnn1988/upic1988/raw/master/uPic/BpW3M0.jpg)

## HTTPS 证书校验过程
**非对称加密**
* 首先接收方生成一对密钥，即私钥和公钥；
* 然后，接收方将公钥发送给发送方；
* 发送方生成会话秘钥，用收到的公钥对会话秘钥加密，再发送给接收方；
* 接收方收到数据后，使用自己的私钥解密得到会话秘钥，双方使用会话秘钥进行数据交换。
![uPOwO0](https://gitee.com/lnn1988/upic1988/raw/master/uPic/uPOwO0.jpg)

## HTTP 的 URL
eg:
`http://www.aspxfans.com:8080/news/index.asp?boardID=5&ID=24618&page=1#name`

1. 协议（http/https）
2. 域名（或者 IP）
3. 端口（http80，https443）
4. 虚拟目录
5. 文件名
6. 参数


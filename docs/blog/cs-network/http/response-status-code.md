---
title: 响应状态码详解
date: 2020-05-01
tags:
    - http
    - 状态码
---

状态码是响应报文状态行最重要的字段，表示服务器对请求处理的结果。

由3位10进制数组成，标准包含100-600间的数值。
## 1××
表示提示信息，当前是协议处理的中间状态，还需后续操作。
## 2××
表示服务器收到并成功处理用户的请求，最想看到的结果。
### 200
200 OK表示一切正常，非Head请求的响应一般都会有主体数据。
### 204
204 No Content表示一切正常，但是响应不含主体数据。
### 206
206 Partical Content用于HTTP分块下载、断点续传，常用于客户端发送“范围请求”、获取资源的部分数据时返回，所以其数据主体只是资源的一部分。常与字段Content-Range一块使用，表示当前返回主体数据是资源的哪一部分。
## 3××
### 301
301 Moved Permanently 表示永久重定向，该请求资源不存在了，将用新的URI访问。
### 302
302 Found表示暂时重定向，该请求资源还在，但是暂时需要用其他URI访问。

301和302都表示重定向，区别在于一个暂时的一个是永久的，响应报文中Location字段表示后续应该要请求的URI。比如说，网站或资源的彻底迁移用301，而网站的的临时维护应该用302，不做缓存优化，维护结束后继续访问之前URI。
### 304
304 Not Modefied表示请求的的资源没有修改，常和If-Modefied-Since等条件请求配合使用，用于缓存控制。可以理解为对缓存的重定向。
## 4××
4××表明客户端出错，可理解为客户端的错误码。
### 400
400 Bad Request表示当前的请求报文有误，但只是一个笼统的报错，不会指出是哪部分的错误。
### 403
403 Forbidden表示请求资源被服务器禁止访问，可能原因有信息敏感、法律条文等。
### 404
404 Not Found表示请求资源在服务器上找不到，过于常见。
### 其他常见4××
- 405 Method Not Allowed：不允许使用某些方法操作资源，例如不允许 POST 只能 GET；
- 406 Not Acceptable：资源无法满足客户端请求的条件，例如请求中文但只有英文；
- 408 Request Timeout：请求超时，服务器等待了过长的时间；
- 409 Conflict：多个请求发生了冲突，可以理解为多线程并发时的竞态；
- 413 Request Entity Too Large：请求报文里的 body 太大；
- 414 Request-URI Too Long：请求行里的 URI 太大；
- 429 Too Many Requests：客户端发送了太多的请求，通常是由于服务器的限连策略；
- 431 Request Header Fields Too Large：请求头某个字段或总体太大。
## 5××
5××表示服务器内部处理时出错，可理解为是服务端的错误码。
### 500
500 Internal Server Error，通用的错误码，和400一样不利于调试。
### 501
501 Not Implemented表示请求的功能还没支持开放。
### 502
502 Bad Way是服务器作为网关、代理时出错所返回的，表示服务器本身运行正常，但访问后端服务器出错，不显示具体错误原因。
### 503
503 Service Unavailabel表示当前服务器很忙，暂时无法响应服务。但忙碌状态不是一直存在，所以可设置Retry-After字段表明客户端可在多长时间后再次请求。





一、常用Http状态码

![[截屏2024-12-08 18.09.03.png]]

常见的HTTP状态码

1XX
- 100 Continue：表示正常，客户端可以继续发送请求
- 101 Switching Protocols：切换协议，服务器根据客户端的请求切换协议。

2XX
- 200 OK：请求成功
- 201 Created：已创建，表示成功请求并创建了新的资源
- 202 Accepted：已接受，已接受请求，但未处理完成。
- 204 No Content：无内容，服务器成功处理，但未返回内容。
- 205 Reset Content：重置内容，服务器处理成功，客户端应重置文档视图。
- 206 Partial Content：表示客户端进行了范围请求，响应报文应包含Content-Range指定范围的实体内容

3XX
- 301 Moved Permanently：永久性重定向
- 302 Found：临时重定向
- 303 See Other：和301功能类似，但要求客户端采用get方法获取资源
- 304 Not Modified：所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。
- 305 Use Proxy：所请求的资源必须通过代理访问
- 307 Temporary Redirect： 临时重定向，与302类似，要求使用get请求重定向。

4XX
- 400 Bad Request：客户端请求的语法错误，服务器无法理解。
- 401 Unauthorized：表示发送的请求需要有认证信息。
- 403 Forbidden：服务器理解用户的请求，但是拒绝执行该请求
- 404 Not Found：服务器无法根据客户端的请求找到资源。
- 405 Method Not Allowed：客户端请求中的方法被禁止
- 406 Not Acceptable：服务器无法根据客户端请求的内容特性完成请求
- 408 Request Time-out：服务器等待客户端发送的请求时间过长，超时

5XX
- 500 Internal Server Error：服务器内部错误，无法完成请求
- 501 Not Implemented：服务器不支持请求的功能，无法完成请求

二、Https流程

![[Pasted image 20241208180618.png]]

	主要流程：
	1. 客户端发起Https请求，连接到服务器的443端口。
	2. 服务器必须要有一套数字证书（证书内容有公钥、证书颁发机构、失效日期等）。
	3. 服务器将自己的数字证书发送给客户端（公钥在证书里面，私钥由服务器持有）。
	4. 客户端收到数字证书之后，会验证证书的合法性。如果证书验证通过，就会生成一个随机的对称密钥，用证书的公钥加密。
	5. 客户端将公钥加密后的密钥发送到服务器。
	6. 服务器接收到客户端发来的密文密钥之后，用自己之前保留的私钥对其进行非对称解密，解密之后就得到客户端的密钥，然后用客户端密钥对返回数据进行对称加密，酱紫传输的数据都是密文啦。
	7. 服务器将加密后的密文返回到客户端。
	8. 客户端收到后，用自己的密钥对其进行对称解密，得到服务器返回的数据。
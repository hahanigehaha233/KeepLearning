## HTTP相关问题

#### 一次http请求所经历的步骤

1. 在进行通信之前，浏览器要与web服务器建立tcp连接。
2. 发送请求命令，如 GET/sample/hello.jsp
3. 发送请求之后，以头信息的形式发送一些其他信息，最后以一个空白行来通知web服务器发送完成。
4. web服务器会发送应答信息，如200 等。
5. web服务器发送应答头信息，如果第三步一样。
6. 以空白行结束之后，开始发送数据。
7. 关闭TCP连接。

----

#### 1.0和1.1的区别
1. **HTTP1.0是短连接，HTTP1.1是长连接**。 HTTP1.0规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪每个客户也不记录过去的请求。如果一个html包含多个图片或资源时需要多次与服务器建立连接。 HTTP 1.1支持持久连接，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟。一个包含有许多图像的网页文件的多个请求和应答可以在一个连接中传输，但每个单独的网页文件的请求和应答仍然需要使用各自的连接。HTTP 1.1采用了流水线的持久连接，即客户端不用等待上一次请求结果返回，就可以发出下一次请求，但服务器端必须按照接收到客户端请求的先后顺序依次回送响应结果，以保证客户端能够区分出每次请求的响应内容，这样也显著地减少了整个下载过程所需要的时间。
2. **HTTP1.0不支持Host请求头字段**，WEB浏览器无法使用主机头名来明确表示要访问服务器上的哪个WEB站点，这样就无法使用WEB服务器在同一个IP地址和端口号上配置多个虚拟WEB站点。在HTTP1.1中增加Host请求头字段后，WEB浏览器可以使用主机头名来明确表示要访问服务器上的哪个WEB站点，这才实现了在一台WEB服务器上可以在同一个IP地址和端口号上使用不同的主机名来创建多个虚拟WEB站点。
3. **HTTP/1.1增加了OPTIONS方法，它允许客户端获取一个服务器支持的方法列表。**


---

#### GET和POST
1. GET 和 POST 数据内容是一模一样的，只是位置不同，一个在 URL 里，一个在 HTTP 包的包体里
2. GET在浏览器回退的时候时无害的，而POST会再次请求
3. GET表示只会获取数据，而POST可能改变资源
4. GET请求会被浏览器主动cache，而POST不会，除非手动设置。
5. GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
6. GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。

---

#### 状态码
1xx：临时响应
2xx：表示响应成功
- 200 （成功） 服务器已成功处理了请求。 通常，这表示服务器提供了请求的网页。
- 201 （已创建） 请求成功并且服务器创建了新的资源。
- 202 （已接受） 服务器已接受请求，但尚未处理。
- 203 （非授权信息） 服务器已成功处理了请求，但返回的信息可能来自另一来源。
- 204 （无内容） 服务器成功处理了请求，但没有返回任何内容。
- 205 （重置内容） 服务器成功处理了请求，但没有返回任何内容。
- 206 （部分内容） 服务器成功处理了部分 GET 请求。
3xx：表示重定向，完成请求需要进一步操作
4xx：请求错误
- 404 （未找到） 服务器找不到请求的网页。
- 403 （禁止） 服务器拒绝请求。
5xx：服务器错误
- 502 （错误网关） 服务器作为网关或代理，从上游服务器收到无效响应。
- 503 （服务不可用） 服务器目前无法使用（由于超载或停机维护）。 通常，这只是暂时状态。



----

[HTTP 常见面试题](https://zhuanlan.zhihu.com/p/85363975)
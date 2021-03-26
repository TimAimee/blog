# HTTP

# HTTP/0.9

	只接受GET一种请求方法，
	没有在通讯中指定版本号
	不支持请求头

# HTTP/1.0

	指定版本号的HTTP协议版本。
	长连接的话，头部指定要加Connection: keep-alive

	备注：keep-alive --> 一个http产生的tcp连接在传送完最后一个响应后，还需要hold住keepalive_timeout秒后，才开始关闭这个连接。

# HTTP/1.1（1999年）

	默认采用持续连接Connection: keep-alive
	很好地配合代理服务器工作
	支持以管道方式（A->(B代理)->C变A------>C）在同时发送多个请求
	
# HTTP/2(2015年正式发布)
 
	HTTP/2 采用了新的方法来编码、传输客户端与服务器间的数据，未加密

# HTTP2与HTTPS的区别

	HTTP2是下一代HTTP协议，内容做了一定的改变，如二进制分帧、多路复用、头部压缩、服务器推送等
	HTTPS使用的则是加密传输,在HTTP协议下增加了一层SSL协议，通过加密整个通信线路进行加密来防止通信内容被窃听、篡改或伪装。




 



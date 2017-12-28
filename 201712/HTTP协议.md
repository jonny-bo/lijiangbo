# HTTP协议

## 描述
- 时间：2017.12.28
- 题目：HTTP协议

## 内容

1.相关简介
HTTP协议，超文本传输协议，诞生于1989年3月。CERN（欧洲核子研究组织）的一个博士提出了共享知识设想：借助文档之间的关联性形成超文本（巨大型文本），连接成为WWW（万维网）。
WWW：HTML、HTTP、URL。
1990年11月，CERN成功研发出世界第一台WEB服务器和WEB浏览器。1992年9月，日本第一个网站上线。
1993年1月，NCSA（美国国家超级计算机应用中心）研发出Mosaic浏览器。下半年，Mosaic浏览器Windows版本和Macintosh面世，使用CGI技术的NCSA Web服务器出现。
1994年12月，网景通讯公司发布了Netscape Navigator 1.0。
1995年，微软公司发布Internet Explorer1.0 和2.0。WEB服务器标准Apache发布，浏览器大战。
2000年，网景公司衰落，浏览器大战结束。
2004年，Mozilla基金会发布FireFox浏览器。
Internet Explorer 浏览器 6、7、8、9、10
Chrome、Opera、Safari等

1990年，HTTP问世
1996年5月，正式命名HTTP/1.0公布 RFC1945；RFC：Request for Comment， 征求修订意见书，HTTP协议技术标准文档
1997年1月，HTTP/1.1公布，目前的主流，HTTP/2.0

2.URI和URL
  URI：Uniform Resource Identifier，统一资源标识符，标识某一互联网资源
  URL：Uniform Resource Locator，统一资源定位符，标识资源在互联网上所处的位置。URL是URI的子集。

  http://user:pass@www.baidu.com:80/news/index.html?uid=1#para1
  
  http://：协议名
  user:pass：登陆信息，（认证）
  www.baidu.com：服务器地址
  80：端口号
  news/index.html：文件路径
  uid=1：表单数据
  para1：锚点
3.HTTP方法
   GET：获取资源
   请求#####################################################################
   GET /index.html?age=23 HTTP/1.1
   Host: 127.0.0.1
   响应#####################################################################
   返回 index.html 资源
   ########################################################################

   POST：传输实体主体
   PUT：传输文件
   HEAD：获取报文首部，确认URI的有效性和资源更新的日期时间等
   DELETE：删除文件
   OPTIONS：询问支持的方法
   TRACE：追踪路径
   CONNECT：要求用隧道协议连接代理，SSL和TLS

4.持久连接和管线化
   建立一次TCP链接，进行多次HTTP通信。多次HTTP请求可以同时发送，不必等待确认后再发送

5.Cookie
   HTTP是一种无状态协议，客户端与服务器之间的通信状态不进行保存。服务器在响应客户端请求时生成cookie，在响应报文首部加入Set-Cookie：[cookie值]，返回给客户端，之后的HTTP通信都会在首部加入Cookie值告知服务器自己身份

6.HTTP报文
   ########################################################################
   +----------------------------+
   |           报文首部         |
   |----------------------------|
   |            CR+LF           |
   |----------------------------|
   |           报文主体         |
   +----------------------------+
   ########################################################################

   HTTP报文示例
   ########################################################################
   HTTP/1.1 200 OK 
   Server: Powered By LEOS-Studio
   Content-Type: text/html
   Accept-Ranges: bytes
   Set-Cookie: 123
   Content-Length: 150

   <html xmlns="http://www.w3.org/1999/xhtml">
       <head>
           <title ></title>

       </head>
       <body>
           <div>Welcome to LEOS Studio</div>
        </body>
   </html>
   ########################################################################

7.状态码
   1XX：信息性状态码，接受的请求正在处理
   2XX：成功状态码，请求正常处理完毕
        200 OK:请求已正常处理
        204 No Content:请求处理成功，但没有资源可返回
        206 Partial Content:成功处理对部分资源的请求
   3XX：重定向状态码，需要进行附加操作以完成请求
        301 Moved Permanaently:永久性重定向，请求的资源已经被分配性的URI
        302 Found：临时性重定向
   4XX：服务器无法处理请求
        400 Bad Request：错误的请求
        403 Forbidden：请求的资源被服务器拒绝
        404 Not Found：请求的资源不存在
   5XX：服务器错误代码，服务器处理请求错误
        501 Internal Server Error：服务器内部错误
        503 Service Unavailable:服务器超负载或停机维护
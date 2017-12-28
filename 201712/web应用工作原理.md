# web应用工作原理

## 描述
- 时间：2017.12.28
- 题目：web应用工作原理

## 内容

web应用工作原理
解释：
1.使用HTTP协议进行数据通讯的计算机应用 
2.B/S架构。Browser/Server，通用的标准化浏览器和服务器工作平台，开发者使用脚本（对现有应用的补充和增强，类似于插件）进行应用开发的工作模式。
   浏览器：Safari、Chrome、FireFox、Opera ；脚本HTML、CSS、javaScript
   服务器：Apache、IIS、NodeJS、Tomcat；脚本：ASP、ASP.NET、VBScript、PHP、JSP、JS
基本原理
1.客户端（浏览器）向服务器发送请求，一般使用GET，POST。
2.请求被封装在HTTP报文中，HTTP报文被封装在TCP报文中，端口是计算机随机从本地空闲端口中选取的。HTTP报文中指定了TCP链接的端口号（URI），一般默认为80
3.数据被层层封装，一直到底层的比特流，通过物理通信链路进行传输
4.服务器接收到客户端请求。请求数据包被层层解封装，由操作系统内核交送给监听此端口的应用，一般是Apache、IIS等。
5.Apache对下层传送上来的TCP数据包进行解封装，解析出HTTP报文。
   HTTP报文示例
   ########################################################################
   GET /index.html HTTP/1.1
   Host: 127.0.0.1
   Connection: keep-alive
   Cache-Control: max-age=0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
   User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.111 Safari/537.36
   Accept-Encoding: gzip, deflate, sdch
   Accept-Language: zh-CN,zh;q=0.8
   Cookie: 123
   ########################################################################

6.处理客户端的请求。
   静态资源：如果分析后发现客户端请求的是静态资源，服务器直接查找请求的资源。如果存在，将资源封装成HTTP包；如果不存在，则将错误信息封装为HTTP包。                    
   动态资源：如果分析后发现客户端请求的是脚本文件，服务器查找脚本文件是否存在，如果存在，通过CGI接口将其交给相应的脚本解释器，由脚本解释器解释执行完毕后，通过CGI接口向Apache服务器返回资源；如果不存在，如果不存在，则将错误信息封装为HTTP包。
   CGI：Common GateWay Interface,通用网关接口，是指WEB服务器在接收到客户端发送过来的请求后转发给程序的一组机制。
7.响应客户端请求。通过已经建立的TCP链接通道将HTTP报文返回给客户端。
8.客户端接收到数据后进行解析。对HTTP报文进行分析，报文首部+报文主体
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
           <title></title>

       </head>
       <body>
           <div>Welcome to LEOS Studio</div>
        </body>
   </html>
   ########################################################################
   报文首部->浏览器与服务器之间相关参数属性的约定
   报文主体->HTML文本、js文本、css文本、图片、其他数据等
   9.渲染数据，呈现给用户。扫描HTML文档结构（需要的js、css、图片等资源会实时向服务器发送GET请求）、计算对应的CSS样式并生成RenderTree，然后根据RenderTree进行布局和绘制
   ########################################################################
                                       DOM
                                        |                 Layout
   HTML       --> HTML Parser  --> DOM Tree    --\          |
                          |--> Attachment -->Render Tree     |--> Painting -->Display
   StyleSheet --> CSS Parser   --> Style Rules --/

   ########################################################################
   Recalculate Style：计算如何操作DOM结构、并计算节点的最终样式规则。
   Layout：根据节点CSS规则，计算节点在屏幕上的位置和尺寸。由于页面是按着文档流从上到下、从左至右布局的，一个节点发生变化，可能使得多个节点重新布局。
   Update Layer Tree：一个页面可能有多个渲染层，Layer Tree用来维护各个渲染层的顺序。
   Paint：绘制本质上就是填充像素的过程。包括绘制文字、颜色、图像、边框和阴影等，由此确定一个DOM元素所有的可视效果。绘制一般是在多个层上同时进行。
   Composite Layer：在多个层上分别完成绘制后，浏览器会按各个绘制层的正确顺序进行合并，最终显示在屏幕上。
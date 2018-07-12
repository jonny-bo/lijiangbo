# sapi 

## Apache模块

PHP以mod_php5模块的形式集成在Apache，接收Apache传递过来的PHP文件请求，并处理这些请求，最后返回给Apache结果。PHP模块通过注册apache2 `ap_hook_post_config`挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。

## Apache的运行过程

Apache的运行分为启动阶段和运行阶段。

这个阶段包括配置文件解析(如http.conf文件)、模块加载(如mod_php，mod_perl)和系统资源初始化（例如日志文件、共享内存段、数据库连接等）等工作。
* Apache为了获得系统资源最大的使用权限，将以特权用户root（*nix系统）或超级管理员Administrator(Windows系统)完成启动
* 整个过程处于一个单进程单线程的环境中

Apache对HTTP的请求可以分为连接、处理和断开连接三个大的阶段。同时也可以分为11个小的阶段，依次为： 
* Post-Read-Request
* URI Translation
* Header Parsing
* Access Control
* Authentication
* Authorization
* MIME Type Checking
* FixUp
* Response
* Logging
* CleanUp

## Apache Hook机制

Apache 的Hook机制是指：Apache 允许模块(包括内部模块和外部模块，例如mod_php5.so，mod_perl.so等)将自定义的函数注入到请求处理循环中。

模块可以在Apache的任何一个处理阶段中挂接(Hook)上自己的处理函数，从而参与Apache的请求处理过程。

## Apache常用对象

`httpd.h` 文件包含了Apache的所有模块都需要的核心API。

- `request_rec` 对象 当一个客户端请求到达Apache时，就会创建一个`request_rec`对象，当Apache处理完一个请求后，与这个请求对应的`request_rec`对象也会随之被释放。 `request_rec`对象包括与一个HTTP请求相关的所有数据，并且还包含一些Apache自己要用到的状态和客户端的内部字段。

- `server_rec` 对象 `server_rec`定义了一个逻辑上的WEB服务器。如果有定义虚拟主机，每一个虚拟主机拥有自己的`server_rec`对象。`server_rec`对象在Apache启动时创建，当整个httpd关闭时才会被释放。它包括服务器名称，连接信息，日志信息，针对服务器的配置，事务处理相关信息等。`server_rec`对象是继`request_rec`对象之后第二重要的对象。

- `conn_rec` 对象 `conn_rec`对象是TCP连接在Apache的内部实现。它在客户端连接到服务器时创建，在连接断开时释放。

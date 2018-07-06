# php 内核学习 day2

## 生命周期和Zend引擎

PHP开始执行以后会经过两个主要的阶段：处理请求之前的开始阶段和请求之后的结束阶段。

 开始阶段:
 
1.  `MINIT`， 模块初始化阶段，在整个SAPI生命周期内该过程只进行一次。
2.  `RINIT`，模块激活阶段，该过程发生在请求阶段，每次请求之前都会进行模块激活。

模块在实现时可以通过如下宏来实现这些回调函数：
```
PHP_MINIT_FUNCTION(myphpextension)
{
    // 注册常量或者类等初始化操作
    return SUCCESS; 
}
```

请求到达之后PHP初始化执行脚本的基本环境，然后PHP会调用所有模块的RINIT函数，在这个阶段各个模块也可以执行一些相关的操作，模块的RINIT函数和MINIT回调函数类似：

```
PHP_RINIT_FUNCTION(myphpextension)
{
    // 例如记录请求开始时间
    // 随后在请求结束的时候记录结束时间。这样我们就能够记录下处理请求所花费的时间了
    return SUCCESS; 
}
```

结束阶段:
1. `RSHUTDOWN`，请求结束后停用模块，对应`RINIT`。
2. `MSHUTDOWN`，一个在SAPI生命周期结束时关闭模块，对用`MINIT`。

```
PHP_RSHUTDOWN_FUNCTION(myphpextension)
{
    // 例如记录请求结束时间，并把相应的信息写入到日至文件中。
    return SUCCESS; 
}

```

单进程SAPI生命周期:

CLI/CGI模式的PHP属于单进程的SAPI模式。这类的请求在处理一次请求后就关闭。

![image](http://www.php-internals.com/images/book/chapt02/02-01-01-cgi-lift-cycle.png)

- 启动
- 初始化若干全局变量
- 初始化若干常量
- 初始化Zend引擎和核心组件
- 解析php.ini
- 全局操作函数的初始化
- 初始化静态构建的模块和共享模块(MINIT)
- 禁用函数和类
- ACTIVATION
- 激活Zend引擎
- 激活SAPI
- 环境初始化
- 模块请求初始化
- DEACTIVATION:
    1. 调用所有通过register_shutdown_function()注册的函数。这些在关闭时调用的函数是在用户空间添加进来的。
    2. 执行所有可用的__destruct函数。 这里的析构函数包括在对象池（EG(objects_store）中的所有对象的析构函数以及EG(symbol_table)中各个元素的析构方法。
    3. 将所有的输出刷出去。
    4. 发送HTTP应答头。这也是一个输出字符串的过程，只是这个字符串可能符合某些规范。
    5. 遍历每个模块的关闭请求方法，执行模块的请求关闭操作。
    6. 销毁全局变量表（PG(http_globals)）的变量。
    7. 通过zend_deactivate函数，关闭词法分析器、语法分析器和中间代码执行器。
    8. 调用每个扩展的post-RSHUTDOWN函数。只是基本每个扩展的post_deactivate_func函数指针都是NULL。
    9. 关闭SAPI，通过sapi_deactivate销毁SG(sapi_headers)、SG(request_info)等的内容。
    10. 关闭流的包装器、关闭流的过滤器。
    11. 关闭内存管理。
    12. 重新设置最大执行时间。
- flush
- 关闭Zend引擎
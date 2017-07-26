# ES-X8 ApiBundle接口开发

## 前言

x8之前api开发是用的[Silex](https://silex.symfony.com/doc/2.0/intro.html)一个基于symfony组件小型的api开发框架，集成到主系统去的，在<code>web/app_dev.php</code> or <code>web/app.php</code>根据url路径是否包含**/api**关键字段决定是否进去api框架的程序中．代码如下：

```
if ((strpos($_SERVER['REQUEST_URI'], '/api') === 0) || (strpos($_SERVER['REQUEST_URI'], '/app_dev.php/api') === 0)) {
    define('API_ENV', 'dev');
    include __DIR__.'/../api/index.php';
    exit();
}
```
x8系统引入了ApiBundel的模式，不过还是兼容旧的api接口，代码如下：

```
if (isOldApiCall()) {
    define('API_ENV', 'dev');
    include __DIR__.'/../api/index.php';
    exit();
}

function isOldApiCall()
{
    return (!(isset($_SERVER['HTTP_ACCEPT']) && $_SERVER['HTTP_ACCEPT'] == 'application/vnd.edusoho.v2+json'))
    && ((strpos($_SERVER['REQUEST_URI'], '/api') === 0) || (strpos($_SERVER['REQUEST_URI'], '/app_dev.php/api') === 0));
}
```
对于<code>isOldApiCall()</code>我停留了下看看，刚开始不明所以，在后面遇到一个坑，突然想起这个，恍然大悟啊．所以我这里稍微解释下．

**新的api接口规定在调用接口的时候需要在http header里面加入参数：ACCEPT 值为：application/vnd.edusoho.v2+json**

不然一直调用接口会提示路由不存在的，然而旧的api路由是不需要的．所以在这里在以前判断是否接口路由**包含/api**路径的基础上多了一个排除设置了<code>HTTP_ACCEPT</code>参数并参数值为<code>application/vnd.edusoho.v2+json</code>的路由．
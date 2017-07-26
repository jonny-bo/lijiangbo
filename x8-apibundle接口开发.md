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

## ApiBundle跟踪解读

先去看看路由配置<code>app/config/routing.yml</code>文件，发现如下行代码：

```yml
api:
    resource: "@ApiBundle/Controller/"
    type:     annotation
    prefix:   /api

```
发现路由带**/api**前缀的接口都是访问<code> ApiBundle/Controller/ </code>下的资源，访问方式是通过注解去解读路由．我们去ApiBundle/Controller/下面看看，发现了<code> EntryPointController </code>如下：

```php
<?php

namespace ApiBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;

class EntryPointController extends Controller
{
    /**
     * @Route("/{res1}")
     * @Route("/{res1}/{slug1}")
     * @Route("/{res1}/{slug1}/{res2}")
     * @Route("/{res1}/{slug1}/{res2}/{slug2}")
     * @Route("/{res1}/{slug1}/{res2}/{slug2}/{res3}")
     * @Route("/{res1}/{slug1}/{res2}/{slug2}/{res3}/{slug3}")
     * @Route("/{res1}/{slug1}/{res2}/{slug2}/{res3}/{slug3}/{res4}")
     * @Route("/{res1}/{slug1}/{res2}/{slug2}/{res3}/{slug3}/{res4}/{slug4}")
     */
    public function startAction(Request $request)
    {
        $kernel = $this->container->get('api_resource_kernel');
        return new JsonResponse($kernel->handle($request));
    }
}
```
注意这里的**注解**，发现它只是把路由规范成变量去匹配，符合让面所有条件的路由都会到<code> @ApiBundle:EntryPoint:start </code>控制器去．比如我的路由是<code> /api/user/2 </code> 匹配到第二个<code> @Route("/{res1}/{slug1}") </code>　res1变量的值为user，slug1的值为2．通过后边的解读，发现了res存储的是资源名，slug存储的是对应资源的标识．
- 这里面路由规范是符合[REST API](http://edusoho.pages.codeages.net/edusoho-develop-guide/api/)规范的，对外资源一般都是这种格式．网上也有对[restful api](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)的讲解.

- 接下看看控制器里面具体干了什么吧：

<code> $kernel = $this->container->get('api_resource_kernel'); </code> 从symfony容器里面取出**api_resource_kernel**，到哪里去找呢．看看<code>/src/ApiBundle/Resources/config/services.yml</code>文件．里面配置了关于ApiBundle的一些类到symfony服务容器里面，找到如下行．

```
api_resource_kernel:
    class: ApiBundle\Api\ResourceKernel
    arguments: ['@service_container']
```
通过class类名大概知道这个类是api的资源管理内核，找到这个类的handle方法，最后控制器返回的是这个的值：

```
public function handle(Request $request)
{
    $this->container->get('api_firewall')->handle($request);

    $this->biz['user'] = $this->container->get('security.token_storage')->getToken()->getUser();

    if ($this->isBatchRequest($request)) {
        return $this->batchRequest($request);
    } else {
        return $this->singleRequest($request);
    }
}
```

- 第一步是把请求通过api防火墙过滤，这样做是为了安全性考虑．有兴趣可以去找找看这个类，这里就不做详解了，大概就是一些基于basic，OAuth2，session，XAuthToken等认证等等．

- 第二步是通过<code> security.token_storage </code>获取token并且拿到当前登录认证的用户，并且设置到biz里面去．

- 判断是否是批量请求，是的话返回<code> $this->batchRequest($request); </code>，否则返回<code> $this->singleRequest($request); </code>

在来看看singleRequest干了什么吧，批量请求一般也用不上的．

```
private function singleRequest(Request $request)
{
    $apiRequest = new ApiRequest($request->getPathInfo(), $request->getMethod(), $request->query, $request->request, $request->headers);
    return $this->handleApiRequest($apiRequest);
}

private function handleApiRequest(ApiRequest $apiRequest)
{
    $pathMeta = $this->pathParser->parse($apiRequest);
    $resourceProxy = $this->resManager->create($pathMeta);

    $this->container->get('api_authentication_manager')->authenticate($resourceProxy, $pathMeta->getResMethod());

    return $this->invoke($apiRequest, $resourceProxy, $pathMeta);
}
```
首先把request请求的信息传递到自己封装的一个类<code>ApiRequest</code>里面去，然后<code>handleApiRequest()</code>处理请求．过程如下：

- 通过pathParser路径解析类解析api请求(pathParser在构造函数里面已经初始化过了)

具体代码可以自己看，解析过程是，先拿到路径信息，然后通过路径解析路由<code>@Route("/{res1}/{slug1}/{res2}/{slug2}/{res3}/{slug3}/{res4}/{slug4}")</code>把这里面的第奇数位参数放到构造的<code>PathMeta</code>类里面的属性<code>resNames</code>数组里面，偶数位的参数放到<code>slugs</code>数组里面．(PathMeta类是用来存放解析后的结果的，还有对应到对应的资源类的方法去)．

- 通过解析路径获取到pathMeta信息后，创建resourceProxy(对应的资源代理类，资源代理里面集成了字段过滤的工厂方法，具体代码可以自己看)．

在<code>$resourceProxy = $this->resManager->create($pathMeta);
</code>创建资源代理resourceProxy过程中，通过pathMeta里面设定好的规则：

pathMeta的resNames里面，数组第一个当作文件夹名称，命名空间是ApiBundle\Api\Resource\.数组第一个值，然后把里面所有的值拼接成驼峰的类名．例如：
```

    $resNames = array('user'); 
    //clsaaName = "ApiBundle\Api\Resource\User\User"
    
    $resNames = array('classroom', 'course'); 
    //clsaaName = "ApiBundle\Api\Resource\Classroom\ClassroomCourse"

    $resNames = array('classroom', 'course', 'member'); 
    //clsaaName = "ApiBundle\Api\Resource\Classroom\ClassroomCourseMember"

    $resNames = array('classroom', 'course_student'); 
    //clsaaName = "ApiBundle\Api\Resource\Classroom\ClassroomCourseStudent"
```

获取资源类名．方法名的获取是根据请求的方法来对应访问的．加入你是post请求，根据restful代表你是要创建一个资源，则访问资源的add()方法．具体的对应规则如下：

```

    private $singleMap = array(
        'GET' => AbstractResource::METHOD_GET,
        'PATCH' => AbstractResource::METHOD_UPDATE,
        'DELETE' => AbstractResource::METHOD_REMOVE
    );

    private $listMap = array(
        'GET' => AbstractResource::METHOD_SEARCH,
        'POST' => AbstractResource::METHOD_ADD,
        'DELETE' => AbstractResource::METHOD_REMOVE
    );

    //AbstractResource对应的属性如下

    const METHOD_SEARCH = 'search';
    const METHOD_GET = 'get';
    const METHOD_ADD = 'add';
    const METHOD_REMOVE = 'remove';
    const METHOD_UPDATE = 'update';

    //获取method方法如下
    public function getResMethod()
    {
        $isSingleMethod = ($this->resNames[0] == 'me' && count($this->resNames) - 1 == count($this->slugs)) || (count($this->resNames) == count($this->slugs));
        if ($isSingleMethod) {
            return $this->singleMap[$this->httpMethod];
        } else {
            return $this->listMap[$this->httpMethod];
        }
    }
```
详细过程请仔细看<code>/src/ApiBundle/Api/PathMeta.php</code>里面的<code>getResMethod()</code>方法

**特殊的路由无法满足restful api接口规范在解析路径时候里殊处理过**
例如/api/me/cash_count

```

    private function parsePathInfo($pathMeta, $pathInfo)
    {
        $pathExplode = explode('/', str_replace(ApiBundle::API_PREFIX, '', $pathInfo));
        //默认第一个是资源名称
        $nextIsResName = 1;
        foreach ($pathExplode as $part) {
            if ($part == '') {
                continue;
            }

            if ($part == 'me') {
                $pathMeta->addResName($part);
                continue;
            }

            if ($part == 'app') {
                $pathMeta->addResName($part);
                continue;
            }

            if ($part == 'auth') {
                $pathMeta->addResName($part);
                continue;
            }

            if ($part == 'user_profile') {
                $pathMeta->addResName($part);
                continue;
            }

            if ($nextIsResName) {
                $pathMeta->addResName($part);
                $nextIsResName = 0;
            } else {
                $pathMeta->addSlug($part);
                $nextIsResName = 1;
            }
        }
    }
```

## 总结

- 路由不需要创建，有既定的规则/资源名1/资源1标识(id)/资源名2/资源2标识(id)...
- 资源类对应路由的resNames值，对应: 资源名1/资源名1.资源名2
- 请求的方法(POST,GET,UPDATE...)和参数类型，对应资源的函数入口
- 接口资源类都继承自AbstractResource，里面封装哦一些通用方法
- 所有的异常都会被捕获，然后返回json格式**错误的响应**
- 不需要认证的接口在方法前面加上注解

```
/**
 * @ApiConf(isRequiredAuth=false)
 */
```
- 资源的返回格式参考[ES api接口指南](http://edusoho.pages.codeages.net/edusoho-develop-guide/api/) (需要帐号密码)
---
layout: Services 和 Listeners
title: symfony学习日常第一章
date: 2017-11-10 9:00:08
tags: symfony
---

# symfony学习日常第一章

## 章节介绍

本次学习主要是研究symfony的Services 和 Listeners讨论基于服务和侦听器的基础。
如何在symfony中拓展和使用。

## Services服务
Symfony2框架是基于服务为基础搭建的，框架自身已经预定义了大概200多个服务，通过控制台命令：`app/console container:debug`　查看被定义的服务。`app/console container:debug　<service_name>`来查看定义服务的具体信息。

服务只是给定类的特定实例，例如当你在控制器中访问`$this->get('doctrine ')`时，就能得到一个`EntityManager`类的一个实例，而你不必自己创建这个实例。实际上要得到这个`EntityManager`类的实例并不简单，通常我们实例化一个是通过`new EntityManager(<require_tag...>);`来得到的，这里在实例化对象可能需要更多的参数，需要连接到数据库，
其他一些配置，等等。通过service服务容器管理我们可以减少各个对象之间的依赖关系，让服务容器去把类实例化并且把实例化需要的对象或者参数也注入进去。

（关于容器如何解决依赖，如何得到类的实例可以去了解一下`依赖注入，控制反转`等概念，还有一个简单的php容器`pimple`可以去看看．）

一些目前在Symfony2默认服务如下：

- `The annotation reader` 注释阅读器
- `Assetic the asset management library` 资产管理库
- `The event dispatcher` 事件分配器
- `The form widgets and form factory` 表单小部件和表单工厂
- `The Symfony2 Kernel and HttpKernel` Symfony2内核和HttpKernel
- `Monolog the logging library` 日志库
- `The router` 路由器
- `Twig the templating engine` twig模板引擎

symfony 框架让我们能够简单的去创建一个服务，并使用它．如果你的控制器代码特别长，包含太多的业务逻辑，那么就需要用服务去定义一个类去处理，这样就重构了你的代码，让人看起来更加的简洁清晰．

下面是创建一个服务例子：

  - 以下是php代码，这是在没有定义services情况下使用的方法:

```php
    //获取客户端ip地址
    $ip = $this->get('request')->getClientIp();
    
    //实例化地址转换工具类
    $adapter = new \Geocoder\HttpAdapter\CurlHttpAdapter();
    $geocoder = new Geocoder();
    $geocoder->registerProviders(array(
        new \Geocoder\Provider\FreeGeoIpProvider($adapter),
    ));

    //调用工具类解析ip到地理位置
    $result = $geocoder->geocode($ip);

```
  - 定义service.yml，用依赖注入的方式实现以上php的重构

```yml
services:
    geocoder_adapter:
        class: Geocoder\HttpAdapter\CurlHttpAdapter
        public: false
    geocoder_provider:
        class: Geocoder\Provider\FreeGeoIpProvider
        public: false
        arguments: [@geocoder_adapter]
    geocoder:
        class: Geocoder\Geocoder
        calls:
        - [registerProviders, [[@geocoder_provider]]]
```
  我们实际上定义了三种服务，`geocoder_adapter`，`geocoder_provider`需要一个`geocoder_adapter`的实例当作构造函数的参数，`geocoder`在初始化之后会执行`registerProviders`方法，并且这个方法需要一个`geocoder_provider`的实例当做参数.

  我们用arguments: [@+service_name, ...]将参考服务作为一个参数
  我们可以做的不仅仅是定义新类（参数）；我们还可以实例化后调用类上的方法（calls参数配置）。

  甚至可以设置，直接声明为公共属性时（默认为共有）。
  我们将前两项服务列为私有（public: false）。这意味着他们不会在我们的控制器访问。然而，它们可以被依赖性所利用。注入容器（DIC）被注入到其他服务中。

  - 调用定义服务实现上面的代码

```
    //获取客户端ip地址
    $ip = $this->get('request')->getClientIp();
    //调用ip转换地址服务解析ip
    $result = $this->get('geocoder')->geocode($ip);
```

  - 我们定义一个服务类来获取用户的地理位置

```
namespace AppBundle\Geo;

use Geocoder\Geocoder;
use Symfony\Component\HttpFoundation\Request;

class UserLocator
{
    protected $geocoder;
    protected $userIp;

    public function __construct(Geocoder $geocoder, Request
    $request) {
        $this->geocoder = $geocoder;
        $this->userIp = $request->getClientIp();
    }

    //获取用户的地理位置
    public function getUserGeoBoundaries($precision = 0.3) {
        // 找到用户的坐标
        $result = $this->geocoder->geocode($this->userIp);
        $lat = $result->getLatitude();
        $long = $result->getLongitude();
        $latmMax = $lat + 0.25; // (大约 25km)
        $latMin = $lat - 0.25;
        $longMax = $long + 0.3; // (大约 25km)
        $longMin = $long - 0.3;

        return [
            'latmMax' => $latmMax,
            'latMin' => $latMin,
            'longMax' => $longMax,
            'longMin' => $longMin
        ];
    }
}
```
  # services.yml
```
services:
    user_locator:
        class: AppBundle\Geo\UserLocator
        scope: request
        arguments: [@geocoder, @request]
```
注意，我们在这里定义了作用域`scope`。默认情况下，DIC有两个作用域：容器Container和原型Prototype，其中框架还添加了第三个名为请求。下表显示了他们之间的差异：

|     Scope   |  Differences |
| :---------: | :----------: |
| Container   |　所有的调用 $this->get('service_name') 返回同一个实例（单例）|
| Prototype   |　每次调用 $this->get('service_name') 返回一个新的实例 |
| Request     |  每次调用$this->get('service_name')　返回在请求的服务相同的实例（区别在twig可能调用的subrequests）|


服务标签Tagging services：

在上面的示例中，我们创建了一个`user_locator`服务依赖于地理编码服务（`geocoder`）。但是，有许多可能的方法来定位用户。

例如有`freegeoip_geocoder`，`random_geocoder`两种地理编码服务

```
    freegeoip_geocoder:
        class: AppBundle\Geo\FreeGeoIpGeocoder
        arguments: [@geocoder]
        tags:
            - { name: app.geocoder }
    random_geocoder:
        class: AppBundle\Geo\RandomLocationGeocoder
        tags:
            - { name: app.geocoder }
```

我们可能在`user_locator`中使用最精确的一个地理编码服务，我们不能把所有这些服务在我们的类的构造函数直接使用（不利于扩展），所以我们要修改我们的`UserLocator`类有一个`addGeocoder`方法如下：

```
namespace AppBundle\Geo;

use Geocoder\Geocoder;
use Symfony\Component\HttpFoundation\Request;

class UserLocator
{
    protected $userIp;
    protected $geocoders = [];

    public function __construct(Request $request) {
        $this->userIp = $request->getClientIp();
    }

    //获取用户的地理位置
    public function getUserGeoBoundaries() {
        return $this->getBestGeocoder()->geocode($this->userIp);
    }

    public function addGeocoder(Geocoder $geocoder)
    {
        $this->geocoders[] = $geocoder;
    }

    //挑选最精确的地理编码
    public function getBestGeocoder(){/* ... */}
}
```

  通知我们要添加标签服务（`tags`）的DIC（`依赖注入`）不能通过配置完成。这是在编译DIC时通过`Compiler passes`编译器传递完成的。
  用于如下：

```
namespace AppBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Compiler
 \CompilerPassInterface;
use Symfony\Component\DependencyInjection\Reference;

class UserLocatorPass implements CompilerPassInterface
{
    public function process(ContainerBuilder $container) {
        if (!$container->hasDefinition('user_locator')) {
            return;
        }
        $service_definition = $container->getDefinition('user_locator');
        $tagged = $container->findTaggedServiceIds('app.geocoder');
        foreach ($tagged as $id => $attrs) {
            $service_definition->addMethodCall(
                'addGeocoder',
                [new Reference($id)]
            );
        }
    }
}

```

  在确认user_locator服务的存在后，我们找到相对应的标签`app.user_locator`所有的服务，使用`addMethodCall`添加`calls`参数（上面提到过，该参数会在实例化后执行定义在calls里面的方法，即`addGeocoder`）加载这些服务到'user_locator'。

  我们还可以在标签上定义自定义属性。因此，我们可以将每个请求的精度的配置如下，然后用编译器通过只提供最准确的地理编码的用户定位：

```
tags:
    - { name: app.geocoder, accuracy: 69 }
```

 在`UserLocatorPass`里面通过accuracy参数确认最精确地理编码服务，然后做一些事情．

 每当我们定义服务的YAML配置，symfony会内部基于该信息创建服务定义。通过添加一个编译器传递，我们可以动态修改这些服务定义（例如上面的`UserLocatorPass`就动态的修改了服务的定义）。


## Listeners监听器

侦听器是实现观察者设计模式的一种方法。在这种模式下，某一段代码并不立即执行。相反，它通知所有的观察者，它已经到达了一个给定的执行点，如果需要则允许这些观察者接管控制流。

在symfony中，我们通过事件使用观察者模式。任何类或函数都可以在事件合适时触发事件。事件本身可以在类中定义。这允许将更多信息传递给观察该事件的代码。框架本身将触发处理请求过程中不同节点的事件。这些事件如下：

  - kernel.request：此事件发生在一个请求发起时到进入控制器之前，它在内部用于填充请求（Request）对象。
  - kernel.controller：此事件发生后立即执行控制器（即将进入控制器）。它可以用来改变正在执行的控制器。
  - kernel.view：在执行控制器，如果控制器没有返回响应对象时触发此事件。例如，在渲染　twig模板视图时候触发。
  - kernel.response：此事件在响应发送出去之前触发。它可以用于在发出响应之前修改响应。
  - kernel.terminate：在响应发送后触发此事件出。它可以用来执行任何耗时的操作，而不是生成响应所必需的。
  - kernel.exception：此事件发生在框架捕获了一个异常被触发。
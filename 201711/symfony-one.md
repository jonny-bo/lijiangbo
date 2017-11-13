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

下面是一个一个地理位置的服务例子：




---
layout: symfony security
title: symfony security 安全机制
date: 2017-11-8 10:00:08
tags: symfony
---

# symfony security　安全机制学习

## 常规用户身份验证流程
  1. 登录表单输入用户`身份信息`（用户名，密码）
  2. 服务端接受请求，根据用户名查找用户信息（能够提供`身份信息的提供者`，通常为数据库的user表，文件等）
  3. 根据用户输入的密码和查找到的用户身份信息的密码做比较，`验证用户`
  4. 验证成功，存储当前用户到`seesion`或者`cookie`中保持用户的登录状态
  
  存在的隐患或者漏洞（思考）:

  1. 千万不要明文保存用户的密码, 应该使用加密算法`加密`
  2. 千万不要在cookie中存放用户的密码
  3. 不要让cookie有权限访问所有的操作（XSS攻击）
  ......

## symfony security组件 身份验证

  使用symfony security创建一个传统登录表单：
  
    1. 配置登录设置
    2. 创建一个用户实体类，实现接口`Symfony\Component\Security\Core\User\UserInterface`
    3. 创建一个身份信息的提供者UserProvider类实现接口`Symfony\Component\Security\Core\User\UserProviderInterface`
    4. 创建登录的路由，控制器和视图文件

    通过配置security.yml配置身份验证的模式，可以看到组成基础的验证流程需要配置：
    - encoders                  加密算法
    - providers                 身份信息的提供者
    - firewalls                 防火墙
      - pattern                 匹配经过防火墙的路径
      - form_login              选择表单方式提交验证
      - remember_me             记住我配置
      - logout                  登出配置
    - access_control　           访问控制

  其他的多种配置方式，参考学习[完整的默认配置](http://symfonychina.com/doc/current/reference/configuration/security.html)

  至此一个用户身份验证已经配置好了，到了这里可以发现，自己并没有写登录检查的任何逻辑，只是进行了配置和登录页面的显示而已，那么身份验证到底在哪里进行呢？

  身份验证的核心就是防火墙，用户身份认证是由防火墙完成的．防火墙的运作中，我们会发现，防火墙也是由以下几块积木组成的：

  - `Firewall listener`      
  - `Token & Token Storage`
  - `Authentication provider`
  - `User provider`
  - `Entry pointer`

  防火墙监听器跟所有其他监听`kernel.request`事件的监听器没有本质上的区别，都会去处理 `GetResponseEvent` 事件对象。这个事件对象简而言之就是个容器，里面包含了 `Request` 对象，并且可以设置一个 `Response` 对象。如果 `Response` 对象被设置，无视后面所有事件，直接被框架返回 `Response`。

  `Firewall Listener` 到底具体做什么？首先它判断 `Request`对象里是否包含身份认证信息，如果不包含，什么事都不做；如果包含，他会将 `Request` 里的用户登录信息抽取出来，创建一个 `token`。
  
  跟踪SecurityBundle：

  - `Symfony\Bundle\SecurityBundle\EventListener\FirewallListener`
  -　`Symfony\Component\Security\Http\Firewall`
      - `Symfony\Component\Security\Http\Firewall\ChannelListener`
          限制是否HTTPS/HTTP
      - `Symfony\Component\Security\Http\Firewall\ContextListener`
          session里读取token，刷新token
      - `Symfony\Component\Security\Http\Firewall\LogoutListener`
          检查当前请求是否为登出，执行登录逻辑(验证crsf_token，清除token)
          - `Symfony\Component\Security\Http\Logout\SessionLogoutHandle`
          - `Symfony\Component\Security\Http\RememberMe\TokenBasedRememberMeServices`
      - `Symfony\Component\Security\Http\Firewall\UsernamePasswordFormAuthenticationListener`
          用户名密码表单验证逻辑 ***关键逻辑***
          根据表单信息创建UsernamePasswordToken
          调用authenticationManager的authenticate方法验证UsernamePasswordToken
          - `Symfony\Component\Security\Core\Authentication\AuthenticationProviderManager`
              通过看支持的providers来认证token
              - `Symfony\Component\Security\Core\Authentication\Provider\DaoAuthenticationProvider`
              - `Symfony\Component\Security\Core\Authentication\Provider\UserAuthenticationProvider`
                  认证流程：token中取出username，根据username从数据提供者（UserProvider）加载user，checkPreAuth认证前检查用户（用户是否被禁...），checkAuthentication检验用户名密码，checkPostAuth检查用户证书是否过期，生成认证后的token
      - `Symfony\Component\Security\Http\Firewall\RememberMeListener`
          如果下次未登录访问，根据remeber配置，从cookie中读取认证信息，自动登录
      - `Symfony\Component\Security\Http\Firewall\AnonymousAuthenticationListener`
        
      - `Symfony\Component\Security\Http\Firewall\AccessListener`

## Security 组建介绍: (`vendor/symfony/symfony/src/Symfony/Component/Security`)
  - Core                组建核心
      - Authentication  认证
      - Authorization 　 授权
      - Encoder 　　　　　　　 加密
      - Event 　　　　　　　　　 事件
      - Exception       异常
      - Role            角色
      - User            用户
      - Validator       验证
  - Csrf               （Cross-site request forgery）跨站请求伪造
      - TokenGenerator  CSRF　token生成
      - TokenStorage    CSRF tokens容器
      - CsrfToken       CSRF　token
      - CsrfTokenManager创建并且验证CSRF tokens.
  - Guard               简单版身份验证组建（暂不介绍，自己学习）
  - Http                安全服务协议(调用组建构建安全服务)
      - Authentication  认证服务(Handler和Interfcae)
      - Authorization   授权服务
      - EntryPoint      入口点(定义了需要用户认证时，或者用户身份认证失败的时候的处理方式)
      - Event          　事件
          - InteractiveLoginEvent  
          - SwitchUserEvent        
      - Firewall        防火墙(Listener事件监听)　***核心***
      - Logout  　　　　　　　　登出服务(Handler和Interfcae)
      - RememberMe      记住我
      - Session         会话
## Endpoints:

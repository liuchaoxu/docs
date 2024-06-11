---
layout: doc
lang: zh-CN
title: JUC
titleTemplate: spring
description: 原理解析
#导航栏
#navbar: true
#侧边栏
sidebar: true
#侧边大纲默认在右侧 ，通过 aside 设置左侧或关闭，默认 true
aside: true
#右侧的大纲，默认显示是二级标题，通过设置 outline 实现多级标题
#注意:设置到六级标题可以用 'deep' ，关闭 false,此设置与 页面中的大纲 设置相同，会覆盖！
#outline: [2,3]
outline: [1,2,3,4,5,6]
#上次更新时间，默认开启，不想显示可以关闭
lastUpdated: false

#仅更改上/下页显示的文字，跳转还是按照侧边栏配置的读取的
#prev: '页面 | 更详细的页面配置'
#next: 'Markdown | 更详细的markdown'

#更改跳转链接
#  可更改成任意自己想跳转的文章
prev:
  text: '知识地图'
  link: '../知识地图'
#next:
#  text: 'Markdown'
#  link: '/markdown'

#  不想显示可以选择关闭
#prev: false
next: false
#页脚
footer: false

---
# Spring
## 01 对Spring中IOC的理解

**IOC即“控制反转”，不是什么技术，而是一种设计思想。IOC意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制**

**谁控制谁，控制什么：**传统Java程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IOC是有专门一个容器来创建这些对象，即由IOC容器来控制对象的创建

**为何是反转，哪些方面反转了：**有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；**因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转**

## 02 对Spring中AOP的理解

AOP就是我们常说的面向切面编程。使用AOP可以将系统中的一些公共模块抽取出来，减少公共模块和业务代码的耦合。利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各 部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率

像Spring中的事物控制使用的就是AOP机制实现的 , 同时也可以使用AOP做一些权限控制 , 日志记录 , 接口统一缓存等操作

Spring中的AOP底层主要是通过动态代理机制实现的 , 主要使用的是**JDK动态代理**和**CGLIB动态代理**

## 03 Spring中AOP的原理

在 Spring 中 AOP 代理使用 JDK 动态代理和 CGLIB 代理来实现，默认如果目标对象是接口，则使用 JDK 动态代理，否则使用 CGLIB 来生成代理类

**JDK 动态代理是基于接口的代理 ,** 产生的代理对象和目标对象实现了相同的接口 , 创建代理对象的方法

```
Object proxy = Proxy.newProxyInstance(类加载器,目标对象接口, 代理执行器InvocationHandler)
```

**CGLIB动态代理是基于父类的代理 ,** 产生的代理对象是目标对象的子类 , 创建代理对象的方法

```
Object proxy = Enhancer.create(目标对象类型 , 代理回调callback)
```

## 04 Spring中在什么情况下事物会失效

Spring中事物失效的场景很多 , 例如 :

1. 因为Spring事务是基于代理来实现的，所以某个加了@Transactional的⽅法只有是被代理对象调⽤时， 那么这个注解才会⽣效 ! 如果使用原始对象事物会失效
2. 同时如果某个⽅法是private的，那么@Transactional也会失效，因为底层cglib是基于⽗⼦类来实现 的，⼦类是不能重载⽗类的private⽅法的，所以⽆法很好的利⽤代理，也会导致@Transactianal失效
3. 如果在业务中对异常进行了捕获处理 , 出现异常后Spring框架无法感知到异常, @Transactional也会失效
4. @Transational中默认捕获的是RuntimeException , 如果没有指定捕获的异常类型, 并且程序抛出的是非运行时异常, 事物会失效

## 05 Spring的事务传播行为

1. PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置
2. PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务
3. PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
4. PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
5. PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常
6. PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常
7. PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行

## 06 SpringMVC执行流程

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678757871421-eaf46719-9b19-4ced-9165-8dfe1330440f.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_21%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

1. 用户发送请求至前端控制器DispatcherServlet
2. DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handle
3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器 , 组成一个处理器链 , 一并返回给DispatcherServlet
4. DispatcherServlet 调用 HandlerAdapter处理器适配器；
5. HandlerAdapter 经过适配调用具体处理器(Handler，也叫后端控制器)
6. Handler执行完成返回ModelAndView
7. HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析
9. ViewResolver解析后返回具体View
10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）
11. DispatcherServlet响应用户

## 07 Spring MVC常用的注解

1. @RequestMapping：用于处理请求 url 映射的注解，可用于类或方法上
2. @RequestBody：注解实现接收http请求的json数据，将json转换为java对象
3. @ResponseBody：注解实现将conreoller方法返回对象转化为json对象响应给客户
4. @Controller：控制器的注解，表示是表现层,不能用用别的注解代替
5. @RestController : 组合注解 @Conntroller + @ResponseBody
6. @PathVariable : 接收请求路径中的参数
7. @RequestParam : 接收请求参数
8. @RequestHeader : 接收请求头数据
9. @ExceptionHandler : 用于全局异常处理器, 指定补货具体类型的异常
10. @ControllerAdvice: 主要用来处理全局数据，一般搭配@ExceptionHandler使用 , 进行异常处理
11. @RestControllerAdvice : 组合注解@ControllerAdvice + @ResponseBody
12. @CookieValue : 获取请求中指定名称的cookie值
13. @SessionAttribute : 获取带session中属性值
14. @CrossOrigin : 设置CROS跨域访问

## 08 #{}和${}区别

- mybatis在处理`#{}`时，底层会将SQL中的`#{}`替换为`?`号，调用 PreparedStatement进行预编译处理 , 然后调用set方法来赋值 , 可以有效的防止SQL注入
- mybatis在处理`${}` , 底层采用的是字符串拼接的方式, 将参数直接拼接到SQL语句中 , 会有SQL 注入风险

## 09 SpringBoot自动装配的原理

在SpringBoot项目的启动引导类上都有一个注解@SpringBootApplication

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678786945657-23ed4bcc-8eb5-4f81-ad90-f70f83c49708.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

这个注解是一个复合注解, 其中有三个注解构成 , 分别是

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678786970759-4e76badc-4045-4676-96c4-d90b450fb27d.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_33%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

- @SpringBootConfiguration : 是@Configuration的派生注解 , 标注当前类是一个SpringBoot的配置类
- @ComponentScan : 开启组件扫描, 默认扫描的是当前启动引导了所在包以及子包
- **@EnableAutoConfiguration : 开启自动配置(自动配置核心注解)**

在@EnableAutoConfiguration注解的内容使用@Import注解导入了一个AutoConfigurationImportSelector.class的类

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678786999620-cfc8f8bf-493c-482d-834a-04a06caa69fa.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_15%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

在AutoConfigurationImportSelector.class中的selectImports方法内通过一系列的方法调用, 最终需要加载类加载路径下META-INF下面的spring.factories配置文件

在META-INF/spring.factories配置文件中, 定义了很多的自动配置类的完全限定路径

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678787025975-026cbe8f-d634-4ee8-a0dc-3fc62a5f5eb5.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_30%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

这些配置类都会被加载

加载配置类之后, 会配置类或者配置方法上的@ConditionalOnXxxx条件化注解是否满足条件

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678787045279-5d2de03b-f60a-4b63-9552-d87fe5b3244e.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_22%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

如果满足条件就会使用`@ConfigurationProperties`注解从属性配置类中读取相关配置 , 执行配置类中的配置方法 , 完成自动配置

![](https://cdn.nlark.com/yuque/0/2023/png/22479650/1678787091252-ae7ff83e-4059-486e-b0f6-e308af7ad758.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_23%2Ctext_ZHd4d2FuZw%3D%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

  

## 10 Spring Cloud 5大组件有哪些

Spring Cloud五大组件主要是

1. 注册中心组件 , 例如 : Eureka 和 Nacos
2. 负载均衡组件 , 例如 : Ribbon 和 Spring Cloud LoadBalancer
3. 远程调用组件 , 例如 : Feign 和 Dubbo
4. 服务熔断组件 , 例如 : Hystrix 和 Sentinel
5. 服务网关组件 , 例如 : Zuul 和 Spring Cloud Gateway

## 11 项目中微服务之间是如何通讯

服务的通讯方式主要有二种 :

1．同步通信：通过Feign发送http请求调用 或者 通过Dubbo发送RPC请求调用

2．异步通信：使用消息队列进行服务调用，如RabbitMQ、KafKa等
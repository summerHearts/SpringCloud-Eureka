 - 服务的注册与发现（服务的生产者和消费者）。这里主要使用的是 Eureka 。
  -  Eureka是Netflix开发的服务发现框架，SpringCloud将它集成在自己的子项目spring-cloud-netflix中，实现SpringCloud的服务发现功能。 
  -  为什么要使用Eureka，因为在一个完整的系统架构中，任何单点的服务都不能保证不会中断，因此我们需要服务发现机制，在某个节点中断后，其它的节点能够继续提供服务，从而保证整个系统是高可用的。 
  -  服务发现有两种模式：一种是客户端发现模式，一种是服务端发现模式。Erueka采用的是客户端发现模式。

![](http://upload-images.jianshu.io/upload_images/325120-ed458fb803e1c74d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- Eureka Server会提供服务注册服务，各个服务节点启动后，会在Eureka Server中进行注册，这样Eureka Server中就有了所有服务节点的信息，并且Eureka有监控页面，可以在页面中直观的看到所有注册的服务的情况。同时Eureka有心跳机制，当某个节点服务在规定时间内没有发送心跳信号时，Eureka会从服务注册表中把这个服务节点移除。Eureka还提供了客户端缓存的机制，即使所有的Eureka Server都挂掉，客户端仍可以利用缓存中的信息调用服务节点的服务。Eureka一般配合Ribbon进行使用，Ribbon提供了客户端负载均衡的功能，Ribbon利用从Eureka中读取到的服务信息，在调用服务节点提供的服务时，会合理的进行负载。 
- Eureka通过心跳检测、健康检查、客户端缓存等机制，保证了系统具有高可用和灵活性。

- eureka- server    注册服务 application.yml 文件

![](http://upload-images.jianshu.io/upload_images/325120-fe413cd53a24edb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](http://upload-images.jianshu.io/upload_images/325120-c9dd2b562d5398e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- eureka-consumer 消费服务 application.yml 文件

![](http://upload-images.jianshu.io/upload_images/325120-af3bc01b9da8dc5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![](http://upload-images.jianshu.io/upload_images/325120-af752fb9da6ae572.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 启动两个项目， 访问 http://localhost:8761  ，进入如下界面

![](http://upload-images.jianshu.io/upload_images/325120-8aa124ea682302e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 访问 http://localhost:8762/hi?name=AKyS

![](http://upload-images.jianshu.io/upload_images/325120-6bc95a47f4dcbf1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


##2、服务消费者

  - 在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于http restful的。
这里我们使用ribbon ，实现服务的转发功能，从而实现负载均衡。

  - 在Spring Cloud框架中，负载均衡服务本身也要作为一个发现客户端注册到Eureka服务器上。客户发起一个请求时，需要在Eureke服务器上发现负载均衡服务，负载均衡服务通过RestTemplate调用微服务的接口时，会通过Ribbon进行负载均衡。这样，不同的服务请求会由负载均衡机制分别调用微服务的不同实例。

![](http://upload-images.jianshu.io/upload_images/325120-301c666d55157fbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

 - ribbon 是一个客户端负载均衡器，可以简单的理解成类似于 nginx的负载均衡模块的功能。ribbon是一个为客户端提供负载均衡功能的服务，它内部提供了一个叫做ILoadBalance的接口代表负载均衡器的操作，比如有添加服务器操作、选择服务器操作、获取所有的服务器列表、获取可用的服务器列表等等。

- Ribbon的工作原理
     分为两步： 
    - 1、 第一步有限选择Eureka Server，它优先选择在同一个Zone且负载较少的Server， 
    - 2、 第二步在根据用户指定的策略，在从Server取到的服务注册列表中选择一个地址。其中Ribbon提供了多重策略，例如轮询round robin、随机Random、根据相应时间加权等。

## 项目中如何配置
   - 新建项目Server-Ribbon项目

   - application.yml  配置信息

     ![](http://upload-images.jianshu.io/upload_images/325120-cb14556a1e3af8a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

    - pom文件配置

  ![](http://upload-images.jianshu.io/upload_images/325120-7bd1b2f52dc153e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  -   启动类注入

  ![](http://upload-images.jianshu.io/upload_images/325120-573095172f225e40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

   - 首先先使用ribbon提供的LoadBalanced注解加在RestTemplate上面，这个注解会自动构造LoadBalancerClient接口的实现类并注册到Spring容器中。

   - 接下来使用RestTemplate进行rest操作的时候，会自动使用负载均衡策略，它内部会在RestTemplate中加入LoadBalancerInterceptor这个拦截器，这个拦截器的作用就是使用负载均衡。

![](http://upload-images.jianshu.io/upload_images/325120-a87b8adeb763b66b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

在浏览器上多次访问http://localhost:8764/hi?name=AKyS，浏览器交替显示：

```
hi AKyS,i am from port:8762

hi AKyS,i am from port:8763
```

-  当sercvice-ribbon通过restTemplate调用service-consumer的hi接口时，因为用ribbon进行了负载均衡，会轮流的调用service-consumer：8762和8763 两个端口的hi接口。这样就能实现交替服务的提供。


- 1、Hystrix简介 

  ![](http://upload-images.jianshu.io/upload_images/325120-b13d814ddba53a56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  [Hystrix](https://github.com/Netflix/Hystrix)是Netflix针对微服务分布式系统的熔断保护中间件，当我们的客户端连接远程的微服务时，有两种情况需要考虑：首先，如果远程系统当机了我们怎么办？其次，我们如何管理对远程微服务的调用性能，以保证每个微服务以最小延迟最快性能响应？
 
  Hystrix是一个有关延迟和失败容错的开源库包，用来设计隔离访问远程系统端点或微服务等，防止级联爆炸式的失败，也就是由一个小问题引起接二连三扩大的疯狂的错误爆炸直至整个系统瘫痪，能够让复杂的分布式系统更加灵活具有弹性。

-  2、 如何在项目中使用断路由

    启动`eureka-consumer`和 `eureka-server`服务。

    改造`serice-ribbon` 工程的代码，首先在pox.xml文件中加入`spring-cloud-starter-hystrix`的起步依赖：

    ![](http://upload-images.jianshu.io/upload_images/325120-eab2d776897e2f67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

在程序的启动类`ServiceRibbonApplication `加`@EnableHystrix`注解开启`Hystrix`

   ![](http://upload-images.jianshu.io/upload_images/325120-e0b8b819d4674c72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

 ` @HystrixCommand`注解。该注解对该方法创建了熔断器的功能，并指定了`fallbackMethod`熔断方法，熔断方法直接返回了一个字符串，字符串为”hi,”+name+”,sorry,error!”

   ![](http://upload-images.jianshu.io/upload_images/325120-8a9d8ec7669b68e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

   启动：`service-ribbon` 工程，当我们访问http://localhost:8764/hi?name=AKyS,浏览器显示：

```
hi AKyS,i am from port:8762
```
  当关闭 `eureka-consumer`服务，浏览器显示：

 ```
hi , AKyS,orry,error!
```

很早之前看到过一句话： 

![](http://upload-images.jianshu.io/upload_images/325120-88cb6aad083e4c85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)



## 1、路由网关简介
   - 服务网关是微服务架构中一个不可或缺的部分。通过服务网关统一向外系统提供REST API的过程中，除了具备服务路由、均衡负载功能之外，它还具备了权限控制等功能。Spring Cloud Netflix中的Zuul就担任了这样的一个角色，为微服务架构提供了前门保护的作用，同时将权限控制这些较重的非业务逻辑内容迁移到服务路由层面，使得服务集群主体能够具备更高的可复用性和可测试性。

  - 路由在微服务体系结构的一个组成部分。例如，/可以映射到您的Web应用程序，/api/users映射到用户服务，并将/api/shop映射到商店服务。Zuul是Netflix的基于JVM的路由器和服务器端负载均衡器。

  - Netflix使用Zuul进行以下操作：
      - 认证
      - 洞察
      - 压力测试
      - 金丝雀测试
      - 动态路由
      - 服务迁移
      - 负载脱落
      - 安全
      - 静态响应处理
      - 主动/主动流量管理
      - Zuul的规则引擎允许基本上写任何JVM语言编写规则和过滤器，内置Java和Groovy。

##2、什么是服务网关
   服务网关 = 路由转发 + 过滤器
   - 1、路由转发：接收一切外界请求，转发到后端的微服务上去。

   - 2、过滤器：在服务网关中可以完成一系列的横切功能，例如权限校验、限流以及监控等，这些都可以通过过滤器完成（其实路由转发也是通过过滤器实现的）。

##3、为什么需要服务网关

- 上述所说的横切功能（以权限校验为例）可以写在三个位置：

 - 每个服务自己实现一遍
           第一种，缺点太明显，基本不用；
 - 写到一个公共的服务中，然后其他所有服务都依赖这个服务
     - 第二种，相较于第一点好很多，代码开发不会冗余，但是有两个缺点：
          - 由于每个服务引入了这个公共服务，那么相当于在每个服务中都引入了相同的权限校验的代码，使得每个服务的jar包大小无故增加了一些，尤其是对于使用docker镜像进行部署的场景，jar越小越好；
由于每个服务都引入了这个公共服务，那么我们后续升级这个服务可能就比较困难，而且公共服务的功能越多，升级就越难，而且假设我们改变了公共服务中的权限校验的方式，想让所有的服务都去使用新的权限校验方式，我们就需要将之前所有的服务都重新引包，编译部署。
 - 写到服务网关的前置过滤器中，所有请求过来进行权限校验
      - 将权限校验的逻辑写在网关的过滤器中，后端服务不需要关注权限校验的代码，所以服务的jar包中也不会引入权限校验的逻辑，不会增加jar包大小；
      - 如果想修改权限校验的逻辑，只需要修改网关中的权限校验过滤器即可，而不需要升级所有已存在的微服务。

##4、服务网关技术选型

![](http://upload-images.jianshu.io/upload_images/325120-b897cf7641024e3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

引入服务网关后的微服务架构如上，总体包含三部分：服务网关、open-service和service。

 - 1、总体流程：

    - 服务网关、open-service和service启动时注册到注册中心上去；
    - 用户请求时直接请求网关，网关做智能路由转发（包括服务发现，负载均衡）到open-service，这其中包含权限校验、监控、限流等操作
open-service聚合内部service响应，返回给网关，网关再返回给用户
 - 2、引入网关的注意点

      - 增加了网关，多了一层转发（原本用户请求直接访问open-service即可），性能会下降一些（但是下降不大，通常，网关机器性能会很好，而且网关与open-service的访问通常是内网访问，速度很快）；
      - 网关的单点问题：在整个网络调用过程中，一定会有一个单点，可能是网关、nginx、dns服务器等。防止网关单点，可以在网关层前边再挂一台nginx，nginx的性能极高，基本不会挂，这样之后，网关服务就可以不断的添加机器。但是这样一个请求就转发了两次，所以最好的方式是网关单点服务部署在一台牛逼的机器上（通过压测来估算机器的配置），而且nginx与zuul的性能比较，根据国外的一个哥们儿做的实验来看，其实相差不大，zuul是netflix开源的一个用来做网关的开源框架；
      - 网关要尽量轻。
 - 3、服务网关基本功能

      - 智能路由：接收外部一切请求，并转发到后端的对外服务open-service上去；
      - 注意：我们只转发外部请求，服务之间的请求不走网关，这就表示全链路追踪、内部服务API监控、内部服务之间调用的容错、智能路由不能在网关完成；当然，也可以将所有的服务调用都走网关，那么几乎所有的功能都可以集成到网关中，但是这样的话，网关的压力会很大，不堪重负。
      - 权限校验：只校验用户向open-service服务的请求，不校验服务内部的请求。服务内部的请求有必要校验吗？
      - API监控：只监控经过网关的请求，以及网关本身的一些性能指标（例如，gc等）；
      - 限流：与监控配合，进行限流操作；
      - API日志统一收集：类似于一个aspect切面，记录接口的进入和出去时的相关日志

## 5、项目集成

   -   创建service-zuui  项目 ，其pom.xml文件内容是
   
   ![](http://upload-images.jianshu.io/upload_images/325120-3d340e12046fbbc4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

  -  在applicaton类加上注解@EnableZuulProxy，开启zuul的功能：

   ![](http://upload-images.jianshu.io/upload_images/325120-66be41fadf9be2cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 配置文件application.yml

 ![](http://upload-images.jianshu.io/upload_images/325120-60c9ef2161da7493.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

- 首先指定服务注册中心的地址为[http://localhost:8761/eureka/](http://localhost:8761/eureka/)，服务的端口为8769，服务名为service-zuul；以/api-a/ 开头的请求都转发给service-ribbon服务；以/api-b/开头的请求都转发给service-ribbon服务；

依次运行这五个工程;打开浏览器访问：[http://localhost:8769/api-a/hi?name=AKyS](http://localhost:8769/api-a/hi?name=AKyS) ;浏览器显示：

> hi AKyS,i am from port:8762

打开浏览器访问：[http://localhost:8769/api-b/hi?name=AKyS](http://localhost:8769/api-b/hi?name=AKyS) ;浏览器显示：

> hi AKyS,i am from port:8762


##6、服务过滤

   - zuul不仅只是路由，并且还能过滤，做一些安全验证。继续改造工程；

  ![](http://upload-images.jianshu.io/upload_images/325120-020d6e7432eec040.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

*   filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下： 

    *   pre：路由之前
    *   routing：路由之时
    *   post： 路由之后
    *   error：发送错误调用
    *   filterOrder：过滤的顺序
    *   shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
    *   run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。

这时访问：[http://localhost:8769/api-a/hi?name=AKyS](http://localhost:8769/api-a/hi?name=AKyS) ；网页显示：

> token is empty

访问 [http://localhost:8769/api-a/hi?name=AKyS&token=22](http://localhost:8769/api-a/hi?name=AKyS&token=22) ； 
网页显示：

> hi AKyS,i am from port:8762

注意事项：

   本文参考 博客文章地址   [[史上最简单的SpringCloud教程 | 第五篇: 路由网关(zuul)](http://blog.csdn.net/forezp/article/details/69939114)
](http://blog.csdn.net/forezp/article/details/69939114)




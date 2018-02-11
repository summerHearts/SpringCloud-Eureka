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






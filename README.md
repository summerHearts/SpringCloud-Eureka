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

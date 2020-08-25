---
layout: mypost
title: dubbo
categories: [java]
---

#### Dubbo架构，启动过程
```
1. 服务容器启动，加载运行Provider。
2. Provider向注册中心注册自己的服务。
3. Consumer启动，向注册中心订阅所需服务。
4. 注册中心返回服务地址列表给Consumer，如有变更注册中心主动基于长连接推送新的服务列表给Consumer。
5. Consumer从Provider列表中基于负载均衡选一台服务器调用。
6. Provider和Consumer记录调用次数和调用时间定时发送给监控中心。
```
#### Java spi 和 dubbo spi
```
1. Java Spi：是一种服务发现机制，本质是将接口实现的全限定名配置在文件(META-INF/services/接口全限定名)中，并由服务加载器(ServiceLoader)加载配置文件，从而实现加载实现类的功能。
2. Dubbo Spi：并未使用javaSPI，而是实现了一套功能更强的SPI，配置文件放在META-INF/dubbo目录下，以键值对的形式进行配置，可以通过ExtensionLoader加载指定的实现类。
```
#### Dubbo标签
```
provider:
  1.register。
  2.protocol。
  3.service（id：service id。interface：接口名。ref：接口实现的的bean引用。version：版本号，接口不兼容时再升级。timeout：超时时间。retries：远程服务调用次数。connections：最大连接数。loadbalance：负载均衡策略。cluster：集群方式。register：该服务是否注册到注册中心。）
Consumer：
  1.reference（id：引用服务的Bean id。interface：服务接口名。version：版本号，与服务版本号相同。check：启动时检查提供者是否存在。url：点对点直连的地址。）  
```
#### Dubbo协议
```
通信协议：
  1. dubbo协议: 缺省协议采用单一长连接和NIO异步通讯，Hessian的二进制序列化，适合小数据量大并发量的服务调用，以及消费者机器远大于提供者机器的情况，不适合传送大数据量的服务。
  2. rmi协议: 采用阻塞式短连接和JDK标准序列化方式，适用于传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
  3. hessian: 底层是Http，同步传输，Hessian二进制序列化，适用于传入传出参数数据包较大，提供者比消费者个数多，可传文件。
  4. http: 传入传出参数数据包大小混合，提供者比消费者多，可用表单或URL传入参数，不支持传文件。
  5. webservice
序列化协议：
  1. hessian2序列化: 默认序列化协议.
  2. json序列化
  3. java序列化
  4. Kryo
  5. FST
```
#### 集群容错模式cluster
```
1. Failover: 失败自动切换其他服务器。
2. Failfast：失败立即报错。
3. Failsafe：出现异常直接忽略。
4. Failback：失败则记录，定时重发。
5. ForKing：并行调用，成功一个即可。
6. Broadcast：调用所有服务器，任意一个报错即报错。
```
#### 负载均衡策略loadbalance
```
1. Random：按权重随机。
2. RoundRobin：轮循。
3. LeastActive：慢提供者收到较少请求。
4. ConsistentHash：相同参数的请求总是发到同一提供者。
```
#### 为什么Spring注入远程的Dubbo Bean像注入本地普通Bean一样？
####
```
1. 所有的Dubbo消费者Bean都是ReferenceBean类型的对象，interface标签中配置的接口只是让ReferenceBean知道Dubbo的服务提供方提供的方法签名而已。
2. 在java体系中动态创建对象一定用到了动态代理。
3. 在Spring容器中会利用BeanDefinition对象信息来初始化创建之后的bean对象，但如果你想插手"创建对象"这一步，有两种方法：1.使用工厂方法模式来指定某个工厂方法创建Bean。 2. Bean类实现FactoryBean接口。Dubbo就是用第二种方法来创建bean的，所以显然ReferenceBean实现了FactoryBean，而getObject()正是FactoryBean里定义的。
4. Dubbo Spi。
```
#### 注册中心Zookeeper
```
1. /dubbo/Service/目录下有四个目录
2. Providers目录下存放提供者的URL和元数据。
3. Consumers目录下存放着消费者的URL和元数据。
4. Routers目录下存放消费者的路由策略。
5. Configurators目录下存放多个用于服务提供者动态配置URL元数据信息。
```
#### zookeeper的四种类型的节点
```
1. 持久节点。
2. 临时节点。
3. 持久顺序节点。
4. 临时顺序节点。
```

#### zookeeper的典型的应用场景
```
1. 注册中心。
2. 分布式锁（第一种锁独占：我们把zk上的一个znode看做一把锁，通过createznode的方式来实现。所有线程都去创建/distribute_lock节点，最终成功创建的那个线程也即获得了这把锁，用完删除即可。第二种锁控制时序：/distribute_lock已经预先存在，所有线程都在它下面创建临时顺序节点，编号最小的获得锁）。
3. 负载均衡。
4. Master选举。
5. 分布式队列。
6. 数据发布/订阅。
7. 命名服务。
8. 分布式协调/通知。
9. 集群管理。
```

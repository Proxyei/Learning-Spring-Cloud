# Spring Cloud学习笔记

# 1、学习目标

## Spring Cloud能做什么？怎么做？实现原理是什么？

## 微服务技术栈

## 假如注册中心挂了，然后重新加入？

## 假如provider挂了，然后重新加入？

# 2、微服务概述

1. 微服务是什么？

   - 目前，微服务没有统一、标准的定义。
   - 微服务架构是一种架构模式或者架构风格。
   - **微服务将单一应用拆分为一组小的服务，每个服务作为单独进程运行，都可以有单独的数据库（数据不一致怎么办？），各个服务使用通信机制相互通信（像rpc-远程过程调用，restful）**。
   - 一般根据业务来拆分，一个服务只做一件事情。

2. 微服务优缺点

   1. 优点
      - 开发效率高
      - 松耦合
      - 可以混合语言开发
      - 独立
   2. 缺点
      - 开发人员要处理分布式系统的复杂性
      - 运维人员部署复杂
      - 服务之间通信成本
      - 数据一致性的保证

3. 微服务技术栈

   ![](E:\IT\Spring Cloud\微服务技术栈1.jpg)

   ![](E:\IT\Spring Cloud\微服务技术栈2.jpg)

4. 各个微服务框架对比

   ![](E:\IT\Spring Cloud\微服务框架对比.jpg)

5. 比较主流的Spring Cloud与Dubbo对比

   ![](E:\IT\Spring Cloud\dubbo-springcloud对比.jpg)

# 3、Spring Cloud入门

## 1、RestTemplate整合consumer-provider小应用



## 2、Eureka

### 1、作用

主管服务注册与发现，类似dubbo使用的Zookeeper。

### 2、架构

1. 采用C/S架构。
2. 系统其他微服务通过Eureka Client注册到Eureka并维持心跳连接，使得维护人员可以通过Eureka Server监控微服务的运行状态。
3. 其他模块可以通过Eureka Server发现系统其他微服务，进而执行相关逻辑代码。

### 3、与Zookeeper对比

![](E:\IT\Spring Cloud\Eureka架构.jpg)

![](E:\IT\Spring Cloud\Zookeeper架构.jpg)

### 4、使用步骤

#### 1、建立Eureka注册中心

- 导包，修改application.yml配置文件，添加@EnableEurekaServer支持。

#### 2、修改provider

- 导包，修改application.yml配置文件，添加@EnableEurekaServer支持。

- 修改默认服务名字：eureka.instance.id=服务名

  ![](E:\IT\Spring Cloud\修改服务默认名字.jpg)

- 显示服务IP地址：eureka.instance.prefer-ip-address=true

- info信息显示:

  导包spring-boot-starter-actuator

  key-value配置：

  ![](E:\IT\Spring Cloud\info信息配置.jpg)

- Eureka自我保护机制

  某时刻，某个服务无法用了，Eureka不会立刻清理，依旧会保留该服务名。

  - 原因1：可能服务挂了或者网络拥堵。

  - 实际原因：

    ![](E:\IT\Spring Cloud\Eureka自我保护机制1.jpg)

    ![](E:\IT\Spring Cloud\Eureka自我保护机制2.jpg)

  - **不推荐禁用自我保护机制**

### 5、服务发现

不访问Eureka中心页面，使用URL方式获取服务信息。

使用步骤：

1. provider里面配置discoveryClient。
2. consumer访问

### 6、集群配置

步骤：

1. 配置多个Eureka服务注册中心，服务中心相互配置defaultzone
2. provider的service-url配置所有的注册中心defaultzone

### 7、eureka与zookeeper区别

CAP原则

C：强一致性

A：可用性

P：分区容忍性

因为网络延迟丢包的问题，所以，P必须保证有。

zookeeper保证CP：

- zookeeper的master挂了之后，slave之间会选举出新的master，在此期间，服务不可用。

eureka保证AP：

- 自我保护机制
- 服务可用了，网络稳定了就同步到其他节点

## 3、Ribbon（客户端、负载均衡工具）

### 1、ribbon是什么

![](E:\IT\Spring Cloud\ribbon定义.jpg)

### 2、负载均衡概述

![](E:\IT\Spring Cloud\LB分类-偏硬件-集中LB.jpg)

![](E:\IT\Spring Cloud\LB分类-偏软件-进程内LB.jpg)

### 3、ribbon使用

1. 在客户端也就是consumer端导包：eureka+ribbon（需要合eureka整合，类似Java访问数据库需要MySQL JDBC包)
2. consumer端application.yml配置eureka，启动类配置eurekaclient，resttemplate。
3. 使用服务名代替ip+port访问。

### 4、ribbon负载均衡算法

其实就是访问provider端资源的规则。

![](E:\IT\Spring Cloud\ribbon负载均衡访问算法.jpg)

1. 轮询
2. 随机
3. 重试（轮询到一个错误的，重试几次之后就废弃该错误的provider）

使用：直接配置一个bean

### 5、自定义负载均衡算法

使用步骤：

1. 在客户端启动类添加@ribbonclient(name="微服务名字"，configuration="自定义规则类")注解，
2. 编写自定义负载均衡算法类，需要配置@configuration，@bean。注意：该类不能在SpringbootApplication标注类的包下或者子包下！
3. 如果工程配置类与负载均衡配置类同时存在irule，则负载均衡类优先被使用。

## 4、feign负载均衡（整合了ribbon）

本质就是接管ribbon，类似spring接管mybatis，hibernate数据源配置。

先说问题：如何实现ribbon的负载均衡？（直接像单独使用ribbon那样）

使用步骤

1. API工程导包feign包。
2. API工程建立service接口，接口内容与provider的controller一致，使用@feignclient注解标识服务ID。
3. 新建feign工程，参考consumer-dept-80工程。
4. 建立controller，套路回到老套路。
5. 启动类配置上扫描API工程service层的包名。

## 5、hystrix断路器

## 疑问，如果服务熔断与服务降级同时配置了，会如何？

答案：会先通过客户端降级，二者可以二选一。

### 1、服务熔断（provider服务端完成）

需要导包

### 2、服务降级（consumer客户端完成）

在API模块中完成接口代码

consumer中的yml开启配置

### 3、服务监控（针对监控provider类）

- 需要actuator包
- provider端启动类有@EnableCircuitBreaker注解
- controller上需要有@HystrixCommand.

一圈一线七色



## 6、zuul路由网关

疑问：如果设置了，那consumer，api模块如何访问？还怎么做服务降级？负载均衡？这些还有什么用？consumer还有什么用？熔断自身服务端还可以做。

## 7、SpringCloud   config

集中式协调各个微服务

### Spring Clud配置文件管理中心

想象中的效果是config-server监控每个工程的配置文件，一旦远程仓库的配置文件发生变化，就会更新每个服务本地的配置文件。

### 使用步骤：

准备工作：

1. 本地安装了git并且推送了公钥到远程git仓库。
2. 远程git仓库建立工程仓库：spring-cloud-config-server，然后clone下来，以后的配置文件就放置在上面。

### 1、建立配置文件服务管理中心模块config-server

#### 1、导包，spring-cloud-starter-config

#### 2、建立application.yml文件，配置远程git仓库。

```yaml
# 不是必须的
server: 
  port: 6666 #谷歌浏览器屏蔽了6666端口，应该换成其他的
# 集合到eureka，配置远程git仓库地址  
spring:
  application:
    name:  microservicecloud-config
  cloud:
    config:
      server:
        git:
          uri: git@XXX/spring-cloud-config-serve.git #GitHub上面的git仓库名字

```

### 2、整合config客户端

在需要动态切换配置文件的微服务工程下进行配置，以privider-dept-8001为例

建立bootstrap.yml文件，配置spring-cloud-config-server服务的访问地址可以是

- http://ip:port
- http://hostname:port

配置需要加载的远程配置文件名，不包括后缀。

如下：

```yaml
spring:
  cloud:
    config:
      name: xxx #需要从github上读取的资源名称，注意没有yml后缀名
      #profile配置是什么就取什么配置dev or test
      profile: dev
      #profile: test
      label: master
      uri: http://confi-server:6666  #SpringCloudConfig获取的服务地址
```

上传privider-dept-8001工程的配置文件，privider-dept-8001.yml

```yaml
spring:
  profiles:
    active:
    - dev
--- 
server:
  port: 8001
  
## mybatis配置
mybatis:
  config-location: classpath:mybatis/config/mybatis.cfg.xml
  type-aliases-package: com.xywei.entity
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml 
  

## 服务名/数据库/数据源配置
spring: 
  profiles: dev
  application:
    name: provider-dept-service #eurakla中对外暴露的微服务名字， 与server.context-path有什么区别
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/xy-spring-cloud-first-8001-dev
    username: root
    password: xieSHI123321
    dbcp2:
      initial-size: 5 # 初始化连接数
      min-idle: 5 # 维持最小连接数
      max-total: 5 # 最大连接数
      max-wait-millis: 1000 # 等待时间

eureka:
  client:
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: provider-dept-8001
    prefer-ip-address: true
    
info:
  app.name: $project.build.finalName$
  build.version: $project.version$
--- 
server:
  port: 8001
  
## mybatis配置
mybatis:
  config-location: classpath:mybatis/config/mybatis.cfg.xml
  type-aliases-package: com.xywei.entity
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml 
  

## 服务名/数据库/数据源配置
spring: 
  profiles: test
  application:
    name: provider-dept-service #eurakla中对外暴露的微服务名字， 与server.context-path有什么区别
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/xy-spring-cloud-first-8001-test
    username: root
    password: xieSHI123321
    dbcp2:
      initial-size: 5 # 初始化连接数
      min-idle: 5 # 维持最小连接数
      max-total: 5 # 最大连接数
      max-wait-millis: 1000 # 等待时间

eureka:
  client:
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: provider-dept-8001
    prefer-ip-address: true
    
info:
  app.name: $project.build.finalName$
  build.version: $project.version$
```



### 总结：

- 客户端bootstrap.yml必须配置上profiles:属性，否则会报错
- 微服务启动的时候会优先找github上的，也就是github的配置会覆盖本地配置。
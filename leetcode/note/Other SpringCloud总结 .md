---
title: SpringCloud总结
date: 2023-11-23 20:26:09
tags: [SpringCloud, 八股文]
---

# SpringCloud 总结

分布式架构相比于单体架构具有低耦合、业务拆分的特点，但也引入了一些问题比如数据一致性问题、复杂调用关系等等。微服务是一种分布式架构标准，微服务强调拆分粒度小、每个微服务独立程度高、每个微服务向外提供标准接口。

微服务拆分原则需要遵守：

- 不同微服务，不要重复开发相同业务
- 微服务数据独立，不要访问其它微服务的数据库

SpringCloud 是目前国内使用最广泛的微服务框架。官网地址：https://spring.io/projects/spring-cloud。SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配。SpringCloud和SpringBoot之间存在版本兼容关系。

SpringCloud 中的各种微服务之间采用 http 协议进行接口调用。RestTemplate 组件提供了 http 借口调用的实现。

## 微服务注册中心

微服务注册中心支持服务提供者注册信息，支持服务消费者获取服务提供者的注册信息。相比于服务提供者的接口地址写死在服务消费者代码中的方式，微服务注册中心一方面支持随时的服务提供者变动，另一方面也支持微服务调用的负载均衡。

### Eureka

Eureka 支持服务提供者注册服务信息，支持服务消费者定时拉取服务信息。服务提供者通过心跳的方式（默认 30s）告知 Eureka 自己的健康状态。

我们可以直接在 SpringBoot 中启动一个 Eureka 微服务（通过引入相关依赖、配置 yml 文件、配置注解），然后将其他微服务作为服务提供者注册到 Eureka（通过引入相关依赖、配置 yml 文件），最后作为服务消费者的任何微服务都可以拉取 Eureka 中的服务注册信息进行相应的微服务接口调用（通过引入相关依赖、配置 yml 文件、配置注解）。

微服务接口调用的负载均衡是通过 Ribbon 实现的，Ribbon 默认是采用懒加载。负载均衡接口 IRule 有一些默认的接口实现比如轮询（RoundRobin）、权重、区域优先等等。

### Nacos

Nacos 是阿里的产品，相比 Eureka 功能更丰富。Nacos 直接提供了操作系统可以直接运行的 bin 文件（依赖 jre）。

#### 服务注册与拉取

Nacos 将一个服务分为了多个集群（通常一个集群指的是某区域的一个机房），每个集群又分为了多个实例。微服务互相访问时，应该尽可能访问同集群实例，因为本地访问速度更快。当本集群内不可用时，才访问其它集群。

我们同样可以通过引入相关依赖、配置 yml 文件、操作 Nacos 的 web 控制台的方式，完成服务提供者的集群注册、负载权重配置、服务消费者拉取信息等操作。

Nacos 注册中心对于服务提供者除了心跳检测还会主动询问，除了服务消费者会定时拉取 Nacos 注册中心信息外，Nacos 还会向服务消费者主动推送变更消息。

#### 配置管理与热更新

我们可以在 Nacos 的 web 控制台上创建 yaml 配置，并将需要热更新的配置放在其中，这样微服务的配置信息就由 bootstrap.yaml、Nacos 在线 yaml 和 application.yml 共同组成，我们通过@Value 注解或@ConfigurationProperties 注解就可以读取热更新配置。当然上述过程还是需要通过引入相关依赖、配置 yml 文件、配置注解来完成。

Nacos 在线 yaml 配置有两种：

- `[spring.application.name]-[spring.profiles.active].yaml`，例如：userservice-dev.yaml

- `[spring.application.name].yaml`，例如：userservice.yaml

`[spring.application.name].yaml`不包含环境，因此可以被多个环境共享。

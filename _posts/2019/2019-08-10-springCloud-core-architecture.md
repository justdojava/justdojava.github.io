---
layout: post
categories: SpringCloud
title: Spring Cloud 的核心架构原理
tagline: by 乔二爷
tags:
    - 乔二爷
---

最近在补一些分布式系列的面试内容，提前几个月做做准备吧，你们懂的，也跟大家分享分享。现在分布式系统基本上都是标配了，如果你现在还在玩儿单机，没有接触过这些东西的话，权当是为你打开一扇新的大门吧。
<!--more-->

### 大的单体项目有多蛋疼
以前我们做单机系统的时候，所有的代码都在一个项目里面，只是不同的模块按照包名来划分的。我们以前做的一个某省的教育项目，有学生信息和就业系统、有高校培训系统、有一个人资系统等一共六个，4个小伙伴都在一个代码里面进行开发，各个系统之间有一定的联系，但是大部分是不相关的，但管理页面在一起。

那时候我们都在一个项目里面码代码，每次启动好几分钟，还有就是包版本冲突问题，搞得真是蛋疼。大家经历过大型的单体项目开发，相信你有体会的。

还有各系统的使用量也不一样，有的比较大。比如学生信息和就业系统，面向的是所有高校，特别是快毕业那段时间，每个学校会上报就业率等信息，还有就是打印报到证呀什么的。有的系统就使用比较少，比如人资、培训系统 使用的基本上就教育厅的一些员工，和部分老师，流量不大，勉强能扛得住。


### 模拟业务背景

大点的企业，比如做电商的，用户几十万的，日活几万的，背后好几十人上百人的团队在支撑开发，单体系统就不太合适了。

比如现在有一个下单买东西的需求，就需要订单系统、库存系统、仓库系统和积分系统 等来进行处理。如下图：

![](http://www.justdojava.com/assets/images/2019/java/image_qry/20190810-springcloud/1.png)

订单系统、库存系统、仓储和积分系统都是部署到不同的机器上的。

当用户下单了，那么订单服务会发进行扣件库存、通知仓储系统要发货、通知积分系统累加积分的操作。

如果我们此时需要用到 Spring Cloud 来做一个分布式架构的话，那么我们需要什么东西呢？每个东西都是干嘛的呢？


### 如果使用 Spring Cloud 来实现，需要哪些组件？


#### Eureka
首先，我们需要一个注册中心 Eureka ，主要负责每个服务的注册和发现。

每个微服务中都有一个Euraka client组件，专门负责将这个服务的服务id（serviceId）、ip、端口等信息注册到Eureka server中。

Euraka Server是一个注册中心，该组件内部维护了一个注册表，保存了各个服务所在的机器ip和端口号等信息。


#### Feign

其次每个服务还需要一个远程服务调用的组件 Feign ，他主要负责与其他服务建立连接，构造请求，然后发起请求来调用其他服务来获取数据。

#### Ribbon

然后我们一个服务可能会部署很多台机器，那么我们使用Feign 去调用这个服务的时候，到底把请求发送到哪台机器上去呢？此时我们就需要一个组件来根据一定的策略来选择一台机器。不管怎么选的，总之得选一台机器给 Feign 去调用就好了。

这个组件就是 Ribbon，Ribbon 主要负责就是负载均衡。Ribbon 会定期去从Eureka 注册中心拉取注册中心，缓存到本地，每次发起远程调用的时候，Ribbon 就会从 Eureka 注册表拉去下来的数据中挑选一个机器让 Feign 来发起远程调用。

#### Zuul

我们这么多的微服务，如果一个服务一个IP，使用方都需要进行调用的话，是不是得知道每一个服务的IP地址才行呢？那得记住多少才行呀，多不好管理。

如果有一个统一的地址，然后根据不同的请求路径来跟我进行转发多少是不，比如 /user/* 是转发到用户服务 ，/product/* 是转向到商品服务等等。我使用的时候，只需要访问同一个IP ，只是路径不一样，就行了。

Spring Cloud 也给我们提供了一个组件，那就是 Zuul ，他是一个网关，就是负责网络的路由的。每个请求都经过这个网关，我们还可以做统一鉴权等等很多事情。

#### Hystrix

还有一个东西也得说一下，就是 Hystrix，它是一个隔离、熔断以及降级的一个框架 。

在微服务的相互调用过程中，可能会出现被调用服务错误或者超时的情况，从而导致整个系统崩溃不可用，也就是我们常说的服务雪崩问题，Hystrix 的存在就是为了结局这种问题的。


### 整体架构

我们按照以上使用到的这些组件，来往下单这个流程来套一下：

![](http://www.justdojava.com/assets/images/2019/java/image_qry/20190810-springcloud/2.png)

整个调用流程：

1. 首先每个服务启动的时候都需要往注册中心进行注册。
2. 用户先对网关发起下单请求，网关收到请求后发现呃，是下单操作，要到订单系统，然后把请求路由到订单系统。
3. 订单系统啪啦啪啦一顿操作，然后通过 Feign 去调用 库存系统减库存，通知仓储服务发货，调用积分系统加积分。
4. 在发起调用之前，订单系统还得通过Ribbon 去注册中心去拉取各系统的注册表信息，并且挑一台机器给 Feign 来发起网络调用。

### 最后

OK，以上就是整个Spring Cloud 的核心架构了，面试题额，别错过了，朋友。这只是给大家一些普及，面试的时候遇到了可以这么去说的。每个组件的细节，后续继续写。

下面是搭建了一个脚手架测试代码，大家可以跑一下，体验一下面的流程。

https://github.com/heyxyw/spring-cloud/

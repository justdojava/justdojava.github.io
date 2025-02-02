---
layout: post
categories: Java
title: Java 程序员可以学习的技术方向，精通任何一个都可以成为专家
tagline: by 子悠
tags: 
  - 子悠

---

Hello 大家好，相信大家跟阿粉一样，在成为卓越的` Java` 程序员的路上从未停止过学习，作为一个 `Java` 程序员还有很多我们需要学习的东西，特别是在这样一个技术快速发展的时期可能昨天还在流行的技术，转眼就已经落后了。那么在 2021 年已经接近尾声的时候有哪些技术我们可以继续不断的学习呢？

<!--more-->

### JDK 源码

毫无疑问作为` Java` 程序员 `JDK` 的源码是我们一直需要不断学习的一个技能。最新发布的版本是在今年 3 月份发布的 `Java SE 16`，前两个较成熟的版本的 `Java 11` 和 `Java 8`，因为这两个版本相对维护的时间会较长，属于` LTS（Long Time Support）`。

对于我们开发者来说，日常工作的项目肯定是要在稳定版本上的，但是日常的学习就可以随意发挥。通过阅读优秀的人写的代码来提高我们自己的能力，附一张 `Java` 语言发布史。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/0820/1.png)

从这张图中我们可以看到` Java` 版本有四种类型，分别是旧版本，旧版本依旧维护，当前版本，未来版本。当前版本是` Java SE16`，未来会有 `Java SE 17` 和` Java SE 18`。而我们常用的 `Java SE 8` 和 `Java SE 11` 属于两个 LTS 虽然是旧版本但是依旧在维护。

另外我们可以知道 `Java SE 17` 将会是一个 LTS 版本，虽然还没有发布，但是我们可以通过学习` Java SE 16` 来提前了解。

附上` Java SE 16` 的下载地址：https://www.oracle.com/java/technologies/javase-jdk16-downloads.html 大家可以自行选择适配的操作系统进行下载学习。

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/2021/0820/2.png)



### RestFul Web Service

近几年 `RestFul `风格也较为流行，所谓 `RestFul `是一个设计风格，通过 `URL` 和`HTTP` 的动词来表示要进行的操作。可能对于一些小伙伴来说只知道 `HTTP` 有 `GET` 和 `POST` 方法，其实 `HTTP` 的动词除了这两个常用的还有 `PUT`，`PATCH`，`DELETE`，对应的说明如下，其中 `POST`，`DELETE`，`PUT`，`GET` 对应的就是我们常说的总删改查：

- `GET（SELECT）`：从服务器取出资源（一项或多项）。
- `POST（CREATE）`：在服务器新建一个资源。
- `PUT（UPDATE）`：在服务器更新资源（客户端提供改变后的完整资源）。
- `PATCH（UPDATE）`：在服务器更新资源（客户端提供改变的属性）。
- `DELETE（DELETE）`：从服务器删除资源。

例如在进行 API 设计的时候，假设我们有个用户管理的页面需要设计 API，则我们可以设计如下 API

1. `GET /users，GET /users/id`：查询用户列表或者一个用户信息；
2. `POST /users`：创建一个用户信息；
3. `PUT /users/id`：更新一个用户信息；
4. `DELETE /users/id`：删除一个用户信息；

其中 `id` 代码用户编号，通过这种路径参数的形式我们就实现了 `RestFul` 风格的设计，当然如果有多层关系我们可以继续加路径，比如要获取某个班级的某个同学，则可以设计 `GET /classes/1/student/2` 表示要获取 1 班学号为 2 的同学信息。`Spring` 目前是支持 `RestFul`风格的，可以直接使用路径参数就行。

### Spring Framework

前面提到 `Spring` 支持路径参数，`Spring` 作为` Java` 领域的优秀框架，我相信目前很多小伙伴应该都在使用，那如果在有时间和精力的情况下，再学习一下 `Spring` 的源码，这样不管在工作中还是面试中都会有很不错的表现。

很多时候我们可以通过看别人写的优秀的代码来提高自己的代码水平。像 `Spring` 这样优秀框架代码，很值得我们去深入研究一下。

### Serveless 架构

`Serveless` 架构可能很多小伙伴还没听过，而且很多小伙伴可能在日常工作中除了写需求代码之外还会涉及到服务器的配置以及运维的工作。而在当下的云原生时代，所有的这一切都可以交给云服务器厂商，关于 `Serveless` 架构大家可以去看一下公号之前的文档，[作为 Java 开发程序员，你知道什么是 Serveless 架构吗？](https://mp.weixin.qq.com/s/lsom8niu8ClaDjy5fA2gqw)

写的比较清楚，而且也有案例。

### 大数据开发

身为` Java` 程序员很多时候我们可能都在写一些业务代码，没有很多的数据量，但是这并不代表我们不需要学习大数据处理能力。虽然说大数据开发是专门一个领域，但是如果我们在工作中懂得这一部分的内容，那升职加薪不是你还会是谁呢？关于大数据相关的知识要学习的也有很多，涉及到的主要是计算和存储。技术点有很多，像 `Hadoop`，`MapReduce`，`HDFS` 这些都是很经典的基石。而像这两年比较火的 `Flink` 以及 `Clickhouse` 都是很不错的技术，感兴趣的小伙伴都可以尝试去学习一下，虽然工作中不一定会用到，但是日常学习还是很不错的。

### 机器学习/深度学习

最后一个机器学习以及深度学习这块的内容大家可以作为扩展知识去学习了解，这块的内容说真的难度还是蛮高的，但是可以知道是一直是未来方向，而且这一块的工资比普通的开发工程师高很多。如果是刚毕业的同学对这块感兴趣的话可以考虑从事这方便的学习，那如果是已经工作几年了小伙伴想往这个方向转的话，可以需要好好学习一下，或者报个培训班都是可以的。



除了上面提到的这几点我们可以去深入学习，其实还有很多，比如现在很火的可穿戴设备，自动驾驶，`DevOps`，云计算 等等。不得不说程序员这一行要学习的东西太多了，想要不被淘汰与时俱进，持续学习是不变的道理。


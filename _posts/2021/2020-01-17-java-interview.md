---
layout: post
categories: Java
title:  又快到了跳槽季，分享一次上市公司面试经历
tagline: by 小黑
published: true
tags: 
  - 小黑
---
Hello，大家好，我是阿粉。

又到一年年底了，过完年就到了跳槽的高峰期，不少朋友应该也在摩拳擦掌了吧。

最近阿粉的朋友出去面了一圈大厂，独角兽，上市公司，积累一些面试经验，阿粉特地跟他交流了一下，获取一手面试题。

下面开始进入正文。

这次是一家做电商软件上市公司，名字就不具体介绍，其业务跟有赞类似。

这次面试基本没问业务，全部问的都是技术点。

这里说明一下，每个面试官的风格真的都不一样。阿粉之前的面试经历，基本都有会先让你介绍业务，然后再从业务抓住某些点深入问。

这种面试风格就循序渐进，在这过程中你也会慢慢进入面试节奏。

阿粉的朋友本来以为也是这种模式的，自我介绍完，就等面试官，说你介绍下项目吧。

脑子里都想好了怎么介绍了，冷不丁的，面试官，问了下 mysql 了解多吗，我们现来问下 msyql 的吧。

￣□￣｜｜这一下子就被破功了，然后脑子一下子没跟进节奏，后面回答的问题，就乱了。

由于这次都是技术点的面试，网上都能找到答案，所以就不带解释了。



💊自我介绍

💊事务隔离等级

💊RR 隔离等级如果解决不可重复读

💊RR 有没有解决幻读，如何解决

💊Mysql 默认事务隔离等级

💊SQL 优化经验

💊为什么索引字段加函数，就不走索引了



说说这个问题吧，问到这个的时候，这个原因就很熟悉，但是就是描述不出来。

于是我就想到先跟面试官分析了一下正常索引查找流程，简单来说就是B+树有序性，二分法定位。

而这时索引字段使用函数之后，破坏这种有序性，然后就不会根据索引走了。



💊有没有碰到 mysql iops 或 cpu 占用很高

💊mysql 日志了解吗

💊 redolog 二阶段提交了解吗

💊redolog 这个二阶段相关配置了解吗

💊binlog 主从不一致有碰到过吗

💊mysql 一些配置了解吗



上面都是 mysql 相关的问题，下面开始问了一些 JVM 问题。



💊JVM 内存区域

💊三大常量池了解吗

💊堆内存结构

这个问题，刚开始听到有点懵，后来再问了一下面试官是不是想了解年轻代，老年代这些，面试回答是的。

所以球友们如果在面试中碰到没清除，或者不理解的问题，可以让面试详细说清楚问题。

**不用害怕，面试是一个双向的过程。**

💊GC 算法-复制算法

💊GC 算法-标记整理

💊GC 算法-标记清除

💊三种算法优缺点比较

💊老年代担保是什么

💊GC 算法中标记是什么对象

💊Full GC 触发条件

💊OOM 问题怎么排查

💊 Dump日志分析工具



下面又开始另一块问题，IO 相关问题。

💊解析一下 BIO，NIO 模型

💊Selcet 与 Epoll 的区别

💊为什么 Epoll 比较高效

💊Select 是不是同步的

💊Epoll 是不是同步

💊直接缓冲与间接缓冲区别



最后一块问题，Java 集合相关问题。

💊HashMap,CurrentHashMap 区别

💊CurrentHashMap 1.7 与 1.8 区别

💊CurrentHashMap Put 的过程

💊多线程使用的例子

💊线程池使用经验

💊线程池参数解释

💊死锁解释

💊如何排查死锁



最后其实还问了简单问题了 Redis 的用来做了什么，最后最后到了我们的问题。

**你还有什么要问的吗？**

常规性问题，问下自己感兴趣，技术面就不要问薪资相关的。

## 总结

别看阿粉的朋友现在分析头头是道，其实真正在面试的时候有些问题回答有点乱，其实心里慌了，人就会紧张，一紧张，大脑就越空白。

现在冷静下来分析，其实大部分技术点都是会的，也准备过的。

所以说面试中如何保持沉着冷静真的挺重要的，可惜这次还是慌了，￣□￣｜｜。

现在看来，其实就问技术点的面试，其实相对简单，这是因为这些都是有标准答案的，会就是会，不会就不会了。

相反，一些架构思考问题，相对来说考察不仅是你架构思维，还有平常积累，以及表达能力。

好了，今天面试题就到这了，后续再跟大家分享其他面经。

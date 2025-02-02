---
layout: post
categories: Java
title: Java 中的接口还可以这样用，你知道吗？
tagline: by 子悠
tags: 
  - 子悠
---

`Java` 程序员都知道要面向接口编程，那 `Java` 中的接口除了定义接口方法之外还能怎么用你知道吗？今天阿粉就来带大家看一下 `Java` 中的接口还可以有哪些用法。

<!--more-->

## 基本特性

我们先看一下接口的基本特性

1. 接口的定义需要使用关键字 `interface`；
2. 接口定义的所有方法默认都是 `public abstract`；
3. 当一个具体的 `class` 去实现一个`interface`时，需要使用`implements` 关键字；
4. 接口之间是可以多继承，而类是只能单继承的；

如下所示，我们定义一个接口

```java
package com.example.demo.inter;

import java.io.Serializable;
import java.util.RandomAccess;

/**
 * <br>
 * <b>Function：</b><br>
 * <b>Author：</b>@author java 极客技术<br>
 * <b>Date：</b>2022-09-24 17:38<br>
 * <b>Desc：</b>无<br>
 */
public interface ITest extends Serializable, RandomAccess {

  public abstract String sayHello();
  String sayHello2();// public abstract 可以省略

}

```

## 默认方法

在 `JDK 8` 之前接口是不支持默认方法的，在 `JDK 8` 之后接口支持默认方法，默认方法采用关键词 `default` 声明。

```java
public interface ITest extends Serializable, RandomAccess {

  public abstract String sayHello();
  String sayHello2();// public abstract 省略
  
  // 默认方法
  default String sayHello3() {
    return "hello3";
  }

}
```

默认方法跟抽象方法不一样，接口中定义的抽象方法，当接口被其他类实现的时候都需要全部实现，但是默认方法是不需要被实现就可以直接使用的，类似于直接调用父类的方法一样，所以在很多时候，我们已经继承了一个类，还想有一个能用但是不想每个子类都实现的方法的时候，就可以考虑增加一个接口的默认方法来使用，简单来说就是实现类可以不覆写 `default` 方法。

`default`方法存在的目的是，在我们已经完善的项目中，如果我们直接给一个接口增加一个方法，在没有默认方法的时候就需要给所有的实现类都实现对应的方法，但是这个方法又不一定是每个实现类都需要的，所以这个时候默认方法就很好的解决了这个问题，我们只需要增加一个默认方法，然后在需要使用的实现类中进行实现或者使用就可以了，其他的实现类不需要改动任何的代码。

## 标记

接口还有一个很常见的功能那就是标记功能，这么说可能你没有印象，但是到提到序列化接口 ` java.io.Serializable;` 你肯定知道，我们经常在对应的 `POJO` 中都会实现这个序列化接口，而这个序列化的接口如果看过源码的小伙伴肯定知道里面是没有内容的。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h6hycyqs30j20xk0eutae.jpg)

同样的功能除了序列化的接口，类似的空接口还有很多，比如 `java.util.RandomAccess` 也是空接口，之前阿粉也写过关于 `RandomAccess` 这个接口的用途，感兴趣的可以再去看看。 [RandomAccess 明明是个空接口，能有什么用呢？](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491491&idx=1&sn=e2bbafecd16612dea8eab677ec2d10b2&chksm=c2857062f5f2f974e90de4d6b5476f2d128a86d72fe069c701d86cb05f2742548e869b066017&token=1447702434&lang=zh_CN#rd)

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h6hye3blgoj20z40gowfx.jpg)

通过源码我们可以知道 `RandomAccess` 是用来标识子类是否实现了该接口，如果实现了则走实现了的逻辑，没有实现就走没有实现的逻辑，所以我们在日常开发中也可以利用这个特性，当我们有不同的子类需要根据情况进行不同的实现逻辑的时候就可以采用定义一个空接口来标记一下，方便后面的处理。

## 静态方法

跟默认方法一样，`JDK 8` 还支持在接口中增加静态方法，虽然说在接口中定义静态方法的做法不常见，但是当需要使用的时候也是可以支持的，避免在创建一个单独的工具类，跟在类中定义的静态方法一样，我们可以直接通过接口名称引用静态方法，当然静态变量也是可以的，使用方法一样。

```java

public interface ITest extends Serializable, RandomAccess {

  public abstract String sayHello();
  String sayHello2();// public abstract 可以省略
  default String sayHello3() {
    System.out.println(sayHello4());
    return "hello3";
  }
  public static String sayHello4() {
    return "hello4";
  }
}

```

## 私有方法

大家有没有注意到，上面不管是默认方法还是静态方法其实都是 `public` 公开的，可以让实现类或者其他类直接使用，那有没有可能在接口中定义一个私有方法呢？在 `JDK 9` 之前是不可以的，`JDK 9` 却支持了，如下所示

```java
  private void privateMethod() {
    System.out.println("private私有方法被调用");
  }

  private static void privateStaticMethod() {
    System.out.println("private私有静态方法被调用");
  }
```

从官方的文档中我们可以找到下面的描述，在 `JDK 9` 中接口支持私有方法，主要用于不同的非抽象方法中共享代码。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h6iwd13cmpj22580fwtcs.jpg)

我们可以看到在 `JDK 9` 这样写是可以的

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h6iwvadoj0j20z60rgacz.jpg)

但是在 `JDK8` 就不行了，是无法编译通过的，会提示不允许使用 `private` 修饰符。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h6iww2vn6aj215m0pqtc1.jpg)

## 总结

今天阿粉给大家总结了一个接口的使用方法，后面在日常的开发中我们不单单只是在接口中定义抽象方法，也可以根据需求增加默认方法或者私有方法，以及在需要用到标记的时候也可以通过定义一个空接口来实现，怎么样是不是很棒，感觉动起来吧。最后觉得我们的文章有帮助，欢迎一键三连。

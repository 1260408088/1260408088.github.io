---
title: springboot中@value注解不生效
date: 2020-11-18 22:40:42
categories: [springboot]
tags: [springboot,注解]
---

1. 注解修饰的变量不能是final、static。
2. 类要使用交给spring管理，使用@Component修饰所在类，不能有构造方法(我失败是因为这个)。
3. 不能new这个类，要@Autowried注入。
4. 在使用@Value，可以指定默认值，比如@Value("${local-repository:<spans tyle="color:red">/repository/</span>}") ,红色部分为默认值

尾记：spring加载这个bean后要直接调用其中的一个方法，可以使用@postconstruct注解，能达到这个作用的方式还很多，留个空白，以后补充。
---
layout: post
title: 探秘Spring与Mybatis的集成原理
category: spring
tags: Spring Mybatis IoC AOP
keywords: Spring Mybatis IoC AOP
description: 探秘Spring与Mybatis的集成原理
---
## 概括
Spring提供了多个扩展点，供第三方在Bean初始化的各个阶段进行自己额外功能的扩展

- BeanFactoryPostProcessor
- BeanPostProcessor
- FactoryBean

## Spring初始化Bean
![]({{ site.baseurl }}/assets/img/IoC_all.svg)

## Mybatis初始化

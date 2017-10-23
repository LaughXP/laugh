---
layout: post
title: PowerMock文档翻译-配置
category: translation
tags: PowerMock
keywords: test PowerMock translation
description: PowerMock的配置
---
> [原文链接](https://github.com/powermock/powermock/wiki/PowerMock-Configuration)

# PowerMock 配置

```
从1.7.0开始，PowerMock开始启用配置功能
```
1.7.0版本可以使用全局配置文件。全局配置适用于`classpath`下所有的测试用例。你可以通过在`classpath`创建下面这这个文件来实现全局配置:

```java
org/powermock/extensions/configuration.properties
```
### 全局的@PowerMockIgnore实现方式
默认情况下是由[MockClassLoader](https://www.javadoc.io/doc/org.powermock/powermock-core/1.7.1)来加载所有类的。这个类加载器加载和修改除了以下两种类之外的所有类：
  - 系统类。这些类被系统的类加载器延迟加载
  - 被指定忽略加载的类。
在1.7.0之前，只有通过注解[@PowerMockIgnore](https://www.javadoc.io/doc/org.powermock/powermock-core/1.7.1)这一种方式来实现指定被忽略加载的包：

```java
 @PowerMockIgnore("org.myproject.*")
 @PrepareForTest(MyClass.class)
 @RunWith(PowerMockRunner.class)
 public class MyTest {
 ...
 }
```
在[某些情况](https://github.com/powermock/powermock/wiki/FAQ)下，为了避免某些类加载相关问题的唯一途径就是使用`@PowerMockIgnore`，但是这也意味着你会在需多测试用例中进行拷贝这个注解。
但是在1.7.0版本后，你可以使用配置文件来忽略指定的包：

```java
powermock.global-ignore="org.myproject.*"
```
可以使用逗号分隔多个包或者类：

```java
powermock.global-ignore="org.myproject.*","org.3rdpatproject.SomeClass"
```
这里的[例子](https://github.com/powermock/powermock-examples-maven/tree/master/global-ignore)是使用全局PowerMock配置文件来实现全局`@PowerMockIgnore`。

### Mockito mock-maker-inline
PowerMock可以委托调用其他的`MockMaker`，这些测试用例可以不通过PowerMock来运行。
可以通过在配置文件中增加一下属性来实现：

```java
mockito.mock-maker-class=mock-maker-inline
```
这些[例子](https://github.com/powermock/powermock-examples-maven/tree/master/mockito2)是在PowerMock中使用Mockito的`mock-maker-inline`

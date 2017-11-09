---
layout: post
title: PowerMock文档翻译-原因
category: translation
tags: PowerMock
keywords: test PowerMock translation
description: 作者开发PowerMock的原因
---
> [原文链接](https://github.com/powermock/powermock/wiki/Motivation)

## 原因
PowerMock是一个Java mock框架，它可以用来测试一些被认为是困难或者不可能进行测试的问题。通过使用PowerMock，你可以mock静态方法，去除静态初始化，可以不通过依赖注入直接进行mock等等。PowerMock是在执行测试用例后，通过在运行时修改字节码实现这些功能的。PowerMock还提供一些工具类，使用者通过这些工具可以更容易的获得一个对象的内部状态。

下面举例一些情况，表明使用PowerMock比其他方法更高效:
### 使用第三方或者一些旧的框架
框架一般都是通过静态方法进行类之间通信(例如Java ME)。PowerMock允许你设置静态方法的预期返回值和模拟你需要进行测试的行为。

不使用PowerMock：把所有的静态方法调用封装在一个单独的类，并使用依赖注入模拟这个类。这会在你的代码中额外增加一个不必要的调用层。不过，如果你觉得封装一层对你的架构有用也是可以的。

有一些框架需要你的子类包含静态初始化(Eclipse)或者构造方法(Wicket)。PowerMock允许你
去除静态初始化和禁用构造函数。PowerMock还允许你模拟jar(old Acegi,Eclipse)中的类，即使这个类是私有的。

不使用PowerMock：把你的代码都封装到委托类中，让框架之间的调用都通过委托进行。
### 构思
如果你非常想测试的方法是私有方法。PowerMock允许你模拟`private`和`final`方法。

不使用PowerMock：私有方法不可能被直接模拟。你可以创建一个新类并把private方法移动到该类作为`public`方法。并且使用依赖注入进行模拟。不过，以上绝对是一个你不想使用的架构。
### 性能?
如果你觉得依赖注入的代价太高，可以使用静态方法调用或者静态工厂。再比如你有大量的对象或者你使用的是Java ME。PowerMock允许你使用静态方法提高性能和进行模拟。

不使用PowerMock：为每个服务创建单例，并且总得确保在测试时创建了唯一的实例。这有时会打破单例的设计模式并且会增加改变实例的可能性。

(译者按：总之，使用PowerMock可以很方便的模拟`private`、`final`、`static`方法和属性)

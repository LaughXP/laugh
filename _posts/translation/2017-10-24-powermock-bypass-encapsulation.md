---
layout: post
title: PowerMock文档翻译-绕过封装:白名单
category: translation
tags: PowerMock
keywords: test PowerMock translation
description: PowerMock的配置
---
> [原文链接](https://github.com/powermock/powermock/wiki/Bypass-Encapsulation)

## 绕过封装
### 概要
[Whitebox](http://www.javadoc.io/doc/org.powermock/powermock-reflect/1.7.1)提供了一些方法来帮助你绕过封装。通常，不推荐修改一个非公共字段，但是有时候为了更好的进行优化，我们必须通过这种方式进行测试。
1. 使用`Whitebox.setInternalState(..)`给类或者类的实例的非公共成员变量设置值。
2. 使用`Whitebox.getInternalState(..)`获得类或者类的实例的非公共成员变量的值。
3. 使用`Whitebox.invokeMethod(..)`调用类或者类的实例的非公共方法。
4. 使用`Whitebox.invokeConstructor(..)`可以为拥有私有构造函数的类创建实例。
### 示例
##### 访问内部状态
当一个可变对象的方法被调用时，可能会改变其内部状态。当测试这类对象时，有一个很简单的方法可以察觉到其内部状态是否已经更新。PowerMock为此提供了几个反射工具类专门应用于单元测试。所有的反射工具类在包目录`org.powermock.reflect.Whitebox`下面。

为了进行演示，我们假设有这样一个类:
```java
public class ServiceHolder {

	private final Set<Object> services = new HashSet<Object>();

	public void addService(Object service) {
		services.add(service);
	}

	public void removeService(Object service) {
		services.remove(service);
	}
}
```
假设我们想测试`addService`方法(可能会被认为这个示例太过简单了，但是不要介意，我们只是为了方便演示)。在这里假如我们想确认`ServiceHolder`在方法`addService`被调用后能正确更新其状态，也就是为`services`集合添加一个对象元素。可以通过为此类添加一个包私有或者`protected`的方法比如`getService`来获得`services`属性的值。但是这样做的话，我们想就相当于在`ServiceHolder`类中添加了一个除了测试没有其他作用的方法。
##### 调用私有方法

##### 实例化私有构造函数的类

##### 注意

##### 参考代码

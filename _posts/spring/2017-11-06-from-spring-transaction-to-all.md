---
layout: post
title: 从Spring事务管理器窥探Spring容器
category: spring
tags: Spring transaction IoC AOP
keywords: Spring transaction IoC AOP
description: 从Spring事务管理器窥探Spring容器
---
## 事务管理抽象
![Alt text]({{ site.baseurl }}/assets/img/transaction.png)
- TransactionDefinition
  - 事务隔离级别
  - 事务传播行为
  - 事务超时时间
  - 只读状态
- TransactionStatus
代表一个事务的运行状态，事务管理器可以通过该接口获得事务运行期的状态信息，也可以通过该接口间接的回滚事务。
- PlatformTransactionManager
该接口负责进行事务管理。
```java
public interface PlatformTransactionManager {
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
	void commit(TransactionStatus status) throws TransactionException;
	void rollback(TransactionStatus status) throws TransactionException;
}
```

## 事务管理实现方式

### 编程式事务管理
编程式事务管理基本上很少用到，但是在某些场合下还是一柄利器，可以解决比较棘手的问题。使用编程式事务管理，必须要用到`TransactionTemplate`。
![Alt text]({{ site.baseurl }}/assets/img/TransactionTemplate.jpeg)
`TransactionTemplate`主要有两个方法
- `void setTransactionManager(PlatformTransactionManager transactionManager)`设置事务管理器
- `<T> T execute(TransactionCallback<T> action)`执行回调接口中的事务

### 使用XML配置声明式事务管理
- 使用原始的`TransactionProxyFactoryBean`对事务增强目标进行食物增加代理
- 基于aop/tx命名空间的配置

### 使用注解配置声明式事务管理
即`@Transactional`注解。前提是Spring的配置文件中标注

```xml
//transaction-manager可以不用指定，默认是transactionManager
<tx:annotation-driven transaction-manager="transactionManager" />
```
值得注意的是，`@Transactional`可以用在接口定义、接口方法、类定义、类的public方法上。方法上的注解会覆盖类上的注解。

### 异常回滚规则
- 运行期异常(所有由`RuntimeException`派生的Exception)回滚
- 检查型异常(所有不是由`RuntimeException`派生的Exception)不回滚

## Spring事务初始化
首先，要了解Spring事务初始化，必须要先了解AOP。其次，了解AOP还必须要先了解IoC容器中Bean的生命周期，这样才能融会贯通，柳暗花明。

### IoC容器中Bean的生命周期
![Alt text]({{ site.baseurl }}/assets/img/IoC.svg)

### AOP

#### AOP实现
![Alt text]({{ site.baseurl }}/assets/img/AOP.svg)

#### AOP术语
- **连接点(Joinpoint)**
程序执行的某个特定位置：如类开始初始化前、类初始化后、类某个方法调用前、调用后、方法抛出异常后
- **切点(Pointcut)**
每个程序类都拥有多个连接点，如一个拥有两个方法的类，这两个方法都是连接点，即连接点是程序类中客观存在的事物。AOP通过切点定位特定的连接点。连接点相当于数据库中的记录，而切点相当于查询条件。切点和连接点不是一对一的关系，一个切点可以匹配多个连接点。
- **增强(Advice)**
增强是织入到目标类连接点上的一段程序代码
- 目标对象(Target)
增强逻辑的织入目标类
- **引介(Introduction)**
引介是一种特殊的增强，它为类添加一些属性和方法。这样，即使一个业务类原本没有实现某个接口，通过AOP的引介功能，我们可以动态地为该业务类添加接口的实现逻辑，让业务类成为这个接口的实现类。    
- **织入(Weaving)**
织入是将增强添加对目标类具体连接点上的过程。AOP像一台织布机，将目标类、增强或引介通过AOP这台织布机天衣无缝地编织到一起。根据不同的实现技术，AOP有三种织入的方式：
  - 编译期织入，这要求使用特殊的Java编译器。
  - 类装载期织入，这要求使用特殊的类装载器。
  - 动态代理织入，在运行期为目标类添加增强生成子类的方式。
Spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入。
- **代理(Proxy)**
一个类被AOP织入增强后，就产出了一个结果类，它是融合了原类和增强逻辑的代理类。根据不同的代理方式，代理类既可能是和原类具有相同接口的类，也可能就是原类的子类，所以我们可以采用调用原类相同的方式调用代理类。
- **切面(Aspect)**
切面由切点和增强（引介）组成，它既包括了横切逻辑的定义，也包括了连接点的定义，Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的连接点中。Spring中主要有以下三个切面概念
  - Advisor:一般切面，只包含一个Advice
  - PointcutAdvisor: 包含具体切点和增强的切面
  - IntroductionAdvisor: 类层面的引介切面

### Spring如何加载事务增强
以注解配置声明式事务管理为例子

#### 关键点
- `@Transactional`注解标注的位置对应连接点
- `TransactionAttributeSourcePointcut`对应切点
- `TransactionInterceptor`对应增强
- `BeanFactoryTransactionAttributeSourceAdvisor`对应切面

#### 解析tx:annotation-driven
![Alt text]({{ site.baseurl }}/assets/img/tx_init.svg)

#### Spring生成代理类
![Alt text]({{ site.baseurl }}/assets/img/tx_op.svg)

## SpringAOP(事务)注意事项

### JDK动态代理
在JDK动态代理中通过接口来实现方法拦截，所以必须要保证被拦截的目标方法在接口中有定义，否则将无法拦截

### CGLIB动态代理
在CGLIB动态代理中通过动态生成子类来实现方法拦截，所以必须要确保被拦截的目标方法可被子类访问，也就是目标方法必须定义为非`final`,非`private`方法

### 类内部调用自身方法无法进行代理
以CGLIB动态代理为例子
- 原始类

```java
public class Source{
  @Transactional
  public void add(){
    //其他代码
    try {
      minus();
    }catch (Exception e) {
    }
  }
  @Transactional
  public void minus(){
  }
}
```
- 代理类

```java
public class SourceProxy extends Source{
  public void add(){
    //增加事务管理
    super.add();
  }

  public void minus(){
    //增加事务管理
    super.minus();
  }
}
```
- 执行示例

```java
public class Test{
  @Autowired
  Source source;

  public void test() {
    source.add();
  }
}
```

在执行`Test#test()`方法调用`Source#add()`时，如果`Source#minus()`发生异常，`minus`方法并不会进行回滚，因为`Source#minus()`并没有加入事务管理器。这是因为在实际执行过程中调用的是代理类`SourceProxy#add()`，进一步调用的是`Source#add()`，只有当`Source#add()`发生异常时才会进行事务处理(`Source#minus()`产生的异常被捕获了)
- 解决方案
  - 使用编程式事务处理(也就是自己手动实现事务逻辑，最原始的AOP方案，人肉写切面代码，业务逻辑通过回调处理)
  - 在需要进行代理的Bean中注入自身代理类，通过调用代理类的方法去实现内部调用，如下

```java
public class Source{
  private Source sourceProxy;
  @Transactional
  public void add(){
    //其他代码
    try {
      sourceProxy.minus();
    }finally{

    }
  }
  @Transactional
  public void minus(){
  }
}
```

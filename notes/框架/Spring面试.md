## 1. Spring框架中用到了哪些设计模式？

1. 工厂模式

    在Spring中，BeanFactory就是工厂模式的实现，Spring通过BeanFactory来创建Bean。 

2. 单例模式

    在Spring中默认bean均为单例的。 
    
3. 适配器模式

    在Spring的AOP中，通知（Advice）就是通过适配器模式来实现的。AdvisorAdapter。

4. 包装器模式

5. 代理模式

    在Spring的AOP中，JDKDynamicAopProxy和Cglib2AopProxy都使用到了代理模式。

6. 观察者模式

    在Spring中的监听器，就是使用到了观察者模式，例如ApplicationListener。
    
7. 策略模式

8. 模板方法模式

    在Spring中的JDBCTemplate就是用到了模板方法模式。
    
## 2. 使用Spring框架有什么好处？

- 轻量：Spring是轻量级的，基本的版本大于2MB
- 控制反转：Spring的核心思想，通过控制反转来实现松散耦合，对象们给出它们的依赖，而不是创建或查找依赖的对象们
- 面向切面编程（AOP）：Spring支持面向切面编程，并且把应用业务逻辑和系统服务逻辑分开，实现自己的逻辑增强
- SpringIOC容器：Spring提供了SpringIOC容器，用于管理应用中对象的生命周期和配置
- MVC框架：Spring提供了WEB的MVC框架
- 事务管理：Spring提供了一个持续的事务管理接口，可以扩展到上至本地事务，下至全局事务（JTA）
- 异常处理：Spring提供了方便的API把具体的技术相关的异常转化为统一的unchecked异常

## 3. SpringIOC核心容器模块

在Spring中，IOC容器是整个框架的基础，而BeanFactory是任何以Spring为基础的应用的核心，提供了创建bean，管理bean，获取bean的核心API。Spring应用建立在IOC容器之上。

BeanFactory是工厂模式的一个实现，它给Spring提供了控制反转功能，用来把应用的配置和依赖从代码中分离出来。BeanFactory最常见的实现类是：XMLBeanFactory。

> ApplicationContext

ApplicationContext和BeanFactory都是接口，都是用于加载Bean的，但是ApplicationContext相比于BeanFactory，它在BeanFactory的基础工鞥之上还提供了更多扩展功能，而ApplicationContext更适用于企业级开发。

## 4. 一个Spring Bean的定义包含什么？如何给Spring提供配置元数据？

一个Spring Bean（BeanDefinition）包含了容器必知的所有"配置元数据"。在Spring中有三种方式给Spring容器提供配置元数据：

1. XML配置方式
2. 基于注解的配置方式
3. 基于Java配置

## 5. Spring支持的几种Bean的作用域

Spring框架支持以下五种bean的作用域：

- singleton：单例，bean在每个Spring IOC容器中只有一个实例
- prototype: 一个bean的定义可以有多个实例
- request：每次http请求都会创建一个bean，该作用域仅支持在WEB应用环境
- session：在一个HTTP session中，一个bean定义对应一个实例，同样的该作用域仅支持在WEB应用环境
- global-session：在一个全局的HTTP session中，一个bean定义对应一个实例，同样的该作用域仅支持在WEB应用环境

## 6. Spring中单例bean是线程安全的嘛？说一下Spring中Bean的生命周期

Spring框架中的单例bean不是线程安全的。


1. Spring容器从XML、注解等方式获取bean的定义
2. 实例bean
3. Spring容器会根据bean的定义通过setter方法来填充所有的属性
4. 如果bean实现了BeanNameAware接口，Spring传递bean的ID到setBeanName方法
5. 如果bean实现了BeanFactoryAware接口，Spring传递BeanFactory到setBeanFactory方法
6. 如果bean有任何相关联的BeanPostProcessor，则Spring会在postProcessBeforeInitialization()方法内调用他们
7. 如果bean实现了Initialization了，调用它的afterPropertySet方法；如果bean声明了初始化方法 init-method，则会调用init-method指定的初始化方法
8. 如果bean有任何相关联的BeanPostProcessor，则Spring会在postProcessAfterInitialization()方法内调用他们【注意BeanPostProcessor中会定义postProcessBeforeInitialization和postProcessAfterInitialization】
9. 如果bean实现了 DisposableBean，它将调用destroy()方法


## 7. 可以在Spring中注入一个null和一个空字符串吗？

可以的

## 8. Spring是如何解决循环依赖的？

在Spring中，单例Bean是会出现循环依赖问题，而原型（Prototype）是不支持循环依赖的。

那么在单例Bean场景中，Spring是如何解决循环依赖的？

在Spring中是通过内部维护的三个Map，也就是晚上通常说的"三级缓存"（虽然官方文档中是没有这种说法的）
在Spring中的DefaultSingletonBeanRegistry类中，会包含了三个Map：

- singletonObjects
    
    俗称的"单例池""容器"，缓存创建完成生成bean的地方
        
- singletonFactories

    映射创建Bean的原始工厂

- earlySingletonObjects

    早期的Bean，在这个Map中存的不是bean，而是instance实例
    
对于singletonFactories和earlySingletonObjects是用来赋值创建bean的，创建bean完成之后就清理掉各自map中的元素。

在深入源码步骤之前，需要提前知道一点就是，Spring创建一个bean是需要经过两步：
1. bean对象的实例
2. bean对象属性的实例

整个过程需要涉及到下面四步方法的依次调用： 
1. getSingleton
2. doCreateBean
3. populateBean
4. addSingleton

例如A类中依赖了B，B类中依赖了A。

首先A调用getSingleton()，当来到doCreateBean()时，会设置属性earlySingletonExpose为true，然后将A的bean工厂添加到singletonFactories中，然后调用populateBean()时发现A中依赖了B，则又会
调用B类的getSingleton()，同样的调用doCreateBean()时会设置earlySingletonExpose为true，然后将B的bean工厂添加到singletonFactories中，然后调用populateBean()进行属性填充，此时发现B类中依赖
了A类，所以又回去调用A类的getSingleton()，此时发现在isSingletonCurrentlyInCreation结合中包含了A类，表示A类正在被创建，所以回去earlySingletonObjects中获取"半成品A"，然后删除singletonFactories中的
A的bean工厂。当B拿到半成品"A"时，则从populateBean()返回，最后调用addSingleton()添加到singletonObjects中完成B的bean创建。而A方法也会从populateBean()中返回，拿到了B，则然后调用addSingleton()方法，将
bean添加到singletonObjects。
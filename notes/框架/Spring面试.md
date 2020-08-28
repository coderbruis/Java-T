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

## 8. 
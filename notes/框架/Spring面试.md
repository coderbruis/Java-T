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
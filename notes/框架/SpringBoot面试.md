## 1. SpringBoot系统初始化器解析

系统初始化器名称为：ApplicationContextInitializer

它其实就是Spring容器刷新之前执行的一个回调函数。

> 作用：向SpringBoot容器中注册属性

> 使用方式：

1. 实现ApplicationContextInitializer
2. @Order值
3. application.properties中定义的优先于其他方式

## 2. SpringFactoriesLoader

作用：
1. 框架内部使用的通用工厂加载机制
2. 从classpath下多个jar包特定的位置读取特定文件扩展类，并初始化该类
3. 文件内容必须是K/V形式的，即properties类型
4. 读取的是classpath路径下的spring.factories

## 3. 监听器模式

监听器发送顺序：

1. 框架启动
2. 发送starting事件，表明启动框架
3. 发送environmentPrepared事件，这个时间表示环境属性准备完毕
4. 发送contextInitialized时间，表示容器已经初始化
5. 发送started事件，表示SpringBoot已经加载完bean了
6. 发送ready事件，表示ApplicationRunner和CommandRunner运行完后的事件
7. 启动完毕

监听器原理：
1. 框架启动
2. 调用SimpleApplicationEventMulticaster#getApplicationListeners方法获取监听器
3. 遍历监听器然后调用supportsEvent是否是需要监听的事件
4. 调用onApplicationEvent调用监听事件的逻辑

## 4. 动手搭建starter

首先先介绍下SpringBoot的starter
1. starter是一个可插拔的插件
2. 与jar包类似，不过和jar包单纯导入class不同，starter还可以实现自动配置
3. starter能大幅度提升开发效率

> 新建SpringBoot的starter步骤

1. 新建一个SpringBoot类
2. 引入spring-boot-autoconfigure
3. 编写属性源以及自动配置类（@ConditionOnXXX）
4. 在spring.factories中添加自动配置类的实现
5. maven打包进本地maven仓库

> 使用自定义SpringBoot的starter步骤

1. pom.xml中引入新建的starter
2. 属性properties文件中设置属性开关，决定是否引入starter
3. 可以直接用了

> SpringBoot starter原理解析

starter自动配置类导入：
1. 启动类上的@SpringBootApplication
2. 引入AutoConfigurationImportSelector
3. ConfigurationClassParser中处理
4. 获取spring.factories中EnableAutoConfiguration实现

starter自动配置类：
1. @ConditionalOnProperty
2. OnPropertyCondition
3. getMatchOutCome
4. 遍历注解属性集判断environment中是否含有并且值是一直的
5. 返回对比结果
## Dubbo SPI

### JDK的SPI定义以及使用分为四步
1. 定义一个接口及接口方法
2. 编写该接口的一个实现类
3. 在META-INF/services目录下，创建一个以接口全路径命名的文件
4. 文件内容为具体实现类的全路径名，如果有多个则以逗号分隔开
5. 在代码中通过java.util.ServiceLoader来加载扩展点具体的实现类

### Dubbo SPI使用的是策略模式，为的是”对扩展开放，对修改封闭“（开闭原则）；让接口和实现解耦，为Dubbo框架的扩展性奠定了基础

### JDK SPI的缺点
1. JDK SPI会一次性加载、实例化扩展点所有实现，并且扩展点实现的初始化会非常耗时，如果扩展点没有用到，则会非常浪费资源
2. 不支持IOC和AOP，即没有和Spring集成
3. 加载过程中如果报异常了，则会被吞掉，则无法进一步排查异常

### Dubbo SPI优化的地方
1. Dubbo SPI只是加载扩展点，并不会立即全部初始化，并且会根据扩展点类型的不同而缓存到内存中。、
2. Dubbo自己实现了IOC和AOP，让一个扩展可以通过setter直接注入到其他扩展点中
3. Dubbo SPI在扩展加载失败的时候会先抛出真实异常并打印日志

### Dubbo如何自己实现了IOC和AOP？

Dubbo通过支持包装扩展类，然后把通用功能的抽象逻辑放到包装类中，然后在核心逻辑的前后插入自己的逻辑进行代码增强，从而实现了扩展点的AOP功能。

而Dubb中，通过T injectExtension(T instanc)这个方法来实现的IOC功能。

### Dubbo中扩展点有哪些分类以及缓存
Dubbo的SPI可以分为两种缓存：
1. Class缓存
    
      Dubbo SPI获取扩展类时，会先从缓存中读取。如果缓存中不存在，则加
载配置文件，根据配置把Class缓存到内存中，并不会直接全部初始化。

2. 实例缓存

  基于性能考虑，Dubbo框架中不仅缓存Class,也会缓存Class实例化后的
对象。每次获取的时候，会先从缓存中读取，如果缓存中读不到，则重新加载并缓存
起来。这也是为什么Dubbo SPI相对Java SPI性能上有优势的原因，因为Dubbo SPI
缓存的Class并不会全部实例化，而是按需实例化并缓存，因此性能更好。


这两种缓存中又可以分为普通扩展类、包装扩展类（Wrapper类）、自适应扩展类（Adaptive类）

- 普通扩展类
    
    最基础的，配置在SPI配置文件中的扩展类实现。

- 包装扩展类

  这种Wrapper类没有具体的实现，只是做了通用逻辑的抽象，并且需要
在构造方法中传入一个具体的扩展接口的实现。属于Dubbo的自动包装特性。

- 自适应扩展类
  
  一个扩展接口会有多种实现类，具体使用哪个实现类可以不写死在配
置或代码中，在运行时，通过传入URL中的某些参数动态来确定。这属于扩展点的自适应特性。

### 扩展点的特性

从Dubbo官方文档中可以知道，扩展类一共包含四种特性：自动包装、自动加载、自适应
和自动激活。

### 扩展点注解

> @SPI

@SPI注解可以使用在类、接口和枚举类上，Dubbo框架中都是使用在接口上。它的主要作用就是标记这个接口是一个Dubbo SPI接口，即是一个扩展点，可以有多个不同的内置或用户
定义的实现。运行时需要通过配置找到具体的实现类。

Dubbo中很多地方通过getExtension (Class<T> type. String name)来获取扩展点接口的
具体实现，此时会对传入的Class做校验，判断是否是接口，以及是否有@SPI注解，两者缺一
不可。

> @Adaptive

@Adaptive注解可以标记在类、接口、枚举类和方法上，但是在整个Dubbo框架中，只有
几个地方使用在类级别上，如AdaptiveExtensionFactory和AdaptiveCompiler,其余都标注在
方法上。

如果标注在接口的方法上，即方法级别注解，则可以通过参数动态获得实现类，这一
点已经在自适应特性上说明。方法级别注解在第一次getExtension时，会自动生成
和编译一个动态的Adaptive类，从而达到动态实现类的效果。

 > @Activate
 
 @Activate可以标记在类、接口、枚举类和方法上。主要使用在有多个扩展点实现、需要根
据不同条件被激活的场景中，如Filter需要多个同时激活，因为每个Filter实现的是不同的功能。
©Activate可传入的参数很多。

### ExtensionLoader工作原理

ExtensionLoader是整个扩展机制的主要逻辑类，在这个类里面卖现了配置的加载、扩展类
缓存、自适应对象生成等所有工作。

ExtensionLoader 的逻辑入口可以分为 getExtension、getAdaptiveExtension、
getActivateExtension三个，分别是获取普通扩展类、获取自适应扩展类、获取自动激活的扩
展类。总体逻辑都是从调用这三个方法开始的，每个方法可能会有不同的重载的方法，根据不
同的传入参数进行调整。

三个入口中,getActivateExtension对getExtension 的依赖比较getAdaptiveExtension
则相对独立。

## Dubbo启停原理解析

- Dubbo配置解析原理
- Dubbo服务暴露原理
- Dubbo服务消费原理
- Dubbo优雅停机解析

### Dubbo配置解析

目前Dubbo框架提供了3中配置方式：XML配置、注解、属性文件（properties和yml）。

对于Dubbo的配置，不管是注解还是XML配置、properties配置都是需要对象来承载配置内容的。

> 基于XML配置原理解析

对于XML配置原理解析，在之前学习Spring源码时，就已经有来了解到，一般都是通过XXNamespaceHandler来进行处理的，而在Dubbo中是通过DubboNamespaceHandler来完成。

```
public class DubboNamespaceHandler extends NamespaceHandlerSupport implements ConfigurableSourceBeanMetadataElement {

    static {
        Version.checkDuplicate(DubboNamespaceHandler.class);
    }

    @Override
    public void init() {
        registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("config-center", new DubboBeanDefinitionParser(ConfigCenterBean.class, true));
        registerBeanDefinitionParser("metadata-report", new DubboBeanDefinitionParser(MetadataReportConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("metrics", new DubboBeanDefinitionParser(MetricsConfig.class, true));
        registerBeanDefinitionParser("ssl", new DubboBeanDefinitionParser(SslConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
    }
}    
```

DubboNamespaceHandler主要把不同的标签关联至U解析实现类中o registerBeanDefinitionParser方法约定了在Dubbo框架中遇到标签application> module和registry等都会委托给
DubboBeanDefinitionParser处理。需要注意的是，在新版本中重写了注解实现，主要解决了以前实现的很多缺陷（比如无法处理AOP等）。

来看下DubboBeanDefinitionParser类的parse方法核心逻辑，部分内容省略，只贴出核心逻辑：
```
        // 初始化 RootBeanDefinition
        RootBeanDefinition beanDefinition = new RootBeanDefinition();
        beanDefinition.setBeanClass(beanClass);
        beanDefinition.setLazyInit(false);
        // 获取beanId
        String id = resolveAttribute(element, "id", parserContext);
        // 如果没有beanId则将beanName设置为beanId，确保Spring容器没有重复的Bean定义
        if (StringUtils.isEmpty(id) && required) {
            // 一次尝试获取XML配置标签name和interface作为Bean唯一id
            String generatedBeanName = resolveAttribute(element, "name", parserContext);
            if (StringUtils.isEmpty(generatedBeanName)) {
                // 如果协议标签没有指定name，则使用默认name：dubbo
                if (ProtocolConfig.class.equals(beanClass)) {
                    generatedBeanName = "dubbo";
                } else {
                    generatedBeanName = resolveAttribute(element, "interface", parserContext);
                }
            }
            // 如果beanName也为空，则用beanClass作为beanId
            if (StringUtils.isEmpty(generatedBeanName)) {
                generatedBeanName = beanClass.getName();
            }
            id = generatedBeanName;
            int counter = 2;
            while (parserContext.getRegistry().containsBeanDefinition(id)) {
                id = generatedBeanName + (counter++);
            }
        }
        // Step3 将获取到的Bean注册到Spring
        if (StringUtils.isNotEmpty(id)) {
            // BeanId 出现重复则抛异常
            if (parserContext.getRegistry().containsBeanDefinition(id)) {
                throw new IllegalStateException("Duplicate spring bean id " + id);
            }
            // 将xml转换为的Bean注册到Spring的parserContext，后续属性通过adPropertyValue来增添
            parserContext.getRegistry().registerBeanDefinition(id, beanDefinition);
            beanDefinition.getPropertyValues().addPropertyValue("id", id);
        }
```

小结：首先DubboBeanDefinition#Parse方法前半部分逻辑就是负责把标签解析成对应的BeanDefinition定义，并注册到Spring上下文中，同时保证了Spring容器中相同的id的Bean不会被覆盖。

```
        if (ProtocolConfig.class.equals(beanClass)) {
            // 如果dubbo标签中配置了protocol协议，则添加protocol属性
            for (String name : parserContext.getRegistry().getBeanDefinitionNames()) {
                BeanDefinition definition = parserContext.getRegistry().getBeanDefinition(name);
                PropertyValue property = definition.getPropertyValues().getPropertyValue("protocol");
                if (property != null) {
                    Object value = property.getValue();
                    if (value instanceof ProtocolConfig && id.equals(((ProtocolConfig) value).getName())) {
                        definition.getPropertyValues().addPropertyValue("protocol", new RuntimeBeanReference(id));
                    }
                }
            }
        } else if (ServiceBean.class.equals(beanClass)) {
            // 如果<dubbo:service> 配置了class属性，那么为具体class配置的类注册Bean，并注入ref属性。
            String className = resolveAttribute(element, "class", parserContext);
            if (StringUtils.isNotEmpty(className)) {
                RootBeanDefinition classDefinition = new RootBeanDefinition();
                // 通过类反射工具获取className的实例
                classDefinition.setBeanClass(ReflectUtils.forName(className));
                classDefinition.setLazyInit(false);
                // parseProperties主要是解析<dubbo:service>标签中的name、class、ref属性并通过key-value键值对取出来，放到BeanDefinition中
                // 因此ServiceBean就会包含了用户配置的属性值
                parseProperties(element.getChildNodes(), classDefinition, parserContext);
                beanDefinition.getPropertyValues().addPropertyValue("ref", new BeanDefinitionHolder(classDefinition, id + "Impl"));
            }
        } else if (ProviderConfig.class.equals(beanClass)) {
            parseNested(element, parserContext, ServiceBean.class, true, "service", "provider", id, beanDefinition);
        } else if (ConsumerConfig.class.equals(beanClass)) {
            parseNested(element, parserContext, ReferenceBean.class, false, "reference", "consumer", id, beanDefinition);
        }
```

接下来对于<dubbo:provider>和<dubbo:consumer>标签的解析，通过了parseNested方法来处理，即解析嵌套标签。因为provider和consumer标签可能会在内部嵌套代码，即、
即<dubbo:provider>内部可能会嵌套<dubbo:consumer>标签。,解析内部的service并生成Bean的时候，会把外层provider实例对象注入service,这种设计方式允许内部标签直接获取外部标签属性。


那么标签的attribute是如何提取的呢？对于attribute属性，主要分为两种场景：
- 查找配置对象的get、set和is前缀方法，如果标签属性名和方法名称相同，则通过反
射调用存储标签对应值。
- 如果没有和get、set和is前缀方法匹配，则当作parameters参数存储，parameters
是一个Map对象。

小结：以上两种场景的值最终都会存储到Dubbo框架的URL中，唯一区别就是get、set和is前缀方法当作普通属性存储,parameters是用Map字段存储的

> 基于注解配置原理解析

Dubbo重启开源后，对Dubbo的注解进行了重写，重写后解决了一下几个问题：

- 注解支持不充分，需要XML配置<dubbo:annotation>
- @ServiceBean不支持SpringAOP
- @Reference不支持字段继承性

注解处理逻辑包含3部分内容
1. 如果用户使用了配置文件，则框架按需生成对应bean
2. 将所有使用Dubbo的@Service的class提升为bean存入SpringIOC中
3. 为使用@Reference注解的字段或方法注入代理对象

另外，看下Dubbo注解机制
1. @EnableDubbo激活注解
2. 通过DubboConfigConfigurationSelector来支持配置文件读取配置
3. 通过ServiceAnnotationBeanPostProcessor来提升@Service注解的服务为Spring bean
4. 通过ReferenceAnnotationBeanPostProcessor来注入@Reference引用

```
@EnableDubboConfig
@DubboComponentScan
public @interface EnableDubbo {
    ...
}
```

```
@Import(DubboConfigConfigurationRegistrar.class)
public @interface EnableDubboConfig {
    ...
}
```

```
@Import(DubboComponentScanRegistrar.class)
public @interface DubboComponentScan {
    ...
}
```

1. 由于DubboConfigConfigurationRegistrar实现了ImportBeanDefinitionRegistrar，所以会实现registerBeanDefinition()方法。
2. 在registerBeanDefinition()方法中，会注册一个DubboConfigConfiguration的一个注解，该注解中会有一个@EnableDubboConfigBindings注解
3. 然后将EnableDubboConfigBiding注解修饰的Bean注册到Spring容器中。
4. EnableDubboConfigBinding的作用是进行属性绑定，Dubbo会根据用户配置属性自动填充这些承载的对象。
5. 随后，DubboConfigConfigurationRegistrar的registerBeanDefinition方法后面会继续注册各种BeanPostProcessor，包括了ReferenceAnnotationBeanPostProcessor等。

> Dubbo服务注解扫描和注册

在Dubbo中，通过ServiceAnnotationBeanPostProcessor来进行服务注解扫描和注册, 该类的父类ServiceClassPostProcessor实现了服务注解扫描和注册的核心逻辑。
1. Dubbo框架首先会提取用户配置的扫描包名称，因为包名可能使用${...}占位符，因此框架会调用Spring的占位符解析做进一步解码
2. 开始真正的注解扫描，委托Spring对所有符合包名的.class文件做字节码分析，最终通过AnnotationTypeFilte(Service.class)配置扫描@Service注解作为过滤条件
3. 然后通过findServiceBeanDefinitionHolders来对扫描的服务创建 BeanDefinitionHolder,用于生成ServiceBean的RootBeanDefinition，用于Spring启动后的服务暴露

> Dubbo消费者注入

在Dubbo中，通过ReferenceAnnotationBeanPostProcessor来实现消费者注解注入，该类核心逻辑包含以下几步：
1. 查找Bean中所有通过@Reference修饰的字段或方法
2. 调用InjectionMetadata的inject来对字段、方法进行反射绑定

因为处理器 ReferenceAnnotationBeanPostProcessor 实现了 InstantiationAwareBeanPostProcessor接口，所以在Spring的Bean中初始化前会触发postProcessPropertyValues方法，该方法允许我们做进一步处理，比如增加属性和属性值修改等。

### 服务暴露

不管在服务暴露还是服务消费场景下，Dubbo框架都会根据优先级对配置信息做聚合处理,目前默认覆盖策略主要遵循以下几点规则：
1. -D 传递给 JVM 参数优先级最高，比如-Ddubbo. protocol.port=20880
2. 代码或XML配置优先级次高，比如Spring中XML文件指定<dubbo:protocol port='20880'/>
3. 配置文件优先级最低，比如 dubbo.properties 文件指定 dubbo.protocol.port=20880o一般推荐使用dubbo.properties作为默认值，只有XML没有配置时，dubbo.properties配置项才会生效，通常用于共享公共配置，比如应用名等

#### 远程服务暴露机制

在详细探讨服务暴露细节之前，我们先看一下整体RPC的暴露原理：
1. 服务转换成Invoker
    - ServiceConfig => ref
    - ProxyFactory => Javassist、JDK动态代理
    - Invoker => AbstractProxyInvoker
2. Invoker转化成Exporter
    - Protocol => Dubbo、injvm等
    - Exporter

在整体上看，Dubbo框架做服务暴露分为两大部分，第一步将持有的服务实例通过代理转换成Invoker,第二步会把Invoker通过具体的协议（比如Dubbo 转换成Exporter,框架做了
这层抽象也大大方便了功能扩展。

这里的Invoker可以简单理解成一个真实的服务对象实例，是Dubbo框架实体域，所有模型都会向它靠拢，可向它发起invoke调用。它可能是一个本地的实现，也可能是一个远程的实现，还可能是一个集群实现

首先，RPC暴露和服务暴露有啥区别？是同一个内容吗？

接下来我们深入探讨内部框架处理的细节，框架真正进行服务暴露的入口点在ServiceConfig#doExport中，无论XML还是注解，都会转换成ServiceBean,它继承自
ServiceConfig，在服务暴露前，会按照-D、XML、Properties覆盖属性。

Dubbo支持多注册中心同时写，如果配置了服务同时注册多个注册中心，则会在ServiceConfig#doExportUrls中依次暴露。

```
    private void doExportUrls() {
        ServiceRepository repository = ApplicationModel.getServiceRepository();
        ServiceDescriptor serviceDescriptor = repository.registerService(getInterfaceClass());
        repository.registerProvider(
                getUniqueServiceName(),
                ref,
                serviceDescriptor,
                this,
                serviceMetadata
        );
        // 加载注册中心地址
        List<URL> registryURLs = ConfigValidationUtils.loadRegistries(this, true);
        
        // 获取协议配置
        for (ProtocolConfig protocolConfig : protocols) {
            String pathKey = URL.buildKey(getContextPath(protocolConfig)
                    .map(p -> p + "/" + path)
                    .orElse(path), group, version);
            
            repository.registerService(pathKey, interfaceClass);
            
            serviceMetadata.setServiceKey(pathKey);
            // 如果服务指定暴露多个协议（Dubbo、REST），则依次暴露服务
            doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }
```

真实服务暴露逻辑是在doExportUrlsFor1Protocol方法中实现的。


总结下doExportUrlsFor1Protocol逻辑
1. 通过反射获取配置信息对象如application、module、protocolCOnfig到map中，用于后序构造URL
2. 然后根据map来新建URL对象
3. 如果没有配置任何信息，则调用exportLocal进行本地JVM服务暴露4. 如果配置了监控地址如dubbo-admin，则服务调用信息会上报最后用服。
5. 通过动态代理的方式创建Invoker对象，在服务端生成的是AbstractProxyInvoker实例对象，所有真实的方法调用都会委托给代理，
然后代理转发给服务ref调用
6. 通过动态代理转化成Invoker，registryURL存储的就是注册中心地址，然后使用export作为key追加服务元数据信息
6. 服务暴露后向注册中心注册服务信息
7. 如果之前判断没有注册中心，则直接暴露服务，不经过注册中心。因为这里暴露的URL信息是以具体RPC协议开头的，并不是以注册中心协议开头的

下面看下有注册中心服务的暴露和无注册中心服务的暴露URL：

- registry://host:port/com.alibaba.dubbo.registry.RegistryService?protocol=zookeeper&export=dubbo://ip:port/xxx?
- dubbo://ip:host/xxx.Service?timeout=1000&

protocol实例会自动根据服务暴露URL自动做适配，有注册中心场景会取出具体协议，比如zookeeper协议，首先会创建注册中心实例，然后取出export对于的具体服务URL，
URL对应的协议(默认为Dubbo)进行服务暴露，当服务暴露成功后把服务数据注册到ZooKeeper。

如果没有注册中心，则在⑦中会自动判断URL对应的协议(Dubbo)并直接暴露服务，从而没有经过注册中心。

在将服务实例ref转化成Invoker之后，如果有注册中心，则会通过RegistryProtocol#export进行更细粒度的控制，比如先进行服务暴露在向注册中心进行注册服务元数据。

在注册中心在做服务暴露时，会做以下几件事情：
1. 委托具体协议(Dubbo)进行服务暴露，创建NettyServer监听端口和保存服务实例
2. 创建注册中心对象，与注册中心创建TCP连接
3. 注册服务元数据到注册中心
4. 订阅configurators节点，监听服务动态属性变更事件
5. 服务销毁收尾工作，比如关闭端口、反注册服务信息等


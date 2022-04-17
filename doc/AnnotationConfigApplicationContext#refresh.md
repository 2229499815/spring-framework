### prepareFresh()
+ 记录开始事件
+ 设置close状态为false  此状态用于发布容器关闭事件
+ 设置active状态为true  此状态用判断容器是否激活状态用于后续某些对容器有依赖得其它模块得启动流程
+ initPropertySource()  new 一个StanderEnvironment 对象给到当前属性environment，并且判断是否是web容器
如果是继续则装载servlet的相关配置,servletContext/servletConfig
+ 进行必要属性校验，如果某个属性在你的系统中必须要有，就可以这样设置给容器去判断ctx.getEnvironment().setRequiredProperties("foo");
+ 注册在refresh之前加载的ApplicationListener,并初始化earlyApplicationEvents 事件集合
准备阶段结束
### obtainFreshBeanFactory()
+ 将BeanFactory标记为已刷新并设置serialId
+ 直接返回beanFactory,此时的beanFactory已于scan阶段完成基础配置,包括第一轮的BeanDefinition装载
结束
### prepareBeanFactory()
+ 给beanFactory设置ClassLoader，默认使用线程的ClassLoader,此ClassLoader仅仅被用于BeanDefinition的解析
+ 设置SPEL表达式解析器beanExpressionResolver 支持解析#{...}
+ 配置PropertySource的加载器和解析器
+ 新增BeanPostProcessor，ApplicationContextAwareProcessor用于给实现了aware接口的Bean设置对应属性，并实例化一个
@Vaule注解解析器给到EmbeddedValueResolverAware接口的bean
+ 忽略某些接口的setter注入,用于告诉注入阶段，实现了这些接口的setter方法不需要你注入，我已经统一处理过了。
用于确保，应用程序实现对应接口获取到的对象是容器的内置唯一对象，避免出现乱七八糟的问题
    + beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    + beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    + beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    + beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    + beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    + beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
    + beanFactory.ignoreDependencyInterface(ApplicationStartupAware.class);
+ 添加内部组件对应用层进行注入的支持，即让你可以直接在其它bean中进行注入对应的属性
    + beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    + beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    + beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    + beanFactory.registerResolvableDependency(ApplicationContext.class, this);
+ 新增BeanPostProcessor，ApplicationListenerDetector，在初始化后将applicationListener添加到ApplicationContext中
同时添加到ApplicationEventMulticaster事件广播器中，销毁前从事件广播器中移除。
+ 判断是否有aspectj weaver 支持，如果有添加对应的LoadTimeWeaverAwareProcessor用于处理加载期织入，如javaagent
+ 注册一些内部组件到单例池
    + beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    + beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    + beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    + beanFactory.registerSingleton(APPLICATION_STARTUP_BEAN_NAME, getApplicationStartup());
BeanFactory准备完成
### postProcessBeanFactory 
留给子容器扩展，例如：webApplicationContext 可能需要额外加载或者配置一些内容

### invokeBeanFactoryPostProcessors(beanFactory)
委托给org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors
调用BeanFactoryPostProcessors即在整个BeanFactory准备完成后执行一些修改和增强的动作(包括@Bean @Import等等注解的解析)
invokeBeanFactoryPostProcessors这个方法内只要搞清楚一见事情即可，就是具体的PostProcessor的执行顺序。
第一步先调用BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry方法顺序如下
+ 先执行手动放进来的类型为BeanDefinitionRegistryPostProcessor,多余的分拣存到regularPostProcessor，即通过ctx.addBeanFactoryPostProcessor
+ 执行spring扫描完成后放进来的类型为BeanDefinitionRegistryPostProcessor，并实现了PriorityOrdered接口的
这里有两部分:
    + 1、先执行扫描进来的
    + 2、再执行内置的只有一个为ConfigurationClassPostProcessor（做了很多分拣动作后续介绍)
    
+ 执行spring扫描后放进来的类型为BeanDefinitionRegistryPostProcessor，并实现了Ordered接口的。
+ 最后执行类型为BeanDefinitionRegistryPostProcessor，执行过的就被记录到容器processedBeans中
这里的循环为了处理，执行BeanDefinitionRegistryPostProcessor之后产生的新的BeanDefinitionRegistryPostProcessor

执行的顺序就是ctx.add-->实现PriorityOrdered-->实现Ordered-->other 很容易可以看出来这个代码实际是有问题的
因为BeanDefinitionRegistryPostProcessor 会产生新的BeanDefinitionRegistryPostProcessor，如果再执行ordered阶段产生的实现PriorityOrdered的
BeanDefinitionRegistryPostProcessor,会被放到other阶段去执行了，所以这里并无法完全保障这个顺序执行。所以我们知道这个问题，避免这么用就行了。
最后调用BeanFactoryPostProcessor的postProcessBeanFactory方法，也就是前面产生的所有的BeanFactoryPostProcessor执行一遍
执行顺序与前面顺序一样，执行完之后会产生新的BeanFactoryPostProcessor
所以下面就进入BeanFactoryPostProcessor 的处理流程

+ 将priorityOrdered、Ordered、noOrdered三个分开存放
+ 执行实现priorityOrdered的BeanFactoryPostProcessors
+ 执行实现Ordered的BeanFactoryPostProcessors
+ 执行noOrdered的BeanFactoryPostProcessors
+ 最后清理缓存的BeanDefinition，因为有可能被BeanFactory后置处理器修改过了

回到委托前，判断是否启用LoadTimeWeave，如果启用就添加LoadTimeWeaverAwareProcessor。
到此invokeBeanFactoryPostProcessors结束

### registerBeanPostProcessors
这个阶段是注册BeanPostProcessors前面一个阶段是执行 BeanFactory的PostProcessor

+ 首先获取所有BeanPostProcessors的name
+ 在添加一个检查器检查是否所有的bean都执行过了所有的BeanPostProcessors，如果没有则抛一些警告信息这边使用INFO
+ 分拣不同的类型PriorityOrdered先执行getBean如果是MergedBeanDefinitionPostProcessor再备份到internalPostProcessors
其它的Ordered的以及普通的noOrdered的先放到放到不同的临时容器，升序排列并注册到BeanFactory中
+ 处理实现Ordered接口的BeanPostProcessors流程同上，获取实例，放到orderedPostProcessors 如果是MergedBeanDefinitionPostProcessor
再备份到internalPostProcessors升序排列并注册
+ 处理noOrdered的BeanPostProcessors，流程同上，获取实例放到noOrderedPostProcessors中，如果是MergedBeanDefinitionPostProcessor
再备份到internalPostProcessors直接注册，noOrder所以无序排序
+ 处理internalPostProcessors 升序排列并注册
+ 最后再new ApplicationListenerDetector(applicationContext)注册到BeanFactory中（这也是一个BeanPostProcessor）
于是形成的顺序就是
```mermaid
graph TB
priorityOrdered（BeanPostProcessors）-->ordered(BeanPostProcessors)
ordered(BeanPostProcessors)-->noOrdered(BeanPostProcessors)
noOrdered(BeanPostProcessors)-->priorityOrdered(MergedBeanDefinitionPostProcessors)-->Ordered(MergedBeanDefinitionPostProcessors)
Ordered(MergedBeanDefinitionPostProcessors)-->noOrdered(MergedBeanDefinitionPostProcessors)
noOrdered(MergedBeanDefinitionPostProcessors)-->ApplicationListenerDetector(BeanPostProcessor)
```
所以整个容器最先实例的化的Bean是什么？就是BeanPostProcessor
### initMessageSource()
国际化相关的组件初始化
+ 判断是否包含名称为messageSource 的bean
+ 如果还包含父容器，继续设置当前messageSource的父容器的MessageSource

+ 如果当前容器没有名称为messageSource的bean 就new一个注册到BeanFactory中并设置parent，加入单例池

### initApplicationEventMulticaster()
初始化容器事件广播器
+ 判断容器中是否包含名称为applicationEventMulticaster 的bean 如果包含，直接getBean并设置到applicationEventMulticaster中

+ 如果不包含则new一个SimpleApplicationEventMulticaster并设置到applicationEventMulticaster 并加入单例池

### onRefresh()
这里留给容器扩展，在web容器中初始化了一个多主题的设置
与上述流程一直判断是否包含themeSource名称的bean 有就get没有就new，然后对到单例池设置父容器

### registerListeners()
+ 遍历applicationListeners容器中所有listeners并加入事件广播器中
getApplicationEventMulticaster().addApplicationListener(listener);
+ 从BeanDefinitionRegistry中得到所有的ApplicationListener 并加入事件广播器中
+ 广播earlyApplicationEvents事件

### finishBeanFactoryInitialization(beanFactory);
完成BeanFactory的初始化工作
+ 配置类型转换器
+ 配置占位符解析器
+ 先实例化LoadTimeWeaverAware用于加载时AOP织入（这里可以不要管）
+ 置空tempClassLoader
+ 冻结配置，避免变更
+ 开始实例化所有剩余的非懒加载的Bean
beanFactory.preInstantiateSingletons();
至此refresh结束







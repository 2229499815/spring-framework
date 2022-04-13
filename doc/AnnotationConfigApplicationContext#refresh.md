### prepareFresh()
+ 记录开始事件
+ 设置close状态为false  此状态用于发布容器关闭事件
+ 设置active状态为true  此状态用判断容器是否激活状态用于后续某些对容器有依赖得其它模块得启动流程
+ initPropertySource()  new 一个StanderEnvironment 对象给到当前属性environment，并且判断是否是web容器
如果时继续则装载servlet的相关配置 
+ 进行必要属性校验，如果某个属性在你的系统中必须要有，就可以这样设置给容器去判断ctx.getEnvironment().setRequiredProperties("foo");
+ 注册在refresh之前加载的ApplicationListener,并初始化earlyApplicationEvents 事件集合
准备阶段结束
### obtainFreshBeanFactory()
+ 将BeanFactory标记为已刷新并设置serialId
+ 直接返回beanFactory,此时的beanFactory已于scan阶段完成基础配置
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
```java
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
    if (!beanFactory.containsLocalBean(APPLICATION_STARTUP_BEAN_NAME)) {
        beanFactory.registerSingleton(APPLICATION_STARTUP_BEAN_NAME, getApplicationStartup());
    }
```
BeanFactory准备完成
### postProcessBeanFactory 
留给子容器扩展，例如：webApplicationContext 可能需要额外加载或者配置一些内容

### invokeBeanFactoryPostProcessors(beanFactory)
调用BeanFactoryPostProcessors即在整个BeanFactory准备完成后执行一些修改和增强的动作



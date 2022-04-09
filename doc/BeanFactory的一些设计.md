```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;

public interface BeanFactory {
	String FACTORY_BEAN_PREFIX = "&";

	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
	boolean containsBean(String name);
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;
	String[] getAliases(String name);
}
```
我们单纯的从BeanFactory本身来看你会发现它提供了简单的只读方法，也就是说BeanFactory并没有提供相关的设置其中属性的方法。
接着我们来看它的层次结构,第一层我们并不能看出什么端倪来，来到第二层我们就可以发现,每一层下面都被分为两个子类
+ 一个是读:你看到的所有非Configurable开头的类都是只读功能
+ 一个是写:即你看到Configurable 开头的所有的类都是用来对BeanFactory进行配置的
spring在这里对整个BeanFactory做了一层分离的设计，使框架变得高度解耦，并且职责明确，扩展性也更好。
让做框架的人专注与做框架，写业务的人专注与写业务。
```text
HierarchicalBeanFactory (org.springframework.beans.factory)
    ConfigurableBeanFactory (org.springframework.beans.factory.config)
        AbstractBeanFactory (org.springframework.beans.factory.support)
        ConfigurableListableBeanFactory (org.springframework.beans.factory.config)
    ApplicationContext (org.springframework.context)
        ConfigurableApplicationContext (org.springframework.context)
        WebApplicationContext (org.springframework.web.context)
SimpleJndiBeanFactory (org.springframework.jndi.support)
AutowireCapableBeanFactory (org.springframework.beans.factory.config)
    ConfigurableListableBeanFactory (org.springframework.beans.factory.config)
        DefaultListableBeanFactory (org.springframework.beans.factory.support)
    AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
        DefaultListableBeanFactory (org.springframework.beans.factory.support)
    StubBeanFactory in StubWebApplicationContext (org.springframework.test.web.servlet.setup)
ListableBeanFactory (org.springframework.beans.factory)
    ApplicationContext (org.springframework.context)
        ConfigurableApplicationContext (org.springframework.context)
        WebApplicationContext (org.springframework.web.context)
    StaticListableBeanFactory (org.springframework.beans.factory.support)
        StubBeanFactory in StubWebApplicationContext (org.springframework.test.web.servlet.setup)
    ConfigurableListableBeanFactory (org.springframework.beans.factory.config)
        DefaultListableBeanFactory (org.springframework.beans.factory.support)
```
对于理清楚这个关系对于我们阅读源码也有很大的好处，具体什么好处，自行体会。
### 第二层
顶层的BeanFactory做的最主要的一个功能就是提供了各种getBean的方法，以及一些辅助判断的方法，这通常对于普通使用框架的人来说已经够用了
这就直接屏蔽了框架内部的复杂度。基于顶层的BeanFactory，做了层层继承或实现，进行了一系列的扩展。每个扩展的职责也很明确。
+ HierarchicalBeanFactory 分层次的BeanFactory，也就是父子容器的概念，子容器可以访问父容器的bean，父容器不能访问子容器的Bean
相当于对容器做了一定的隔离，具体目的我也不是很清楚。但是在springboot中已经抛弃了这个设计。
+ SimpleJndiBeanFactory JNDI相关的一些Bean，也就是用于获取通过jndi进行配置的一些bean，比如你如果把数据源的连接配置在tomcat中
就可以通过这种方法拿到到tomcat中的连接，作为spring的bean。但是现在这种方式也很少人使用了，所以也不用去管他
+ AutowireCapableBeanFactory <font color=red>重点</font>具有自动注入能力的BeanFactory，也就是用于解析Autowire注解的
+ ListableBeanFactory <font color=red>重点</font> 这个扩展了是为了获取整个bean的列表，比如获取所有的beanName
与父类的区别就是父类获取的是单个bean，而它获取的是所有。

### 第三层
这里我们只关注第二层中提到重点的两个类即可，即`AutowireCapableBeanFactory`，`ListableBeanFactory`.
#### AutowireCapableBeanFactory
+ ConfigurableListableBeanFactory (org.springframework.beans.factory.config)
对BeanFactory的配置继承了`ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory`
    + ConfigurableBeanFactory是从HierarchicalBeanFactory 继承下来的，它还而外继承了此几口了SingletonBeanRegistry
即拥有了一个新的技能单例Bean的注册容器
+ AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
一个模板方法的设计，定制化了AutowireCapableBeanFactory的一些具体逻辑。并留了一些扩展需要子类去实现。
+ StubBeanFactory in StubWebApplicationContext (org.springframework.test.web.servlet.setup)
对BeanFactory的一个存根，它没有实例化初始化，事件等等功能都没有，也不是我们所关心的内容。
#### ListableBeanFactory
+ ApplicationContext (org.springframework.context)
<font color=red>重点</font> 这个是context包下最核心的一个接口，它还是一个只读接口
继承了这些接口，都是见名知意，除了BeanFactory相关的，它还拥有了获取ENV的能力，国际化的能力，事件发布，以及解析资源的能力（含通配符）
    + EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,MessageSource, ApplicationEventPublisher, ResourcePatternResolver
也就是此时的已经是一个相对比较完整的功能框架勾勒好了。
+ StaticListableBeanFactory (org.springframework.beans.factory.support)
这个是spring当中最为简单的一个BeanFactory，它不具备自动注入能力，也不具备自动创建bean的能力，就是一个Map的封装。
需要你自己new一个对象，然后指定一个名字放进去。这个不是我们的主线，也不是spring的主线，我们不纠结于它。
+ ConfigurableListableBeanFactory (org.springframework.beans.factory.config)
<font color=red>重点</font> 与ApplicationContext相对应的配置BeanFactory的接口，也是AutowireCapableBeanFactory的子类。

### 第四层
在第三层中已经出现了一个抽线类AbstractAutowireCapableBeanFactory，这个已经对具体的实现做了一些定制
并且分了两个重要的接口出来ApplicationContext，ConfigurableListableBeanFactory
而在这一层开始就是对这些接口进行了具体的实现，往后的层次我也不在进行详细的一层一层分下去
#### ApplicationContext
+ ConfigurableApplicationContext (org.springframework.context)
      AbstractApplicationContext (org.springframework.context.support)
          GenericApplicationContext (org.springframework.context.support)
              GenericXmlApplicationContext (org.springframework.context.support)
              StaticApplicationContext (org.springframework.context.support)
              GenericWebApplicationContext (org.springframework.web.context.support)
              ResourceAdapterApplicationContext (org.springframework.jca.context)
              GenericGroovyApplicationContext (org.springframework.context.support)
              AnnotationConfigApplicationContext (org.springframework.context.annotation)
          AbstractRefreshableApplicationContext (org.springframework.context.support)
              AbstractRefreshableConfigApplicationContext (org.springframework.context.support)
      ConfigurableWebApplicationContext (org.springframework.web.context)
          GenericWebApplicationContext (org.springframework.web.context.support)
          StaticWebApplicationContext (org.springframework.web.context.support)
          AbstractRefreshableWebApplicationContext (org.springframework.web.context.support)
  WebApplicationContext (org.springframework.web.context)
      StubWebApplicationContext (org.springframework.test.web.servlet.setup)
      ConfigurableWebApplicationContext (org.springframework.web.context)
          GenericWebApplicationContext (org.springframework.web.context.support)
          StaticWebApplicationContext (org.springframework.web.context.support)
          AbstractRefreshableWebApplicationContext (org.springframework.web.context.support)
从整个结构树我们可以很容易的对整个ApplicationContext的扩展有一个较为全面的理解，首先顶层为两个子类分别为
 + ConfigurableApplicationContext
 + WebApplicationContext
 用于区分不通的上下文，接着各自的子类有进行了明确的分工，WebApplicationContext我们目前也不去关心它
 所以我们先看ConfigurableApplicationContext
 + AbstractApplicationContext 又是一个抽线了，又定制化了某些内容，然后又细分为
    + GenericApplicationContext                普通的ApplicationContext
    + AbstractRefreshableApplicationContext    可刷新的ApplicationContext也就是又给ApplicationContext添加了一个具备reload的功能
 然后底下就是一些不同的具体实现了。
#### ConfigurableListableBeanFactory
+ DefaultListableBeanFactory
    XmlBeanFactory (org.springframework.beans.factory.xml)
它继承和实现了这些接口
    + AbstractAutowireCapableBeanFactory implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable
此处又新增了一个新的功能BeanDefinitionRegistry，具备了BeanDefinition的容器，然后是实现父类的getBean的方法。
其子类提供了一个XmlBeanFactory，也就是通过xml形式加载BeanDefinition的具体实现。
最终会把这个类设置给ApplicationContext，达到两者的相结合。

上面的描述不过是听君一席话，如听一席话的话的感觉



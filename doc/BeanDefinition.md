#### BeanDefinition继承自AttributeAccessor, BeanMetadataElement两个顶层接口
AttributeAccessor 接口用于提供 BeanDefinition 的描述信息(而 BeanDefinition 是用于描述bean的信息)如用于描述 BeanDefinition 的 FULL 和 LITE 的属性就是设置在这里
 BeanMetadataElement 接口提供了一个 geSource() 方法用于跟踪该 BeanDefinition 的来源信息
   + AbstractBeanDefinition 实现了 BeanDefinition
     + RootBeanDefinition
     + ChildBeanDefinition
     + GenericBeanDefinition
        + ScannedGenericBeanDefinition  同时实现了 AnnotatedBeanDefinition
        + AnnotatedGenericBeanDefinition 同时实现了 AnnotatedBeanDefinition
   + AnnotateBeanDefinition 继承自 BeanDefinition
   + 内部类 ConfigurationClassBeanDefinition 定义在 ConfigurationClassBeanDefinitionReader 中继承至 RootBeanDefinition 实现了 AnnotatedBeanDefinition 
   + 内部类 ClassDerivedBeanDefinition 定义于 GenericApplicationContext 中
   
#### RootBeanDefinition
   + spring 刚刚启动时生成的几个默认的 BeanDefinition 都是 RootBeanDefinition
   + getMergeBeanDefinition 返回的都是 RootBeanDefinition ,即最终所有的 BeanDefinition 都会被转换成RootBeanDefinition
   + @Bean注解配置的 bean ，解析出来的 BeanDefinition 都是 RootBeanDefinition
#### ChildBeanDefinition
   + 当前版本已被废弃，spring 源码找不到对应代码，之所以保留目的是为了兼容某些第三方组件或者某些用户自定义扩展
#### GenericBeanDefinition
   + xml解析的BeanDefinition都是 GenericBeanDefinition (详见: BeanDefinitionReaderUtils.createBeanDefinition )
   + 注解配置(不包含 @Bean)的 BeanDefinition 都是 GenericBeanDefinition (详见其子类解析)
   + 替代 ChildBeanDefinition，按需设置 parent，而 ChildBeanDefinition 必须配置 parent。
   
#### 子类解析
+ ScannedGenericBeanDefinition
    + 通过扫描方式( @Component 系列)加载的 BeanDefinition 都是 ScannedGenericBeanDefinition
+ AnnotateGenericBeanDefinition
    + AnnotatedBeanDefinitionReader.register 的 BeanDefinition 都是 AnnotatedGenericBeanDefinition
    + 通过 @Import 注解导入的 BeanDefinition 都是 AnnotatedGenericBeanDefinition (详见 ConfigurationClassBeanDefinitionReader.registerBeanDefinitionForImportedConfigurationClass)
+ ConfigurationClassBeanDefinition
    + 通过 @Bean 注解配置的 BeanDefinition 都是 ConfigurationClassBeanDefintion
+ ClassDerivedBeanDefinition
    + 通过 lambda 回调方式创建对象的方式声明为 ClassDerivedBeanDefinition
    + 其它语言的一些扩展衍生的 BeanDefinition

#### 目的
spring定义了这么多类型的 BeanDefinition 目的是为了从不同维度解析 Bean,但是最终所有的 BeanDefinition 都要转换成 RootBeanDefinition 进行统一
把简单留给了别人，把复杂留给了自己
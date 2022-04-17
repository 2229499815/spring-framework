这个类至关重要，它涵盖了BeanFactory配置阶段的第二轮BeanDefinition扫描分拣处理的核心逻辑
实现如下接口
+ BeanDefinitionRegistryPostProcessor 
+ PriorityOrdered                           
+ ResourceLoaderAware                 
+ ApplicationStartupAware
+ BeanClassLoaderAware
+ EnvironmentAware

根据invokeBeanFactoryPostProcessors我们可以知道它的调用被分为两部分
+ 第一次postProcessBeanDefinitionRegistry 方法
+ 第二次postProcessBeanFactory 方法
这两个方法主要包含以下几个部分
+ 对配置类和非配置类的分分拣
+ 对@Configuration注解的类进行解析，注意第一轮扫描已经被扫描到BeanDefinition中，这次是对它进行解析
+ 对@Import注解的解析
+ 对@ImportResource注解的解析
+ 对@ComponentScan注解的解析
待补充
先来看processConfigBeanDefinitions
+ 先从BeanDefinitionRegistry中拿到当前所有BeanDefinition的名称列表
+ 通过名称遍历拿出对应的BeanDefinition从AttributeAccessor的描述信息中获取FULL和LITE的属性过滤已经处理过BeanDefinition
此处有个概念介绍下，BeanDefinition是用与描述Bean的，而AttributeAccessor存的信息是用于描述BeanDefinition的
checkConfigurationClassCandidate
+ if (className == null || beanDef.getFactoryMethodName() != null)  此判断未解释
+ 获取AnnotationMetadata,需要筛选掉的Bean
    + BeanFactoryPostProcessor
    + BeanPostProcessor
    + AopInfrastructureBean
    + EventListenerFactory
+ 获取@Configuration 注解的属性
+ config.get("proxyBeanMethods"))=true 则标记为FULL，这个是默认值，用于指定@Bean注解的方法是否需要生成代理，为了解决方法被直接调用的问题
这个配置颇有画蛇添足的感觉。还对metaspace产生额外的开销，完全没有必要，可以硬性规定使用传参的方式即可。
+ 判断如果是接口这不标记返回false
+ 判断是否Component、ComponentScan、import、importResource中的一种如果是判断是否包含@Bean如果有标记为Lite
+ 如果被标记了，最后获取Ordered值放入AttributeAccessor中
+ 把被标记的分拣到configCandidates中表示它是个配置类
+ 将配置类容器进行升序排列
+ 配置Bean Name生成器规则
+ 配置env
+ 创建ConfigurationClassParser 用于解析配置类
parser.parse(candidates);
+ 遍历所有的candidates
processConfigurationClass
+ 跳过不满足Conditional条件的Bean
+ 跳过@Import注解导入的配置类
+ 过滤出(className.startsWith("java.lang.annotation.") || className.startsWith("org.springframework.stereotype."))这两个包下的类
+ 然后执行doProcessConfigurationClass过程
doProcessConfigurationClass
+ @Component系列注解走processMemberClasses，处理嵌套类，然后递归到processConfigurationClass
+ @PropertySource注解和嵌套注解执行processPropertySource
+ @ComponentScan注解和嵌套注解执行执行ClassPathBeanDefinitionScanner 扫描与最早的扫描一样，执行扫描并注册到BeanDefinitionRegistry中
然后递归到parse方法，此前同样需要进行分拣。
+ 以上步骤执行完成后处理@Imported注解processImports
+ 然后处理@ImportResource注解
经历以上步骤将大部分的分拣工作处理完成

小结下：解析流程如下
+ @Component-->@PropertySource-->@ComponentScan-->@Imported-->@ImportResource-->@Bean

+ processMemberClasses-->processPropertySource
processPropertySource-->componentScanParser.parse
componentScanParser.parse-->processImports
processImports-->configClass.addImportedResource

### processMemberClasses

### processPropertySource

### componentScanParser.parse

### processImports

### configClass.addImportedResource







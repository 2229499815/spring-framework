AnnotationConfigApplicationContext实现了基于注解配置注册器可配置扫描和手动注册的能力，并继承至GenericApplicationContext被赋予了ApplicationContext的能力
在GenericApplicationContext中的空参构造new 了一个DefaultListableBeanFactory，拥有了配置的BeanFactory的能力
父类中实现了BeanDefinitionRegistry 有用了注册BeanDefinition的能力
也就是说这是一个完整的spring容器了
提供了4个构造函数
+ 自定义BeanFactory还是使用GenericApplicationContext空参构造中去new DefaultListableBeanFactory
+ 外部手动refresh还是通过内部自动refresh
+ 基于扫描的方式启动还是基于手动传入配置类的方式启动

不管在哪个构造中都实例化了两个属性reader，scanner
+ AnnotatedBeanDefinitionReader     基于注解的BeanDefinitionReader
用于将对应注解的类解析成BeanDefinition,在new的过程中做了以下初始化操作
    + 初始化配置了环境变量对象StanderEnvironment
    + 创建一个条件计算器，用于判断condition相关注解，是否被需要跳过。
        + 它需要从BeanFactory中获取是否存在bean，
        + 它需要从classLoader中获取是否存在对应的class
      	+ 它需要从resource中获取是否存在对应的resource
      	+ 它需要从environment中获取是否存在对应的属性
      	
    所以它的实现中就需要有对应的这些属性
    + 注册一些与基于注解相关的PostProcessor到BeanDefinitionRegistry中共注册了6个相关的BeanDefinition
        + 此处注册的BeanDefinition都是RootBeanDefinition
    	+ 1.ConfigurationClassPostProcessor         用于启动时处理@Configuration注解的类
    	+ 2.AutowiredAnnotationBeanPostProcessor    用于处理自动注入注解的
    	+ 3.CommonAnnotationBeanPostProcessor       jsr-250规范的一些注解
    	+ 4.PersistenceAnnotationBeanPostProcessor  对jpa注解的支持
    	+ 5.EventListenerMethodProcessor			事件监听器方法处理
    	+ 6.DefaultEventListenerFactory				默认的事件监听工厂类
    
    这六个类被放入了registry
+ ClassPathBeanDefinitionScanner    基于ClassPath的BeanDefinitionScanner
用于扫描指定包中的基于注解的类解析成BeanDefinition，new的时候同样传入了registry,并给定了一个属性useDefaultFilters=true
此时使用默认的包含过滤器用于鉴别哪些注解的类需要被扫描，默认过滤器放了以下几种类型
+  包含@Component以及它衍生的@Configuration、@Controller、@Repository注解
+ jsr-250的@ManagedBean
+ jsr-330的@Named
同时也调用了getOrCreateEnvironment用于获取一个幂等的StanderEnvironment的环境对象，前面reader中已经创建过了，所以此处返回一个引用
并将它设置给自己，然后接着设置了ResourceLoader，设置了三个属性
+ resourcePatternResolver   资源模式匹配解析器，用于解析路径如ClassPath:*，或者com/xxx
+ metadataReaderFactory     元数据Reader工厂,用于获取对应的元数据，如注解，使用ASM实现（为什么使用ASM）
+ componentsIndex           在spring5.0中新增的功能可以通过配置@Indexed注解生成索引列表进行快速加载，而不用进行扫描，
对应生成的文件META-INF/spring.components。

到这里准备的配置工作就已经完成了,自定义BeanFactory的构造我们先略过先来看自动refresh的两个构造
```text
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		this();
		register(componentClasses);
		refresh();
	}
```
```text
public AnnotationConfigApplicationContext(String... basePackages) {
		this();
		scan(basePackages);
		refresh();
	}
```
一个是调用了register逻辑，另一个调用了scan逻辑，这个很好理解，你给我指定了配置类数组，我直接根据你给的配置类去注册
如果你给我的是一个路径数组，那么我就去扫描对应的配置类。
此时我们先来看扫描的逻辑，等下回过头讲注册逻辑
ClassPathBeanDefinitionScanner#doScan，在doScan中首先调用父类的ClassPathScanningCandidateComponentProvider.findCandidateComponents方法
+ 首先判断是否包含索引组件，如果包含索引组件实例并且找到对应的Idexed注解，那么就使用索引的方式加载候选的Bean
+ 如果不满足条件就使用扫描的方式加载候选的Bean
扫描的方式：
首先将basePackage进行经典模式转换成路径模式（即点转斜杆）然后前面拼接上classpath*: 后面拼接上**/*.class
如果你传入的包是com.xxx,在这里就会被转换成classpath*:com/xxx/**/*.class,如果路径中包含${...}表达式，将会被解析
然后调用前面设置的PattenResourceResolver递归解析成resource数组，然后判断resource是否存在
如果存在就调用getMetadataReaderFactory().getMetadataReader(resource);获取相应的元数据信息(包含类信息、注解、属性、方法等)
然后根据元数据信息进行过滤，这边过滤是根据前面设置includeFilters，和excludeFilters 存的是TypeFilter
过滤出满足条件的类，然后new 一个 ScannedGenericBeanDefinition,见名知意使用扫描方式来的BeanDefinition
并将metadataReader传进去，从metadataReader中getAnnotationMetadata注解信息给到一个ScannedGenericBeanDefinition
并设置类名，以及BeanDefinition的元信息source。
接着判断这个BeanDefinition是否满足候选条件，
+ 是否独立闭合类（即不包含非静态内部类等）
+ 是否具体的类，（非抽象非接口）并且方法是否包含@Lookup注解
如果满足就添加到候选列表。如果不满足条件就忽略，最后返回候选列表
接着遍历列表，
+ 设置scope
+ 生成beanName
+ 如果是AbstractBeanDefinition那么为beanDefinition设置默认信息，肯定是因为继承来的
    + Lazy
    + AutoWireMode = AbstractBeanDefinition.AUTOWIRE_NO
    + dependencyCheck = AbstractBeanDefinition.DEPENDENCY_CHECK_NONE
    + initMethod
    + EnforceInitMethod=false
    + DestroyMethod
    + EnforceDestroyMethod=false
+ 如果是AnnotatedBeanDefinition那么设置补充信息,从注解中获取
    + @Lazy
    + @Primary
    + @DependsOn
    + @Role
    + @Description 
+ 在做一次检查
    + 判断是否与已存在的BeanDefinition兼容，即判断他们是否来源于同一个文件，或者名称相同

+ 检查通过则包装成BeanDefinitionHolder,并设置代理方式scopedProxyMode,默认为Default。
    + scopedProxyMode四种模式解析
        + Default 通常等于NO
        + NO    不代理
        + INTERFACES   使用jdk基于接口的代理
        + TARGET_CLASS 使用子类基于cglib的代理
        
+ 最后放入beanDefinitions列表中并调用registerBeanDefinition 方法注册beanDefinition

#### 注册流程
DefaultListableBeanFactory#registerBeanDefinition(beanDefinition,beanName)
+ 首先校验是否factoryMethod，如果是还存在复写的方法那么抛异常，否则判断是否有一个复写方法，如果有设置setOverloaded=false
+ 通过beanName判断beanDefinitionMap中是否已存在这个BeanDefinition
如果存在，判断是否允许覆盖
如果不允许覆盖抛异常
如果允许则覆盖则，判断是否是系统基础设施，
如果是基础设施则打印警告信息，判断当前beanDefinition对象是否和已存在的对象相等，
如果相等则打印debug信息，否则打印trace信息告诉你这个bean将被覆盖
然后判断这个BeanDefinition是否已经被实例化到1级缓冲中，如果已实例化，则移除并覆盖（这里是个递归）
+ 如果容器中不存在当前beanDefinition,判断是否处于bean创建阶段即alreadyCreated容器不为空
如果是在创建阶段，者做一个防并发对beanDefinition容器加锁
如果是处于启动注册阶段，把beanDefinition放入容器，把beanName 放入beanDefinitionNames列表中，并移除manualSingletonNames中对应的beanName
frozenBeanDefinitionNames  当整个BeanFactory初始化完成后会对整个BeanDefinition进行冻结，防止修改。如果后续在进来则需要清空该值才能重建
至此注册结束
至此扫描结束
接着注册前面说到的6个postProcessor用于准备后续的处理（幂等操作）
最后返回扫描到的数量












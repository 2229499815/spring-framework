### 缓存注解
+ @Cacheable 缓存方法返回值
+ @CacheConfig 类级别注解用于抽取当前类下的缓存配置公共属性例如`cacheNames`，方法上有相同属性时，方法有限级高
+ @CacheEvict  用于删除缓存数据
+ @CachePut    用于更新缓存操作，始终会执行方法逻辑，感觉此注解比较鸡肋，用它就需要注意方法写法返回值必须和缓存的方法返回值一致
+ @Caching     复杂缓存的实现，比如多维度`key`，当然也可以拆成简单的缓存
+ @EnableCaching 启用缓存注解
#### 注解详细介绍
##### 1、`@Cacheable`注解,缓存方法返回值，其参数如下
+ `String[] value() default {};`  与`cacheName`等同
+ `String[] cacheNames() default {};` 这个我们可以理解是`key`的组成部分最后在缓存中的`key`是`cacheNames::xxx`
+ `String key() default "";`  自定义`key`支持使用`SpEL`表达式可以获取参数，或者对象中的字段属性
+ `String keyGenerator() default "";` 指定使用自定义的`key`生成器，默认为`SimpleKey`
+ `String cacheManager() default "";`  顶层接口用于标准化，屏蔽不同组件差异，其实现有`RedisCacheManager`等,如果系统只配置一个缓存组件，不需要显示指定此参数
+ `String cacheResolver() default "";` 缓存解析器是用来管理缓存管理器的
+ `String condition() default "";` 方法执行前条件判断是否进行缓存(此处为缓存的条件)
+ `String unless() default "";`    方法执行后条件判断是否进行缓存(此处为不缓存的条件)，可以取到结果做判断，可以与`condition`组合使用:
   * `condition = false`不缓存
   * `condition = true, unless = true`不缓存
   * `condition = true, unless = false`缓存
+ `boolean sync() default false;`   是否同步，在高并发情况下，可能发生的情况如下，有`10`个线程同时访问同一个资源，这个时候并发获取拿不到缓存就可能发生`10`个线程全部去数据库获取取数据，如果设置`sync=true`，其中一个线程获取完数据之后缓存给其它线程使用，能有效避免缓存击穿问题。

在`springboot`中`@Cacheable`注解可以搭载`@EnableCaching`注解开启使用。
只需要引入依赖
```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```
并在配置文件中指定`redis`配置
```properties
spring.redis.port=6379
spring.redis.host=localhost
```
##### 2、@CacheConfig 抽取公共配置，内含四个属性，介绍同上，只不过他们是当前类级别的公共配置。
+ `String[] cacheNames() default {};` 
+ `String keyGenerator() default "";`
+ `String cacheManager() default "";`
+ `String cacheResolver() default "";`
##### 3、@CacheEvict 属性如下，已有属性同上
+ `String[] value() default {};`
+ `String[] cacheNames() default {};`
+ `String key() default "";`
+ `String keyGenerator() default "";`
+ `String cacheManager() default "";`
+ `String cacheResolver() default "";`
+ `String condition() default "";`
+ `boolean allEntries() default false;` 此属性默认为false，设置true，为清空当前域（cacheNames）下的所有缓存
+ `boolean beforeInvocation() default false;` 设置为true为了防止因删除缓存之后抛异常，导致缓存脏数据的产生，如我更新了一条数据，并提交了事务，但是后续执行抛了异常，导致沒有及时清理缓存，就出现脏数据，这里需要注意的是和事务相关的时候。

##### 4、@CachePut  缓存更新，当方法被注解此注解时，无论如何该方法逻辑都会执行。缓存始终用的是该方法的返回值。其属性如下
比Cacheable少了一个sync属性。
+ `String[] value() default {};`  与`cacheName`等同
+ `String[] cacheNames() default {};` 这个我们可以理解是`key`的组成部分最后在缓存中的`key`是`cacheNames::xxx`
+ `String key() default "";`  自定义`key`支持使用`SpEL`表达式可以获取参数，或者对象中的字段属性
+ `String keyGenerator() default "";` 指定使用自定义的`key`生成器，默认为`SimpleKey`
+ `String cacheManager() default "";`  顶层接口用于标准化，屏蔽不同组件差异，其实现有`RedisCacheManager`等,如果系统只配置一个缓存组件，不需要显示指定此参数
+ `String cacheResolver() default "";` 缓存解析器是用来管理缓存管理器的
+ `String condition() default "";` 方法执行前条件判断是否进行缓存(此处为缓存的条件)
+ `String unless() default "";`    方法执行后条件判断是否进行缓存(此处为不缓存的条件)，可以取到结果做判断，可以与`condition`组合使用:
   * `condition = false`不缓存
   * `condition = true, unless = true`不缓存
   * `condition = true, unless = false`缓存
##### 5、@Caching，此注解实际上是@CacheEvict、@CachePut、@Cacheable三个注解的组合注解，看属性就知道了，复杂操作时可以使用此注解例如
我查一个user对象可以是多维度的，并且是调用同一个接口，根据姓名，根据id，此时我可以在同一个缓存域下设置不同维度的缓存key，就可以使用这个注解达到效果。
+ `Cacheable[] cacheable() default {};`
+ `CachePut[] put() default {};`
+ `CacheEvict[] evict() default {};`

##### 6、@EnableCaching 此注解用于开启注解缓存，其属性如下
+ `boolean proxyTargetClass() default false;` 
+ `AdviceMode mode() default AdviceMode.PROXY;`
+ `int order() default Ordered.LOWEST_PRECEDENCE;`
此注解`@import(CachingConfigurationSelector.class)`此类,在`CachingConfigurationSelector`中导入以下两个类：
    * `AutoProxyRegistrar`
    * `ProxyCachingConfiguration`
请看代码
```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {
    private String[] getProxyImports() {
		List<String> result = new ArrayList<>(3);
		result.add(AutoProxyRegistrar.class.getName());
		result.add(ProxyCachingConfiguration.class.getName());
		if (jsr107Present && jcacheImplPresent) {
			result.add(PROXY_JCACHE_CONFIGURATION_CLASS);
		}
		return StringUtils.toStringArray(result);
	}
	//此处根据`mode`做了一层过滤下面说的内容前提是`mode=PROXY`
    @Override
    public String[] selectImports(AdviceMode adviceMode) {
        switch (adviceMode) {
            case PROXY:
                return getProxyImports();
            case ASPECTJ:
                return getAspectJImports();
            default:
                return null;
        }
    }
}
```
`AutoProxyRegistrar`处理了注解中的`proxyTargetClass`，`AdviceMode`这两个属性
```java
public class AutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

    //此处AUTO_PROXY_CREATOR(APC),如果mode属性为PROXY,使用基于接口的jdk原生代理，如果proxyTargetClass为true则强制使用CGLIB基于子类的代理
    //特别声明：此处配置是对会对spring所管理的所有的bean生效，所以请谨慎操作。
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		boolean candidateFound = false;
		Set<String> annoTypes = importingClassMetadata.getAnnotationTypes();
		for (String annoType : annoTypes) {
			AnnotationAttributes candidate = AnnotationConfigUtils.attributesFor(importingClassMetadata, annoType);
			if (candidate == null) {
				continue;
			}
			Object mode = candidate.get("mode");
			Object proxyTargetClass = candidate.get("proxyTargetClass");
			if (mode != null && proxyTargetClass != null && AdviceMode.class == mode.getClass() &&
					Boolean.class == proxyTargetClass.getClass()) {
				candidateFound = true;
				if (mode == AdviceMode.PROXY) {
					AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
					if ((Boolean) proxyTargetClass) {
						AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
						return;
					}
				}
			}
		}
		//...无关代码忽略
	}

}
```
#### 装配原理
`ProxyCachingConfiguration`对缓存进行了装配,详情请看代码`@Role`注解不需要管他这个注解对`bean`做了个分类而已，沒有实际作用，这里被标记为基础设施
此配置类配置了`3`个具体得`Bean`分别为
+ `BeanFactoryCacheOperationSourceAdvisor`
+ `CacheOperationSource` 
+ `CacheInterceptor`
```java
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class ProxyCachingConfiguration extends AbstractCachingConfiguration {

	@Bean(name = CacheManagementConfigUtils.CACHE_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryCacheOperationSourceAdvisor cacheAdvisor() {
		BeanFactoryCacheOperationSourceAdvisor advisor = new BeanFactoryCacheOperationSourceAdvisor();
		advisor.setCacheOperationSource(cacheOperationSource());
		advisor.setAdvice(cacheInterceptor());
		if (this.enableCaching != null) {
			advisor.setOrder(this.enableCaching.<Integer>getNumber("order"));
		}
		return advisor;
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public CacheOperationSource cacheOperationSource() {
		return new AnnotationCacheOperationSource();
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public CacheInterceptor cacheInterceptor() {
		CacheInterceptor interceptor = new CacheInterceptor();
		interceptor.configure(this.errorHandler, this.keyGenerator, this.cacheResolver, this.cacheManager);
		interceptor.setCacheOperationSource(cacheOperationSource());
		return interceptor;
	}

}
```
在`cacheInterceptor`这个bean中装配了以下实例
+ `this.errorHandler`
+ `this.keyGenerator`
+ `this.cacheResolver`
+ `this.cacheManager`
这些实例有父类来的，那么父类是如何装载这几个类呢？
请看如下代码在父类`AbstractCachingConfiguration`中自动注入了了一个`CachingConfigurer`实现类的集合，意味着我们要么去实现`CachingConfigurer`此接口要么去配置此接口的`bean`即可
```java
@Configuration(proxyBeanMethods = false)
public abstract class AbstractCachingConfiguration implements ImportAware {
//省略。。。
    @Autowired(required = false)
	void setConfigurers(Collection<CachingConfigurer> configurers) {
		if (CollectionUtils.isEmpty(configurers)) {
			return;
		}
		if (configurers.size() > 1) {
			throw new IllegalStateException(configurers.size() + " implementations of " +
					"CachingConfigurer were found when only 1 was expected. " +
					"Refactor the configuration such that CachingConfigurer is " +
					"implemented only once or not at all.");
		}
		CachingConfigurer configurer = configurers.iterator().next();
		useCachingConfigurer(configurer);
	}
    protected void useCachingConfigurer(CachingConfigurer config) {
		this.cacheManager = config::cacheManager;
		this.cacheResolver = config::cacheResolver;
		this.keyGenerator = config::keyGenerator;
		this.errorHandler = config::errorHandler;
	}
}
```
这几个如果沒有配置那么实际值为null那么请看如下代码,会创建默认值。`SimpleCacheErrorHandler`,`SimpleKeyGenerator`,`SimpleCacheResolver`
`SingletonSupplier`是`Supplier`的一个实现构造入参为两个`supplier`其中`get`方法返回了一个泛型实例，使用的时候调用`cacheResolver.get()`即可返回相关泛型实例，get方法中做了个判断，如果有传值那么使用传进来的实例，如果沒有传值则使用默认生成的值。

```java
public abstract class CacheAspectSupport extends AbstractCacheInvoker
		implements BeanFactoryAware, InitializingBean, SmartInitializingSingleton {
    public void configure(
			@Nullable Supplier<CacheErrorHandler> errorHandler, @Nullable Supplier<KeyGenerator> keyGenerator,
			@Nullable Supplier<CacheResolver> cacheResolver, @Nullable Supplier<CacheManager> cacheManager) {

		this.errorHandler = new SingletonSupplier<>(errorHandler, SimpleCacheErrorHandler::new);
		this.keyGenerator = new SingletonSupplier<>(keyGenerator, SimpleKeyGenerator::new);
		this.cacheResolver = new SingletonSupplier<>(cacheResolver,
				() -> SimpleCacheResolver.of(SupplierUtils.resolve(cacheManager)));
	}
}
```
以上是常规的`spring`项目配置需要扩展的东西以及配置

那么在`springboot`下就变得非常简单了，请看`CacheAutoConfiguration`类，此类为`springboot`体系下的所以在`spring-framework`项目下是找不到的
没关系我们扩展下,我们可以看到此类并沒有遵循`spring-cache`中标准进行扩展而是参考`springboot`自己的标准做了扩展，但是它神奇的联系上`spring-cache`的顶层体系。

```java
@Configuration
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
		HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })
@Import(CacheConfigurationImportSelector.class)
public class CacheAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public CacheManagerCustomizers cacheManagerCustomizers(
			ObjectProvider<CacheManagerCustomizer<?>> customizers) {
		return new CacheManagerCustomizers(
				customizers.orderedStream().collect(Collectors.toList()));
	}

	@Bean
	public CacheManagerValidator cacheAutoConfigurationValidator(
			CacheProperties cacheProperties, ObjectProvider<CacheManager> cacheManager) {
		return new CacheManagerValidator(cacheProperties, cacheManager);
	}

	@Configuration
	@ConditionalOnClass(LocalContainerEntityManagerFactoryBean.class)
	@ConditionalOnBean(AbstractEntityManagerFactoryBean.class)
	protected static class CacheManagerJpaDependencyConfiguration
			extends EntityManagerFactoryDependsOnPostProcessor {

		public CacheManagerJpaDependencyConfiguration() {
			super("cacheManager");
		}

	}

	/**
	 * Bean used to validate that a CacheManager exists and provide a more meaningful
	 * exception.
	 */
	static class CacheManagerValidator implements InitializingBean {

		private final CacheProperties cacheProperties;

		private final ObjectProvider<CacheManager> cacheManager;

		CacheManagerValidator(CacheProperties cacheProperties,
				ObjectProvider<CacheManager> cacheManager) {
			this.cacheProperties = cacheProperties;
			this.cacheManager = cacheManager;
		}

		@Override
		public void afterPropertiesSet() {
			Assert.notNull(this.cacheManager.getIfAvailable(),
					() -> "No cache manager could "
							+ "be auto-configured, check your configuration (caching "
							+ "type is '" + this.cacheProperties.getType() + "')");
		}

	}

	/**
	 * {@link ImportSelector} to add {@link CacheType} configuration classes.
	 */
	static class CacheConfigurationImportSelector implements ImportSelector {

		@Override
		public String[] selectImports(AnnotationMetadata importingClassMetadata) {
			CacheType[] types = CacheType.values();
			String[] imports = new String[types.length];
			for (int i = 0; i < types.length; i++) {
				imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
			}
			return imports;
		}

	}
}
```
`@Import(CacheConfigurationImportSelector.class)`导入当前类的一个静态内部类，实际是个`ImportSelector`此接口机制就是收集需要被`spring`加载的类，然后转成`beanDefinition`最后生成bean
此处加载了以下缓存配置类,可以看到`springboot`支持这么多的缓存体系并且都有相关实现。相对应的实现只需要引入相关`starter`即可，而注解还是使用上面那一套。
```java
final class CacheConfigurations {
    static {
		Map<CacheType, Class<?>> mappings = new EnumMap<>(CacheType.class);
		mappings.put(CacheType.GENERIC, GenericCacheConfiguration.class);
		mappings.put(CacheType.EHCACHE, EhCacheCacheConfiguration.class);
		mappings.put(CacheType.HAZELCAST, HazelcastCacheConfiguration.class);
		mappings.put(CacheType.INFINISPAN, InfinispanCacheConfiguration.class);
		mappings.put(CacheType.JCACHE, JCacheCacheConfiguration.class);
		mappings.put(CacheType.COUCHBASE, CouchbaseCacheConfiguration.class);
		mappings.put(CacheType.REDIS, RedisCacheConfiguration.class);
		mappings.put(CacheType.CAFFEINE, CaffeineCacheConfiguration.class);
		mappings.put(CacheType.SIMPLE, SimpleCacheConfiguration.class);
		mappings.put(CacheType.NONE, NoOpCacheConfiguration.class);
		MAPPINGS = Collections.unmodifiableMap(mappings);
	}
}
```
缓存的很多全局属性可以直接直接使用配置文件进行配置详情可以查看`CacheProperties`,以下为部分配置，我这里配置了redis
```properties
#通用缓存域
spring.cache.cache-names=cacheTest
#缓存类型，可以是以上列表的所有缓存
spring.cache.type=redis
#是否缓存null值，默认为true，为了防止缓存穿透
spring.cache.redis.cache-null-values=true
#可以设置key前缀，我此处家了个{}，为redis-cluster集群下的hash tag，可以将所有的节点打到同一个槽位上
spring.cache.redis.key-prefix={user}
#缓存过期时间10秒
spring.cache.redis.time-to-live=10000ms
#是否使用前缀
spring.cache.redis.use-key-prefix=true
```
构建用例如下，开头处引入的包以及配置的属性然后加如下代码
```java
@RestController
@EnableCaching
public class CacheTest {
    
    @PostMapping("/cacheTest")
    @Cacheable(cacheNames = "cacheTest")
    public String testCacheNames(String s) {
        System.out.println("沒有从缓存拿aaa");
        return s;
    }
    
    @GetMapping("delCache")
    @CacheEvict(cacheNames = "cacheTest",allEntries=true)
    public String testDel(){
        System.out.println("删除缓存");
        return "del";
    }
    
}
```
启动测试，当然你也可以构建单元测试。
请求接口你会得到如下输出
```bash
127.0.0.1:6379> keys *
1) "cacheTest::xxxxxx"
```
复杂的使用，自己动手试试吧。
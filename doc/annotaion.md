### beans相关注解
+ @Autowired 自动注入时使用
+ @Configurable 将Spring Bean非Spring管理的对象
+ @Lookup 单例引用多例bean时使用
+ @Qualifier 和@Autowired一起使用多个相同type的对象时指定一个name或者可以使用@Resource(name=""")注解
+ @Required 当属性是被必须依赖时在setter上设置此属性，则在配置时就必须指定属性，否则报错
+ @Value 属性注入
### cache相关注解
+ @Cacheable 缓存方法返回值
+ @CacheConfig 
+ @CacheEvict
+ @CachePut
+ @Caching
+ @EnableCaching
### context相关注解
+ @EnableSpringConfigured
+ @Bean
+ @ComponentScan
+ @ComponentScans
+ @Conditional
+ @Configuration
+ @DependsOn
+ @Description
+ @EnableAspectJAutoProxy
+ @EnableLoadTimeWeaving
+ @EnableMBeanExport
+ @Import
+ @ImportResource
+ @Lazy
+ @Primary
+ @Profile
+ @PropertySource
+ @PropertySources
+ @Role
+ @Scope
+ @EventListener
### format 相关注解
+ @DateTimeFormat
+ @NumberFormat
### jmx 相关注解
+ @ManagedAttribute
+ @ManagedMetric
+ @ManagedNotification
+ @ManagedNotifications
+ @ManagedOperation
+ @ManagedOperationParameter
+ @ManagedOperationParameters
+ @ManagedResource
### scheduling 相关注解
+ @Async
+ @EnableAsync
+ @EnableScheduling
+ @Scheduled
+ @Schedules
### stereotype 相关注解
+ @Component
+ @Controller
+ @Indexed
+ @Repository
+ @Service
### validation 相关注解
+ @Validated

### core 相关注解
+ @AliasFor
+ @Order
### lang 相关注解
+ @NonNull
+ @NonNullApi
+ @NonNullFields
+ @Nullable
+ @UsesJava7
+ @UsesJava8
+ @UsesSunHttpServer
+ @UsesSunMisc
### jms 相关注解
+ @EnableJms
+ @JmsListener
+ @JmsListeners
### messaging 相关注解
+ @DestinationVariable
+ @Header
+ @Headers
+ @MessageExceptionHandler
+ @MessageMapping
+ @Payload
+ @SendTo
+ @ConnectMapping
+ @SendToUser
+ @SubscribeMapping
### spring-test 相关注解
+ @Commit
+ @DirtiesContext
+ @IfProfileValue
+ @ProfileValueSourceConfiguration
+ @Repeat
+ @Rollback
+ @Timed
+ @ActiveProfiles
+ @BootstrapWith
+ @ContextConfiguration
+ @ContextHierarchy
+ @DynamicPropertySource
+ @AfterTestClass
+ @AfterTestExecution
+ @AfterTestMethod
+ @BeforeTestClass
+ @BeforeTestExecution
+ @BeforeTestMethod
+ @PrepareTestInstance
+ @RecordApplicationEvents
+ @Sql
+ @SqlConfig
+ @SqlGroup
+ @SqlMergeMode
+ @DisabledIf
+ @EnabledIf
+ @SpringJUnitConfig
+ @SpringJUnitWebConfig
+ @NestedTestConfiguration
+ @TestConstructor
+ @TestExecutionListeners
+ @TestPropertySource
+ @TestPropertySources
+ @AfterTransaction
+ @BeforeTransaction
+ @WebAppConfiguration
### transaction 相关注解
+ @EnableTransactionManagement
+ @Transactional
+ @TransactionalEventListener
### web 相关注解
+ @ControllerAdvice
+ @CookieValue
+ @CrossOrigin
+ @DeleteMapping
+ @ExceptionHandler
+ @GetMapping
+ @InitBinder
+ @Mapping
+ @MatrixVariable
+ @ModelAttribute
+ @PatchMapping
+ @PathVariable
+ @PostMapping
+ @PutMapping
+ @RequestAttribute
+ @RequestBody
+ @RequestHeader
+ @RequestMapping
+ @RequestParam
+ @RequestPart
+ @ResponseBody
+ @ResponseStatus
+ @RestController
+ @RestControllerAdvice
+ @SessionAttribute
+ @SessionAttributes
+ @ApplicationScope
+ @RequestScope
+ @SessionScope
+ @EnableWebFlux
+ @EnableWebMvc
+ @PathPatternsParameterizedTest
+ @EnableWebSocket
+ @EnableWebSocketMessageBroker
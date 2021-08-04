### @Configuration类中@autowired注入为null的问题

原因为spring在加载这个@Configuration类时，@autowired的类还没有加载到spring容器中，因次注入失败

正常情况下@Configuration类是在spring容器的注入之后，不应该发生这种情况，那是什么原因导致的呢？

在仔细进行对比查看后，发现一段日志

```
2021-08-03 17:15:48.698 [main] INFO  org.springframework.context.annotation.ConfigurationClassPostProcessor - Cannot enhance @Configuration bean definition 'multipleDataSourceConfiguration' since its singleton instance has been created too early. The typical cause is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor return type: Consider declaring such methods as 'static'.
```

发现 MapperScannerConfigurer 是 BeanFactoryPostProcessor 的实现。在 @Configuration 类中使用 BeanFactoryPostProcessor的实现类 会破坏该 @Configuration 类的默认后处理。

因此可在MapperScannerConfigurer方法声明中增加static

stackoverflow的解答

>There is a fundamental lifecycle conflict in handling BeanFactoryPostProcessor @Bean methods within @Configuration classes that use @Autowired, @PostConstruct, @Value, etc. Because BFPPs must be instantiated early in the lifecycle, they cause early instantiation of their declaring @Configuration class - too early to recieve the usual post-processing by AutowiredAnnotationBeanPostProcessor and friends. See https://jira.spring.io/browse/SPR-8269

在使用@Autowired、@PostConstruct、@Value 等的@Configuration 类中，处理BeanFactoryPostProcessor 的@Bean 方法时存在基本的生命周期冲突。因为BFPP 必须在生命周期的早期实例化，它们会导致声明@Configuration 类过早实例化 。 对于Autowired、AnnotationBean、PostProcessor 和其他注解来说，通常来说这样是为时过早的。见https://jira.spring.io/browse/SPR-8269
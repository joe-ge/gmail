# gmal电商项目
SpringBoot集成

1.devtool与dubbobu不兼容，反序列化的结果会出现类匹配异常。

官方文档：

2.mybatis-plus逆向生成的mapper中带泛型的方法与dubbo不兼容。

解决：在provider端自己的mapper中新增一个不带泛型的方法去调用继承父类的带泛型的方法。

3.mybatis-plus逆向生成的mapper中的分页必须添加一个拦截器

    /**
     * mybatis-plus分页插件
     * @return
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }

4.logback整合logstash：(1)导jar包;(2)编写配置文件

        <!-- logback 整合logstash的jar包-->
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>5.3</version>
        </dependency>


5.编写切面

[校验切面](gmall-admin-web/src/main/java/com/joe/gmall/admin/aop/DataValidAspect.java "校验切面") 

(1)导入切面场景

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>

(2)编写切面

	 1）、@Aspect
	 2）、切入点表达式
	 3）、通知
			前置通知：方法执行之前触发
			后置通知：方法执行之后触发
			返回通知：方法正常返回之后触发
			异常通知：方法出现异常触发

		正常执行：   前置通知==>返回通知==>后置通知
		异常执行：   前置通知==>异常通知==>后置通知

			环绕通知：4合1；拦截方法的执行

6.全局异常处理

[异常处理](gmall-admin-web/src/main/java/com/joe/gmall/admin/aop/GlobalExceptionHandler.java "异常处理")

7.SpEL表达式 CacheEnable key的拼接

前缀定义：
`public static final String CATEGORY_MENU_CACHE_KEY = "'sys_category_pid'+";`

key中拼接：`@Cacheable(cacheNames = SysCacheConstant.MENU_CACHE_NAME,key = SysCacheConstant.CATEGORY_MENU_CACHE_KEY + "#p0")`

8.自定义RedisCacheConfiguration

[RedisCacheConfig](gmall-pms/src/main/java/com/joe/gmall/pms/config/RedisCacheConfig.java)

    @Bean
    public RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties){

        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();

        config = config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
        config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        if (redisProperties.getTimeToLive() != null) {
            config = config.entryTtl(redisProperties.getTimeToLive());
        }
        if (redisProperties.getKeyPrefix() != null) {
            config = config.prefixKeysWith(redisProperties.getKeyPrefix());
        }
        if (!redisProperties.isCacheNullValues()) {
            config = config.disableCachingNullValues();
        }
        if (!redisProperties.isUseKeyPrefix()) {
            config = config.disableKeyPrefix();
        }
        return config;
    }

9.oss对象存储&阿里云跨域访问

10.dubbo针对某个服务或者某个方法单独配置超时时间和重试次数

注解中：（注意：用了注解之后，不管有没有在注解中自定义配置，下面其它的配置的都不生效）

    @Reference(version = "1.0" ,parameters = {
            "saveProduct.timeout","5000",
            "saveProduct.retries","0"
    })
    private ProductService productService;

properties中：

`dubbo.reference.com.joe.gmall.pms.service.saveProduct.saveProduct.timeout=5000`
`dubbo.reference.com.joe.gmall.pms.service.saveProduct.saveProduct.retries=0`

xml配置文件中，记得在启动类上添加@ImportResource(locations = "classpath:dubbo-consumer.xml")

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd ">
    <!--    <dubbo:application name="gmall-admin-web" />-->
    <!--    <dubbo:registry protocol="zookeeper" address="zookeeper://zk.registry.com:2181" />-->
    <!--    <dubbo:protocol name="dubbo" port="20895" />-->
        <dubbo:reference interface="com.joe.gmall.pms.service.saveProduct" id="productService">
            <dubbo:method name="saveProduct" timeout="2000" retries="0"></dubbo:method>
        </dubbo:reference>
    </beans>
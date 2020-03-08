### 1 SpringFactoriesLoader介绍

1. 框架内部使用的通用工厂加载机制
2. 从classpath下多个jar包特定的位置读取文件并初始化类
3. 文件内容必须是key-valye形式，即properties类型
4. key是全限定名、values是实现



### 2 源码分析

```java
@SpringBootApplication
public class SpringbootHelloworld1Application {

	public static void main(String[] args) {
        // 启动
		SpringApplication.run(SpringbootHelloworld1Application.class, args);
	}
}

public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}

public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    	// 在SpringApplication个构造器中加载
		return new SpringApplication(primarySources).run(args);
	}

	// SpringApplication的构造器
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
        
        // 这一步就是设置初始化器
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}

// 这个函数负责将要初始化的类放在一个collection中返回
	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
        
        // 获得所有的系统初始化器的全类名
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        // 实例化这些工厂
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
        // 排序，根据Order指定的顺序
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}

	// 获得所有的系统初始化器的全类名
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        // 查看缓存
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
            // 通过classLoader去获取要加载的类的路径
			Enumeration<URL> urls = (classLoader != null ?
                    // FACTORIES_RESOURCE_LOCATION就是"META-INF/spring.factories"
                    // 因此，如果我们要实现自定义的系统初始化器，可以将其全类名放在这个文件中
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
            // 每一条url代表一个spring.factories文件的位置
			while (urls.hasMoreElements()) {
                // 将这些spring。factories文件中的键值对读出来，然后将所有系统初始化器的类名分离出来
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```


### 1 SpringBoot自动配置

1. 首先，在我们的主类上有个注解

   ```java
   @SpringBootApplication
   public class Application {
   
   	public static void main(String[] args) {
   		SpringApplication.run(Application.class, args);
   	}
   	
   }
   ```

2. @SpringBootApplication

   ```java
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Inherited
   @SpringBootConfiguration
   @EnableAutoConfiguration // 这个开启自动配置
   @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
   		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
   public @interface SpringBootApplication {
       ......
   }
   ```

3. @EnableAutoConfiguration

   ```java
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Inherited
   @AutoConfigurationPackage
   @Import(AutoConfigurationImportSelector.class) // 这个是关键
   public @interface EnableAutoConfiguration {
       // ......
   }
   ```

4. AutoConfigurationImportSelector

   ```java
   public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
   		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
   
   // ......
   
       /*
       这个方法返回一个String[]，这个String[]中记录了一系列需要被导入容器的类的全类名
       */
   	@Override
   	public String[] selectImports(AnnotationMetadata annotationMetadata) {
   		if (!isEnabled(annotationMetadata)) {
   			return NO_IMPORTS;
   		}
   		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
   				.loadMetadata(this.beanClassLoader);
           // 主要就是从下面这一行代码扫描的
   		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
   				annotationMetadata);
   		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
   	}
   // ......
   }
   ```

5. getAutoConfigurationEntry

   ```java
   	protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
   			AnnotationMetadata annotationMetadata) {
   		if (!isEnabled(annotationMetadata)) {
   			return EMPTY_ENTRY;
   		}
   		AnnotationAttributes attributes = getAttributes(annotationMetadata);
           // 获取所有候选的配置类，这个就是从META-INF/spring.factories中读取所有的配置类
   		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
           // 去重
   		configurations = removeDuplicates(configurations);
           // 去除被排除的类
   		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
   		checkExcludedClasses(configurations, exclusions);
   		configurations.removeAll(exclusions);
   		configurations = filter(configurations, autoConfigurationMetadata);
   		fireAutoConfigurationImportEvents(configurations, exclusions);
           // 返回所有需要被配置的类
   		return new AutoConfigurationEntry(configurations, exclusions);
   	}
   ```

6. spring.factories文件如下

   ![image-20200325104224010](../images/SpringBoot%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%8E%9F%E7%90%86.assets/image-20200325104224010.png)

   这里面写了所有常用的可能被依赖的配置类。比如Redis、Mybatis等。

   它是在spring-boot-autoconfigure这个jar包下面。

   然后这些配置类中被依赖的配置类，就会被加载到beanFactory的beanDefinitionMap中。然后它们的配置会被加载。

### 2 如何实现一个starter

1. 首先，写一个configure配置工程，写好组件代码。将配置类的全类名写在META-INF/spring.factories中。就像spring-boot-autoconfigure一样，打成普通jar包。
2. 然后，写一个start工程，这个starter工程依赖于这个配置工程，打成普通jar包。
3. 最后，在需要使用到这个starter的地方，依赖这个starter。


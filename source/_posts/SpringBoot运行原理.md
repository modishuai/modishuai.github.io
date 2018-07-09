---
title: SpringBoot 运行原理
categories:
- Java
tags:
- Java
- SpringBoot
---

版本：Spring Boot 2.0 RELEASE
日期：2018/03/21


#@SpringBootApplication 源码
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    //do something
}
```
@SpringBootApplication 是一个组合注解，自动配置核心功能是由 @EnableAutoConfiguration 注解提供。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
	
	//do something
}
```

```java
@Import(AutoConfigurationImportSelector.class)
```
@Import 注解导入配置功能
AutoConfigurationImportSelector 这个类使用 SpringFactoriesLoader.loadFactoryNames 方法扫描具有META-INF/spring.factories 文件中的 jar 包。
在 spring-boot-autoconfigure2.0.0.RELEASE jar 中就存在spring.factories 文件。
spring.factories 声明了自动配置。
```java
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

spring.factories 内容如下
```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition
```

这些自动配置文件在 org.springframework.boot.autoconfig.condition 包下。
条件注解如下：
* @ConditionalOnBean    当容器有指定Bean的条件下。
* @ConditionalOnClass   当类路径下有指定类的条件下
* @ConditionalOnCloudPlatform   当项目在云平台下
* @ConditionalOnExpression  基于SpEL表达式作为判断条件
* @ConditionalOnJava    基于JVM版本作为条件
* @ConditionalOnJndi    在JNDI存在条件下查找指定位置
* @ConditionalOnMissingBean 当容器没有指定Bean的情况下
* @ConditionalOnMissingClass    当类路径下没有指定类的条件下    
* @ConditionalOnNotWebApplication   当前项目不是web项目的条件下
* @ConditionalOnProperty    指定的属性是否有指定的值
* @ConditionalOnResource    类路径下是否有指定的资源
* @ConditionalOnSingleCandidate 当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选的 Bean
* @ConditionalOnWebApplication  当前项目是web项目的情况下

以上注解都组合了@Conditional元注解，只不过使用了不同条件。


---
title: Spring Data Jpa 启动流程分析
date: 2021-03-03 11:03:37
tags:
- spring data jpa
- java
categories:
- spring data jpa
---


# Spring Data Jpa 启动流程分析

## EnableJpaRepositories

目的是导入JpaRepositoriesRegistrar这个配置类

```java

@Import(JpaRepositoriesRegistrar.class)
public @interface EnableJpaRepositories {

	String[] basePackages() default {};
```


> 通过JpaRepositoriesRegistrar的继承一步一步的往上走，发现最终实现了Spring 的ImportBeanDefinitionRegistrar接口

```java

class JpaRepositoriesRegistrar extends RepositoryBeanDefinitionRegistrarSupport {

protected Class<? extends Annotation> getAnnotation() {  //给定EnableJpaRepositories注解
return EnableJpaRepositories.class;
}
@Override 
protected RepositoryConfigurationExtension getExtension() {
    // 这个方法是通知给Spring 我这此次注册的是JpaRepositoryBeanFactory
return new JpaRepositoryConfigExtension();
}

}
```
> ImportBeanDefinitionRegistrar的作用就是自定义扫描文件将符合条件的注册成Bean


下面这个是扫描文件的实现方法，我截取了一些重要片段
```java
public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry registry) {

		AnnotationRepositoryConfigurationSource configurationSource = new AnnotationRepositoryConfigurationSource(
				annotationMetadata, getAnnotation(), resourceLoader, environment, registry);

		RepositoryConfigurationExtension extension = getExtension();
		RepositoryConfigurationUtils.exposeRegistration(extension, registry, configurationSource);

		RepositoryConfigurationDelegate delegate = new RepositoryConfigurationDelegate(configurationSource, resourceLoader,
				environment);
        // 看方法名字就知道 注册仓库
		delegate.registerRepositoriesIn(registry, extension);
	}

```
> 最重要的还是注册Bean，我们跟着registerRepositoriesIn方法进去

> 方法注释： 
> Registers the found repositories in the given **BeanDefinitionRegistry**.
> 注释说的很明显了，通过那个BeanDefinitionRegistry注册那些发现的仓库

```java
extension.registerBeansForRoot(registry, configurationSource);

		RepositoryBeanDefinitionBuilder builder = new RepositoryBeanDefinitionBuilder(registry, extension,
				configurationSource, resourceLoader, environment);
		List<BeanComponentDefinition> definitions = new ArrayList<>();

		StopWatch watch = new StopWatch();


		watch.start();


        //这里查了一下资料，configurations获取了所有的仓库接口
		Collection<RepositoryConfiguration<RepositoryConfigurationSource>> configurations = extension
				.getRepositoryConfigurations(configurationSource, resourceLoader, inMultiStoreMode);

		Map<String, RepositoryConfiguration<?>> configurationsByRepositoryName = new HashMap<>(configurations.size());

        // 下面应该是说通过读取各种情况下的配置文件去导入仓库列表
		for (RepositoryConfiguration<? extends RepositoryConfigurationSource> configuration : configurations) {

			configurationsByRepositoryName.put(configuration.getRepositoryInterface(), configuration);

			BeanDefinitionBuilder definitionBuilder = builder.build(configuration);

			extension.postProcess(definitionBuilder, configurationSource);

			if (isXml) {
				extension.postProcess(definitionBuilder, (XmlRepositoryConfigurationSource) configurationSource);
			} else {
				extension.postProcess(definitionBuilder, (AnnotationRepositoryConfigurationSource) configurationSource);
			}

			AbstractBeanDefinition beanDefinition = definitionBuilder.getBeanDefinition();
			String beanName = configurationSource.generateBeanName(beanDefinition);

		

			beanDefinition.setAttribute(FACTORY_BEAN_OBJECT_TYPE, configuration.getRepositoryInterface());

			registry.registerBeanDefinition(beanName, beanDefinition);
			definitions.add(new BeanComponentDefinition(beanDefinition, beanName));
		}

		potentiallyLazifyRepositories(configurationsByRepositoryName, registry, configurationSource.getBootstrapMode());

		watch.stop();

	

		return definitions;

```

> 上面的大概意思就是说，我通过 读取了所有用户仓库的接口，然后definitionBuilder用这个注册了一个JpaRepositoryFactoryBean
> 然后等spring初始化的时候就会创建JpaRepositoryFactoryBean这种类型的Bean了


> 通过查阅资料得知，spring 一开始就要初始化BeanFactory

## RepositoryBeanDefinitionRegistrarSupport
上面我们一步步的将我们的用户仓库注册到Spring中，那我们的的那些Crud操作是如何自动代理到我们自定义的接口中呢？


这就要重新到JpaRepositoriesRegistrar这个类来说了，他里面new了一个很重要的类JpaRepositoryConfigExtension

```java

@Override
	public String getRepositoryFactoryBeanClassName() {
		return JpaRepositoryFactoryBean.class.getName();
	}
```

> 我们可以看到这里获得了JpaRepositoryFactoryBean的名字，这个方法是用在一个DefaultRepositoryConfiguration的类里面的
> DefaultRepositoryConfiguration 的注释写着Default implementation of RepositoryConfiguration.
> 我们可以知道他其实是RepositoryConfiguration的实现


那么，这个RepositoryConfiguration我们又是在哪里用到的呢?
> 正式上面我们提到的registerBeansForRoot方法。

```java

Map<String, RepositoryConfiguration<?>> 	Map<String, RepositoryConfiguration<?>> configurationsByRepositoryName = new HashMap<>(configurations.size());


configurationsByRepositoryName.put(configuration.getRepositoryInterface(), configuration);
```

> 这个我们就很清楚了，我们通过registerBeansForRoot读取到用户仓库，然后通过上面的函数注册成JpaRepositoryFactoryBean，然后再托管到Spring容器里面。



那现在的重点就是 JpaRepositoryFactoryBean这个东西


## JpaRepositoryFactoryBean

经过查阅知道 afterPropertiesSet()这个函数是重点，他是创建仓库的入口。在哪里进行**动态代理**


下面是afterPropertiesSet()方法执行过程中用到的一个重要函数，意思就是生成仓库

```java
public <T> T getRepository(Class<T> repositoryInterface, RepositoryFragments fragments) {


		RepositoryMetadata metadata = getRepositoryMetadata(repositoryInterface);
		RepositoryComposition composition = getRepositoryComposition(metadata, fragments);
		RepositoryInformation information = getRepositoryInformation(metadata, composition);

		validate(information, composition);

		Object target = getTargetRepository(information);

		// Create proxy
		ProxyFactory result = new ProxyFactory();
		result.setTarget(target);
		result.setInterfaces(repositoryInterface, Repository.class, TransactionalProxy.class);

		if (MethodInvocationValidator.supports(repositoryInterface)) {
			result.addAdvice(new MethodInvocationValidator());
		}

		result.addAdvice(SurroundingTransactionDetectorMethodInterceptor.INSTANCE);
		result.addAdvisor(ExposeInvocationInterceptor.ADVISOR);

		postProcessors.forEach(processor -> processor.postProcess(result, information));

		result.addAdvice(new DefaultMethodInvokingMethodInterceptor());

		ProjectionFactory projectionFactory = getProjectionFactory(classLoader, beanFactory);
		result.addAdvice(new QueryExecutorMethodInterceptor(information, projectionFactory));

		composition = composition.append(RepositoryFragment.implemented(target));
		result.addAdvice(new ImplementationMethodExecutionInterceptor(composition));

		T repository = (T) result.getProxy(classLoader);


		return repository;
	}

```

上面的方法中也有注释，creat proxy。真相大白。
ProxyFactory result = new ProxyFactory();



## 不通过@EnableJpaRepositories注解，JPA如何自动配置


我发现我的spring boot 项目中即使不写@EnableJpaRepositories注解，照样可以启动成功。

经过查阅知道 spring boot 里面有一个 spring-boot-autoconfigure包，包里面有一个spring.factories配置文件，里面就有Jpa的相关配置，所以即使不加注解，照样可以成功。


[![6AJrsH.png](https://s3.ax1x.com/2021/03/03/6AJrsH.png)](https://imgtu.com/i/6AJrsH)


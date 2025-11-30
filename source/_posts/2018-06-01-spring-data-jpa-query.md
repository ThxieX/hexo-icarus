---
title: Spring Data JPA 源码分析 - 方法命名查询
thumbnail: /thumbnails/SpringData.png
date: 2018-06-01 00:00:00
categories:
  - Backend Engineering
tags:
  - Java
  - Spring
  - Spring-Data
  - 源码
excerpt: "深入分析 Spring Data JPA 方法命名查询的源码实现，解析如何通过方法名自动推导 SQL 查询的底层机制。"
aiSummary: "Spring Data JPA 的方法命名查询（Defining Query Method）是一项强大特性，开发者只需按照规范定义方法名（如 findByUsernameAndEmail），框架即可自动推导出 SQL 查询。文章深入源码分析了这一机制的实现原理：JPAQueryMethod 包装查询元信息、PartTree 解析方法名构成分解树（FromClause、WhereClause、SortClause）、RepositoryMetadata 获取实体信息，最后通过 JpaQueryFactory 构建最终查询。是理解 Spring Data JPA 内部工作原理的深度好文。"
---




### 前言

为什么要学习Spring Data JPA, 这里引用《Spring Data JPA实战》一书的一段话


> 为什么要重新学习”Spring Data JPA”？俗话说的好：”未来已经来临，只是尚未流行”，纵观市场上的 ORM 框架，MyBatis 以灵活著称，但是要维护复杂的配置，并且不是 Spring 官方的天然全家桶，还得做额外的配置工作，如果资深的架构师还得做很多封装；Hibernate 以 HQL 和关系映射著称，但是就是使用起来不是特别灵活；那么 Spring Data JPA 来了，感觉要夺取 ORM 的 JPA 霸主地位了，底层以 Hibernate 为封装，对外提供了超级灵活的使用接口，又非常符合面向对象和 Rest 的风格，感觉是架构师和开发者的福音，并且 Spring Data JPA 与 Spring Boot 配合起来使用具有天然的优势，你会发现越来越多的公司的招聘要用会有传统的 SSH、Spring、MyBatis 要求，逐步的变为 Spring Boot、Spring Cloud、Spring Data 等 Spring 全家桶的要求，而很多新生代的架构师基于其生态的考虑，正在逐步推动者 Spring Data JPA 的更多的使用场景。


<!-- more -->

### 方法命名查询 

方法命名查询: Defining Query Method

> 在最初, 我用到方法命名查询时觉得很酷, 这应该算是Spring Data JPA的一大特色了.

在Spring Data JPA中可以用创建规范的自定义方法进行查询

尽管这种方式是很方便的, 但是难免会遇到这样的情况:

- 方法名解析器不支持要使用的关键字
- 复杂糟糕的查询约束会导致方法名也变得很糟糕(例如: `findByXxxXxxXxInXxxOrderXxxXxx...`)

当然Specification, @Query能够解决这样的情况, 但并不是本节的重点. 



首先要知道, 在自定义方法名称中, 只需要在自己的实体仓储(Repository)上继承`Repository`接口即可.

当然, 如果想要有其它默认的通用方法实现, 可以有选择性的继承CrudRepository, PagingAndSortingRepository, JpaRepository等接口. 

以及JpaSpecificationExecutor, QueryByExampleExecutor, QuerydslPredicateExecutor 和自定义Repository, 都可以达到同样的效果.

并且在默认情况下, 通用的默认方法(例如: `findOne(), save()...`)无需我们实现, `SimpleJpaRepository`会对其提供实现.



#### Repository作用


为什么继承一个`Repository`接口就可以呢? 

首先`Repository`由Spring Data Common提供, 如果点进去可以发现它是一个空接口, 没有任何方法声明, 也就是常说的标记接口(Mark Interface).

JDK中提供了也存在这样的接口: 例如最常见的`Serializable`

而标记接口的作用就是当某个类去实现了标记接口, 就会认为这个类具有了标记的某种能力.



观察`Repository`的层次关系, 大致可以分出三条路线来: 

- `CrudRepository -> PagingAndSortingRepository -> JpaRepository -> SimpleJpaRepository`
- `ReactiveRepository `
- `RxJava2CrudRepository`

需要知道的是: `Repository`, `CrudRepository`, `PagingAndSortingRepository`都是Spring Data Common下的标准接口, 而到了`JpaRepository` 开始才进入了Spring Data JPA模块



#### Spring Data 公共模块

所以到这里也能大致看出Spring Data下模块划分的意义

Spring Data Common作为Spring Data所有模块的公用部分. 

如果我们使用JPA, 那它的实现就是Spring Data JPA下的SimpleJpaRepository.

如果是其他NoSQL实现, 例如redis, 那它的实现就是Spring Data Redis里的SimpleKeyValueRepository

提供Repository默认方法实现的源码并不会在这里分析, 因为它不是本节的重点.

> 在接口中定义规范的方法名就可以推导出查询来, 我们并不需要做任何实现, 那么Spring Data JPA是如何做到的呢?  



### 源码分析

#### Repository工厂类

我们在Java Config方式配置Spring Data JPA的@EnableJpaRepositories注解里可以发现这样一句:

`Class<?> repositoryFactoryBeanClass() default JpaRepositoryFactoryBean.class;`

配置中默认指定了Repository的工厂类 -> JpaRepositoryFactoryBean

来看下源码:

```java
public class JpaRepositoryFactoryBean<T extends Repository<S, ID>, S, ID extends Serializable> extends TransactionalRepositoryFactoryBeanSupport<T, S, ID> {
	//...
    private EntityManager entityManager;
    
    @Override
	public void afterPropertiesSet() {

		Assert.notNull(entityManager, "EntityManager must not be null!");
		super.afterPropertiesSet();
	}
    //...
}
```

通过这个类的命名可以猜到是个工厂Bean, 并且有看到Bean初始化后的回调处理`afterPropertiesSet()`

所以立刻点进它的父类继承到抽象类`RepositoryFactoryBeanSupport`中 

没错! 它实现InitializingBean接口和FactoryBean接口.

接下来就是寻找这个RepositoryFactoryBean是在哪里得到的呢. 

自然是从获取工厂实例的实现方法下手: 

`RepositoryFactoryBeanSupport.getObject()`: 

```java
public T getObject() {
    return initAndReturn();
}

public Class<? extends T> getObjectType() {
		return (Class<? extends T>) (null == repositoryInterface ? Repository.class : repositoryInterface);
}

/*
 * 这里是单例
 * @see org.springframework.beans.factory.FactoryBean#isSingleton()
 */
public boolean isSingleton() {
		return true;
}
```



`RepositoryFactoryBeanSupport.initAndReturn()`:

```java
/** 
  * Returns the previously initialized repository proxy or creates 
  * and returns the proxy if previously uninitialized.
  */
private T initAndReturn() {

    Assert.notNull(repositoryInterface, "Repository interface must not be null on initialization!");

    if (this.repository == null) {
        // 看这里
        this.repository = this.factory.getRepository(repositoryInterface, customImplementation);
    }

    return this.repository;
}
```

可以看到Repository Instance是在`this.factory`中得到,  那么再追入`RepositoryFactorySupport factory`.



#### RepositoryFactorySupport#factory的创建

不难找到, 其实上面已经提到过bean初始化回调方法`afterPropertiesSet()` , 就是在这里创建的`factory`

```java
public void afterPropertiesSet() {

		Assert.notNull(repositoryInterface, "Repository interface must not be null on initialization!");

    	// 看这
		this.factory = createRepositoryFactory();
		this.factory.setQueryLookupStrategyKey(queryLookupStrategyKey);
		this.factory.setNamedQueries(namedQueries);
		this.factory.setEvaluationContextProvider(evaluationContextProvider);
		this.factory.setRepositoryBaseClass(repositoryBaseClass);
		this.factory.setBeanClassLoader(classLoader);
		this.factory.setBeanFactory(beanFactory);

		this.repositoryMetadata = this.factory.getRepositoryMetadata(repositoryInterface);

		if (!lazyInit) {
			initAndReturn();
		}
	}
```

`createRepositoryFactory()`的实现是在TransactionalRepositoryFactoryBeanSupport中

```java
	@Override
	protected final RepositoryFactorySupport createRepositoryFactory() {

		RepositoryFactorySupport factory = doCreateRepositoryFactory();
		factory.addRepositoryProxyPostProcessor(exceptionPostProcessor);
		factory.addRepositoryProxyPostProcessor(txPostProcessor);

		return factory;
	}
```

再往下查看源码实际就是在JpaRepositoryFactoryBean中注入`entityManager`后 `return new JpaRepositoryFactory(entityManager);` 然后添加两个PostProcessor后返回factory

继续回到`RepositoryFactoryBeanSupport.initAndReturn()`

```java
this.repository = this.factory.getRepository(repositoryInterface, customImplementation);
```



####获取Repository实例代理

`getRepository()` 的实现就在JpaRepositoryFactory的父类RepositoryFactorySupport中:

```java
public <T> T getRepository(Class<T> repositoryInterface, Object customImplementation) {

		RepositoryMetadata metadata = getRepositoryMetadata(repositoryInterface);
		Class<?> customImplementationClass = null == customImplementation ? null : customImplementation.getClass();
		RepositoryInformation information = getRepositoryInformation(metadata, customImplementationClass);

		validate(information, customImplementation);

		Object target = getTargetRepository(information);

		// Create proxy
		ProxyFactory result = new ProxyFactory();
		result.setTarget(target);
		result.setInterfaces(new Class[] { repositoryInterface, Repository.class });

		result.addAdvice(ExposeInvocationInterceptor.INSTANCE);

		if (TRANSACTION_PROXY_TYPE != null) {
			result.addInterface(TRANSACTION_PROXY_TYPE);
		}

		for (RepositoryProxyPostProcessor processor : postProcessors) {
			processor.postProcess(result, information);
		}

		if (IS_JAVA_8) {
			result.addAdvice(new DefaultMethodInvokingMethodInterceptor());
		}

		result.addAdvice(new QueryExecutorMethodInterceptor(information, customImplementation, target));

		return (T) result.getProxy(classLoader);
	}
```

在这个实现类中, 有获取仓库元数据, 验证, 注册方法拦截器的advice等.  

> 注意: 在创建ProxyFactory实例后, 调用`getProxy()`返回的是Repository接口代理. 
>
> 这也是常说的: Spring Data JPA 的实现原理是动态代理机制

到此, 已经大致了解了一些工厂Bean`JpaRepositoryFactoryBean`所做的一些事情. 

接下来分析方法命名查询的重点就在 `QueryExecutorMethodIntrceptor`中了.



#### QueryExecutorMethodInterceptor方法拦截

首先看到它是RepositoryFactorySupport 的内部类, 并实现MethodInterceptor方法拦截器接口

我们在Repository接口中定义的查询方法是怎么被识别的? 在方法调用之前又经过了哪些处理? 

那么在这个拦截器中应该能找到想要的答案

在实例化时执行构造方法: `QueryExecutorMethodInterceptor()`

```java
public QueryExecutorMethodInterceptor(RepositoryInformation repositoryInformation, Object customImplementation, Object target) {

	//...
	QueryLookupStrategy lookupStrategy = getQueryLookupStrategy(queryLookupStrategyKey,
					RepositoryFactorySupport.this.evaluationContextProvider);
	lookupStrategy = lookupStrategy == null ? getQueryLookupStrategy(queryLookupStrategyKey) : lookupStrategy;
	Iterable<Method> queryMethods = repositoryInformation.getQueryMethods();

	if (lookupStrategy == null) {

		if (queryMethods.iterator().hasNext()) {
			throw new IllegalStateException("You have defined query method in the repository but "
							+ "you don't have any query lookup strategy defined. The "
							+ "infrastructure apparently does not support query methods!");
		}

		return;
	}

    SpelAwareProxyProjectionFactory factory = new SpelAwareProxyProjectionFactory();
    factory.setBeanClassLoader(classLoader);
    factory.setBeanFactory(beanFactory);

    for (Method method : queryMethods) {

        RepositoryQuery query = lookupStrategy.resolveQuery(method, repositoryInformation, factory, namedQueries);

        invokeListeners(query);
        queries.put(method, query);
    }
}
```

该方法的有这样一句原文注释`Builds a model of {@link QueryMethod}s to be invoked on execution of repository interface methods.`

上面的源码大致可以分为两部分: 

- 获取lookupStrategy(查找策略)
- 对Method进行迭代遍历, 根据lookupStrategy(查找策略)查询RepositoryQuery
- 为其添加监听
- 将Method和RepositoryQuery放入`Map<Method, RepositoryQuery> queries`缓存中

一句话总结就是为Repository接口中每个查询定义方法Method去构造相应的RepositoryQuery, RepositoryQuery的相关概念后面还会分析到.



#### lookupStrategy方法查询定义的查找策略

先从获取lookupStrategy(查找策略)看起:

`org.springframework.data.jpa.repository.query.JpaQueryLookupStrategy#create`

```java
public static QueryLookupStrategy create(EntityManager em, Key key, QueryExtractor extractor,
			EvaluationContextProvider evaluationContextProvider) {

		Assert.notNull(em, "EntityManager must not be null!");
		Assert.notNull(extractor, "QueryExtractor must not be null!");
		Assert.notNull(evaluationContextProvider, "EvaluationContextProvider must not be null!");

		switch (key != null ? key : Key.CREATE_IF_NOT_FOUND) {
			case CREATE:
				return new CreateQueryLookupStrategy(em, extractor);
			case USE_DECLARED_QUERY:
				return new DeclaredQueryLookupStrategy(em, extractor, evaluationContextProvider);
			case CREATE_IF_NOT_FOUND:
				return new CreateIfNotFoundQueryLookupStrategy(em, extractor, new CreateQueryLookupStrategy(em, extractor),
						new DeclaredQueryLookupStrategy(em, extractor, evaluationContextProvider));
			default:
				throw new IllegalArgumentException(String.format("Unsupported query lookup strategy %s!", key));
		}
	}
```

通过这段switch代码的条件判断和这三种策略对应的不同实现类, 可以总结出以下内容:

方法的查询策略有三种:

1. `Create`: 通过解析规范命名的方法名来创建查询, 此时即使有`@Query`或`@NameQuery`也会忽略. 如果方法名不符合规则，启动的时候会报异常
2. `USE_DECLARED_QUERY`: 也就是使用注解方式声明式创建, 启动时就会去尝试找声明的查询, 也就是使用`@Query`定义的语句来执行查询, 没有则找`@NameQuery`的, 再没有就异常.
3. `CREATE_IF_NOT_FOUND`: 先找`@Query`或`@NameQuery`定义的语句来查询, 如果没有就通过解析方法名来创建查询.

其中, 第三种方式`CREATE_IF_NOT_FOUND`相当于第一种和第二种的结合版. 也是默认的配置策略.

> QueryLookupStrategy 是策略的定义接口，JpaQueryLookupStrategy 是具体策略的实现类 



本节的重点是自定义方法命名的推导策略, 所以2和3就先略过, 直接找`Create`策略对应的实现类`CreateQueryLookupStrategy`

```java
private static class CreateQueryLookupStrategy extends AbstractQueryLookupStrategy {

	//...
	@Override
	protected RepositoryQuery resolveQuery(JpaQueryMethod method, EntityManager em, NamedQueries namedQueries) {
		//...
		return new PartTreeJpaQuery(method, em, persistenceProvider);
    	//...    
	}

}
```



####PartTreeJpaQuery实例推导

这时, 终于来到了实现name和method的拆分逻辑的方法了.  重点`PartTreeJpaQuery`

```java
/**
 * A {@link AbstractJpaQuery} implementation based on a {@link PartTree}.
 */
public class PartTreeJpaQuery extends AbstractJpaQuery {
    private final Class<?> domainClass;
	private final PartTree tree;
	private final JpaParameters parameters;

	private final QueryPreparer query;
	private final QueryPreparer countQuery;
	private final EntityManager em;

	public PartTreeJpaQuery(JpaQueryMethod method, EntityManager em, PersistenceProvider persistenceProvider) {

		super(method, em);

		this.em = em;
		this.domainClass = method.getEntityInformation().getJavaType();
		this.parameters = method.getParameters();
		// 。。。

		try {

			this.tree = new PartTree(method.getName(), domainClass);
			this.countQuery = new CountQueryPreparer(persistenceProvider, recreationRequired);
			this.query = tree.isCountProjection() ? countQuery : new QueryPreparer(persistenceProvider, recreationRequired);
			//。。。
		} 
	}
    
    // 。。。
}
```

看到注释, 是基于`PartTree实现` 那么一会的重点就是它了

其次, 它继承了抽象类`AbstractJpaQuery` 看下它的层次结构:

```
AbstractJpaQuery
	- StoredProcedureJpaQuery
	- PartTreeJpaQuery
	- NamedQuery
	- AbstractStringBasedJpaQuery
		- SimpleJpaQuery
		- NativeJpaQuery
```

- **SimpleJpaQuery**, **NativeJpaQuery**: 就是@Query查询定义中的`nativeQuery=false|true `

  前者使用JPQL, 后者使用原生SQL

- **NamedQuery**: 不用解释了,  @NamedQuery
- **StoredProcedureJpaQuery**: 根据名字也猜的差不多, 是跟存储过程调用有关的查询
- **PartTreeJpaQuery**: 这就是本节的重点, 方法命名查询的Query实例

到这里可以总结下: 

RepositoryQuery代表Repository接口中的一个查询方法

RepositoryQuery中持有QueryMethod实例, QueryMethod中持有Method实例. 

RepositoryQuery根据Repository接口中方法查询定义的不同, 被实例化成不同的子类实现.



#### PartTree语法树

回到`PartTree`中

```java
public class PartTree implements Iterable<OrPart> {
	private static final String KEYWORD_TEMPLATE = "(%s)(?=(\\p{Lu}|\\P{InBASIC_LATIN}))";
	private static final String QUERY_PATTERN = "find|read|get|query|stream";
	private static final String COUNT_PATTERN = "count";
	private static final String EXISTS_PATTERN = "exists";
	private static final String DELETE_PATTERN = "delete|remove";
	private static final Pattern PREFIX_TEMPLATE = Pattern.compile( //
			"^(" + QUERY_PATTERN + "|" + COUNT_PATTERN + "|" + EXISTS_PATTERN + "|" + DELETE_PATTERN + ")((\\p{Lu}.*?))??By");

	private final Subject subject;
	private final Predicate predicate;

	/**
	 * Creates a new {@link PartTree} by parsing the given {@link String}.
	 * 
	 * @param source the {@link String} to parse
	 * @param domainClass the domain class to check individual parts against to ensure they refer to a property of the
	 *          class
	 */
	public PartTree(String source, Class<?> domainClass) {

		Assert.notNull(source, "Source must not be null");
		Assert.notNull(domainClass, "Domain class must not be null");

		Matcher matcher = PREFIX_TEMPLATE.matcher(source);
		if (!matcher.find()) {
			this.subject = new Subject(null);
			this.predicate = new Predicate(source, domainClass);
		} else {
			this.subject = new Subject(matcher.group(0));
			this.predicate = new Predicate(source.substring(matcher.group().length()), domainClass);
		}
	}

    //...
}
```

配合着注释, 看到除了一些关键字定义外, 还定义了`Subject`和`Predicate`

- **`PartTree.Subject`**:  主语对象

  for example `"findDistinctUserByNameOrderByAge"` would have the subject `"DistinctUser"`.

- **`PartTree.Predicate`**: 谓语对象

  for example` "findDistinctUserByNameOrderByAge"` would have the predicate `"NameOrderByAge"`.



##### Subject主语部分

```java
private static class Subject {

	// ...
    
    public Subject(String subject) {

        this.distinct = subject == null ? false : subject.contains(DISTINCT);
        this.count = matches(subject, COUNT_BY_TEMPLATE);
        this.exists = matches(subject, EXISTS_BY_TEMPLATE);
        this.delete = matches(subject, DELETE_BY_TEMPLATE);
        this.maxResults = returnMaxResultsIfFirstKSubjectOrNull(subject);
    }

	// ...		
}
```

Subject(主语对象)的职责很简单: 通过正则提取几个构建`PartTree`的属性

也就是几个布尔常量: `distinct`, `count`, `exists`, `delete`, `maxResults`



##### Predicate谓语部分

```java
private static class Predicate {

	//...

    public Predicate(String predicate, Class<?> domainClass) {

        String[] parts = split(detectAndSetAllIgnoreCase(predicate), ORDER_BY);

        if (parts.length > 2) {
            throw new IllegalArgumentException("OrderBy must not be used more than once in a method name!");
        }

        buildTree(parts[0], domainClass);
        this.orderBySource = parts.length == 2 ? new OrderBySource(parts[1], domainClass) : null;
    }
}
```

构造参数`predicate`是方法名的谓语部分, `domainClass`就是domainClass

可以看到大致做了这几件事: 

- `detectAndSetAllIgnoreCase()`删除`AllIgnoreCase`关键字, 然后设置`alwaysIgnoreCase=true`
- 将剩余部分根据`orderBy`关键字分割成parts数组
- 用排序关键字`orderBy`之前的部分 `buildTree()`
- 用排序关键字`orderBy`之后的部分构建排序部分

举个例子: 方法名为`findDistinctUserByNameOrderByAgeAllIgnoringCase()`

解析出的谓语部分为: `NameOrderByAgeAllIgnoringCase`, 首先会删除`AllIgnoreCase` 得到`NameOrderByAge`

再根据排序关键字分割为`Name`和`Age`两部分, `buildTree()`, 设置排序`orderBySource`为``Order By age: ASC``



**buildTree()**

该方法实际是对谓语部分进行`or`和`and`分割, 生成语法树节点的过程.

Nodes:  首先用`or`分割, 构建`OrPart`子节点

Children: 子节点包含的`child node`由`and`继续分割`Part`

在Part中还设置了每个属性的`Type`和`PropertyPath`

- **Part.Type**: 属性约束的关键字枚举, 例如`BETWEEN`, `IS_NOT_NULL`等
- **PropertyPath**: 映射对应的`domain class`中的属性路径, 例如`User.name`, `User.age`

到这里整个方法命名查询的PartTreeJpaQuery实例就推导完成了 





#### 命名方法执行过程

PartTreeJpaQuery实例分析完成后会放入上面讲到的`QueryExecutorMethodInterceptor.queries`中

以` ConcurrentHashMap<Method, RepositoryQuery> queries`的形式存放

> 注意: Spring容器启动时候, 初始化时就会创建这些实例. 如果定义了错误的命名查询, 在启动时就会抛出异常.

有了`queries` 在什么时候执行呢?

还记得`RepositoryQuery`接口中的方法`execute（`

通过find used找到上面提到的`RepositoryFactorySupport.QueryExecutorMethodInterceptor#doInvoke`中.

```java
private Object doInvoke(MethodInvocation invocation) throws Throwable {

    Method method = invocation.getMethod();
    Object[] arguments = invocation.getArguments();

    if (isCustomMethodInvocation(invocation)) {

        Method actualMethod = repositoryInformation.getTargetClassMethod(method);
        return executeMethodOn(customImplementation, actualMethod, arguments);
    }

    if (hasQueryFor(method)) {
        // 看这里!
        return queries.get(method).execute(arguments);
    }

    // Lookup actual method as it might be redeclared in the interface
    // and we have to use the repository instance nevertheless
    Method actualMethod = repositoryInformation.getTargetClassMethod(method);
    return executeMethodOn(target, actualMethod, arguments);
}
```

看到了吧? 在实际调用时, 会直接从`queries`缓存中获取`RepositoryQuery`, 执行查询.



### 总结

整个推导过程到最后, 实际就是去看`PartTree`语法生成树的逻辑.

无论是SQL解析器的逆向解析SQL语义还是本节`PartTree`中根据语义解析出SQL.

对我们来说这都不是重点. 所以本节大量篇幅都放在根据方法命名推导的原理上

包括Repository的工厂类 -> JpaRepositoryFactoryBean的由来和Bean初始化后的一系列回调操作

并补充了Repository及其子接口和lookupStrategy查询策略的相关知识.

所以涉及到了`Spring Data Common`和`Spring Data JPA`模块的接口和实现类. 

所以最懵的可能就是这些接口和实现的关系了. 

所以这里引用一张UML图: http://www.dewafer.com/uploads/2017/2017-02-21-JpaRepoUML.png








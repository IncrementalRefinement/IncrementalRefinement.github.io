---
layout: post
title: Spring 容器浅析
tags: [Spring, SourceCode Reading]
---

本文选取 ClassPathXmlApplicationContext 作为例子并结合一定量源码(5.3.7)讲解 Spring 容器的一系列相关概念（容器初始化、IoC、DI等）,我目前只看了个囫囵，先记录一下当下的理解，留待以后纠正。

## 容器初始化过程

首先我们点进 ClassPathXmlApplicationContext 的构造函数

```java
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
    super(parent);
    this.setConfigLocations(configLocations);
    if (refresh) {
        this.refresh();
    }

}
```

首先调用了父类的构造函数，之后使用 fresh() 来初始化当前的上下文，这个 fresh() 就是整个容器初始化的核心，我们之后会讲到

### UML 关系

整个 UML 图比较麻烦，跳过不画了，自己开 IDEA 看一下就行，这里讲一下几个接口和类

我们可以看到 ClassPathXmlApplicationContext 间接实现了 ApplicationContext 这一接口，而 ApplicationContext 接口又继承了 ListableBeanFactory、HierarchicalBeanFactory 等接口， 而 ListableBeanFactory、HierarchicalBeanFactory 接口又继承了 BeanFactory 接口(八股文常考内容)，看到这里肯定会很晕，那么接下来我们大致讲一下这几个接口。

有一句话很有意思，我这里摘出来：

> ApplicationContext inherits from BeanFactory, but it should not be understood as the implementation class of BeanFactory. Instead, it holds an instantiated BeanFactory (DefaultListableBeanFactory) internally.

也就是说虽然 ApplicationContext 实现了 BeanFactory 接口，但是该接口是通过持有一个 BeanFactory 并在实际调用之间做一个代理的方式实现这个接口的，我们之后就能在 IoC 部分看到这一点

#### BeanFactory

```java
public interface BeanFactory {
    // ....

    Object getBean(String var1) throws BeansException;

    // ...

    <T> ObjectProvider<T> getBeanProvider(Class<T> var1);

    <T> ObjectProvider<T> getBeanProvider(ResolvableType var1);

    boolean containsBean(String var1);

    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;

    @Nullable
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;

    @Nullable
    Class<?> getType(String var1, boolean var2) throws NoSuchBeanDefinitionException;

    String[] getAliases(String var1);
}
```

省略了几个重载的函数， 很显然 BeanFactory 用于返回一个 Bean

#### ListableBeanFactory

```java
public interface ListableBeanFactory extends BeanFactory {
    boolean containsBeanDefinition(String var1);

    int getBeanDefinitionCount();

    String[] getBeanDefinitionNames();

    <T> ObjectProvider<T> getBeanProvider(Class<T> var1, boolean var2);

    String[] getBeanNamesForType(ResolvableType var1);

    <T> Map<String, T> getBeansOfType(@Nullable Class<T> var1) throws BeansException;


    String[] getBeanNamesForAnnotation(Class<? extends Annotation> var1);

    Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> var1) throws BeansException;

    @Nullable
    <A extends Annotation> A findAnnotationOnBean(String var1, Class<A> var2) throws NoSuchBeanDefinitionException;
}
```

ListableBeanFactory 用于根据一类属性(Type, Annotation) 返回一组聚合的 Bean

#### HierarchicalBeanFactory

```java
public interface HierarchicalBeanFactory extends BeanFactory {
    @Nullable
    BeanFactory getParentBeanFactory();

    boolean containsLocalBean(String var1);
}
```

HierarchicalBeanFactory 接口可以用来设置 BeanFactory 之间的亲属、继承关系

#### ApplicationContext

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    @Nullable
    String getId();

    String getApplicationName();

    String getDisplayName();

    long getStartupDate();

    @Nullable
    ApplicationContext getParent();

    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

ApplicationContext 在实现了上述等几个接口之外还具有一个 AutowireCapableBeanFactory 的 get 方法

### 初始化过程

refresh() 的代码如下, 我在有理解的部分写了注释

```java
public void refresh() throws BeansException, IllegalStateException {
    // 同步，保证并发安全
    synchronized(this.startupShutdownMonitor) {
        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
        // 设置状态变量，打印日志，记录时间，不重要
        this.prepareRefresh();
        // 重要，BeanDefinition 的解析就在这一步里进行
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
        // 设置类加载器，配置 beanFactory 的参数等，不重要，但是很繁琐
        this.prepareBeanFactory(beanFactory);

        try {
            this.postProcessBeanFactory(beanFactory);
            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
            this.invokeBeanFactoryPostProcessors(beanFactory);
            this.registerBeanPostProcessors(beanFactory);
            beanPostProcess.end();
            this.initMessageSource();
            this.initApplicationEventMulticaster();
            this.onRefresh();
            this.registerListeners();
            // 会在这一步里初始化不是 lazy allocate 的单例 Bean
            this.finishBeanFactoryInitialization(beanFactory);
            this.finishRefresh();
        } catch (BeansException var10) {
            if (this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var10);
            }

            this.destroyBeans();
            this.cancelRefresh(var10);
            throw var10;
        } finally {
            this.resetCommonCaches();
            contextRefresh.end();
        }

    }
}
```

我们来看一眼 obtainFreshBeanFactory() 这个 refresh() 中最核心的函数

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    this.refreshBeanFactory();
    return this.getBeanFactory();
}
```

看起来内容很简单，刷新掉旧的 BeanFactory， 然后根据配置获得新的 BeanFactory 就可以了，当然 getBeanFactory() 里面的具体实现还是挺繁琐的，懒得讲了，反正我一时半会也不会去写 Bean 容器，看了也是忘。这里面的操作可以理解成以下这句话：

> After a long link, a configuration file is finally converted into a DOM tree

通过一个繁复的解析操作，我们把写在 xml 文件里的配置解析成了 BeanDefinition。 BeanDefinition 可以理解成 Bean 在 JVM 里的的 Schema，也就是把 xml 文件里的配置内容转成了一个 POJO，我们根据这个蓝图里的 meatadata 去实例化需要的 Bean

在 Beans 配置文件中：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
```

包含了 Spring 默认 xml 标签的 Schema， 如果要用到更多的标签，可以在 beans 标签中包含更多的 .xsd 链接，也可以自定义标签, 如 \<Dubbo /> \<sofa:service> 等等 （当然我认为这里可能还要自定义解析这些标签的方法，以及自定义对应的 BeanDefinition 类等，不赘述）

## IoC 实现

容器生成之后已经初始化了一部分单例、饿汉型的 Bean，但还有一部分 Bean 没有初始化，这时候我们是如何从容器中获得 Bean 的呢？

大致思路其实很简单，首先去查询 Bean 有没有被缓存 && 是不是单例，如果单例 Bean 已经被初始化并缓存那么就直接读已经初始化过的Bean即可，否则再走一遍初始化的流程

### 获取 Spring Bean

先把代码列出来

```java
public Object getBean(String name) throws BeansException {
    this.assertBeanFactoryActive();
    return this.getBeanFactory().getBean(name);
}
```

AbstractApplicationContext 这个抽象类实现了 BeanFactory 的 getbean() 方法, 他首先获得自己内部持有的 BeanFactory，再从 BeanFactory 里面获得 Bean。那么问题来了，为什么要在这里用代理模式呢？我认为是为了减小耦合，方便在同一个上下文里换上不同的 BeanFactory， 实现多态。

接下来是实现了 BeanFactory 接口的 AbstractBeanFactory 抽象类里的 getBean() doGetBean()

```java
public Object getBean(String name) throws BeansException {
    return this.doGetBean(name, (Class)null, (Object[])null, false);
}
```

这个方法很长，我直接在注释里给出解析了

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    String beanName = this.transformedBeanName(name);
    Object sharedInstance = this.getSingleton(beanName);
    Object beanInstance;
    // 如果是单例，且已经被初始化过，那么直接加载缓存并打印日志，不重要
    if (sharedInstance != null && args == null) {
        if (this.logger.isTraceEnabled()) {
            if (this.isSingletonCurrentlyInCreation(beanName)) {
                this.logger.trace("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
            } else {
                this.logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
            }
        }

        beanInstance = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
    } else {
        if (this.isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        BeanFactory parentBeanFactory = this.getParentBeanFactory();
        // 如果当前的 BeanFactory 中找不到 BeanDefinition， 那么就将任务委派给双亲 BeanFactory
        if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
            String nameToLookup = this.originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) {
                return ((AbstractBeanFactory)parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
            }

            if (args != null) {
                return parentBeanFactory.getBean(nameToLookup, args);
            }

            if (requiredType != null) {
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }

            return parentBeanFactory.getBean(nameToLookup);
        }

        if (!typeCheckOnly) {
            this.markBeanAsCreated(beanName);
        }

        StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate").tag("beanName", name);

        try {
            if (requiredType != null) {
                beanCreation.tag("beanType", requiredType::toString);
            }

            RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
            this.checkMergedBeanDefinition(mbd, beanName, args);
            String[] dependsOn = mbd.getDependsOn();
            String[] var12;
            if (dependsOn != null) {
                var12 = dependsOn;
                int var13 = dependsOn.length;

                for(int var14 = 0; var14 < var13; ++var14) {
                    String dep = var12[var14];
                    if (this.isDependent(beanName, dep)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                    }

                    this.registerDependentBean(dep, beanName);

                    try {
                        this.getBean(dep);
                    } catch (NoSuchBeanDefinitionException var31) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "'" + beanName + "' depends on missing bean '" + dep + "'", var31);
                    }
                }
            }
            
            if (mbd.isSingleton()) {
            // 如果是单例，采用以下的逻辑
                sharedInstance = this.getSingleton(beanName, () -> {
                    try {
                        return this.createBean(beanName, mbd, args);
                    } catch (BeansException var5) {
                        this.destroySingleton(beanName);
                        throw var5;
                    }
                });
                beanInstance = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            } else if (mbd.isPrototype()) {
            // 如果是 Prototype，采用另一套逻辑
                var12 = null;

                Object prototypeInstance;
                try {
                    this.beforePrototypeCreation(beanName);
                    prototypeInstance = this.createBean(beanName, mbd, args);
                } finally {
                    this.afterPrototypeCreation(beanName);
                }

                beanInstance = this.getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            } else {
            // 都不是，采用自定义的 Scope 的初始化方法
                String scopeName = mbd.getScope();
                if (!StringUtils.hasLength(scopeName)) {
                    throw new IllegalStateException("No scope name defined for bean ��" + beanName + "'");
                }

                Scope scope = (Scope)this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                }

                try {
                    Object scopedInstance = scope.get(beanName, () -> {
                        this.beforePrototypeCreation(beanName);

                        Object var4;
                        try {
                            var4 = this.createBean(beanName, mbd, args);
                        } finally {
                            this.afterPrototypeCreation(beanName);
                        }

                        return var4;
                    });
                    beanInstance = this.getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                } catch (IllegalStateException var30) {
                    throw new ScopeNotActiveException(beanName, scopeName, var30);
                }
            }
        } catch (BeansException var32) {
            beanCreation.tag("exception", var32.getClass().toString());
            beanCreation.tag("message", String.valueOf(var32.getMessage()));
            this.cleanupAfterBeanCreationFailure(beanName);
            throw var32;
        } finally {
            beanCreation.end();
        }
    }

    return this.adaptBeanInstance(name, beanInstance, requiredType);
}
```

### 实现依赖注入

我们已经看到了想容器要 Bean 的 IoC 操作了，但 Bean 内部的 Field 是怎么进行注入的呢？我们再去看 Bean 的初始化过程，AbstractAutowireCapableBeanFactory 的 createBean() 会经一步调用 doCreateBean() 函数，doCreateBean() 里的关键操作就是 populateBean() 和紧随其后的 initializeBean()，前者进行依赖注入，后者完成初始化，我们直接看这两个方法

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // ...
    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
    if (resolvedAutowireMode == 1 || resolvedAutowireMode == 2) {
        MutablePropertyValues newPvs = new MutablePropertyValues((PropertyValues)pvs);
        if (resolvedAutowireMode == 1) {
            this.autowireByName(beanName, mbd, bw, newPvs);
        }

        if (resolvedAutowireMode == 2) {
            this.autowireByType(beanName, mbd, bw, newPvs);
        }

        pvs = newPvs;
    }
    // ...
}
```

不多说，都在代码里了，要细看直接点进去

```java
    protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
        if (System.getSecurityManager() != null) {
            AccessController.doPrivileged(() -> {
                this.invokeAwareMethods(beanName, bean);
                return null;
            }, this.getAccessControlContext());
        } else {
            this.invokeAwareMethods(beanName, bean);
        }

        Object wrappedBean = bean;
        // applyBeanPostProcessorsBeforeInitialization
        if (mbd == null || !mbd.isSynthetic()) {
            wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
        }

        // Initialization
        try {
            this.invokeInitMethods(beanName, wrappedBean, mbd);
        } catch (Throwable var6) {
            throw new BeanCreationException(mbd != null ? mbd.getResourceDescription() : null, beanName, "Invocation of init method failed", var6);
        }

        // applyBeanPostProcessorsAfterInitialization
        if (mbd == null || !mbd.isSynthetic()) {
            wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        }

        return wrappedBean;
    }
```

这边很有意思的一点就是我们实现了 BeanPostProcessor 接口的 Bean 的两个方法分别在 Initialization 之前&之后进行了调用，如果我们要解析自定义的注解，那么我们可以将他们写到这两个方法里

## 对比 Spring Boot

现在我们把 Spring Boot 的容器启动部分源码也来对照着看一下，大致原理也是相同的

```java
@SpringBootApplication
public class SpringBootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }

}
```

首先有一个打上 SpringBootApplication 注解的类，类里的 main() 调用 SpringApplication 的静态的 run() 方法

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}

public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);
}
```

run 方法进行了重载，将输入一个 Class 转化成 Class 数组后调用，run 里面先 new 一个 SpringApplication 对象，再调用这个对象的 run 方法，接下来看构造函数和执行的 run() 函数

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.sources = new LinkedHashSet();
    this.bannerMode = Mode.CONSOLE;
    this.logStartupInfo = true;
    this.addCommandLineProperties = true;
    this.addConversionService = true;
    this.headless = true;
    this.registerShutdownHook = true;
    this.additionalProfiles = Collections.emptySet();
    this.isCustomEnvironment = false;
    this.lazyInitialization = false;
    this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
    this.applicationStartup = ApplicationStartup.DEFAULT;
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.bootstrapRegistryInitializers = this.getBootstrapRegistryInitializersFromSpringFactories();
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

构造函数其实很无聊，就是设置了一遍 SpringApplication 对象内部的 Field 并进行一些设置，不过里面的 WebApplicationType.deduceFromClasspath() 这个函数其实很体现 Spring Boot 约定优于配置的特性。根据文件结构来推断应用类型，太典型的约定优于配置了。当然约定优于配置也有其缺点，加大了学习框架以及看框架源码的难度，毕竟约定在源码里的体现就像 magic number 一样，很容易让新手感到一头雾水。

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
    ConfigurableApplicationContext context = null;
    this.configureHeadlessProperty();
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);

    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
        Banner printedBanner = this.printBanner(environment);
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        // 找到你惹, refresh！
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
        }

        listeners.started(context);
        this.callRunners(context, applicationArguments);
    } catch (Throwable var10) {
        this.handleRunFailure(context, var10, listeners);
        throw new IllegalStateException(var10);
    }

    try {
        listeners.running(context);
        return context;
    } catch (Throwable var9) {
        this.handleRunFailure(context, var9, (SpringApplicationRunListeners)null);
        throw new IllegalStateException(var9);
    }
}
```

run() 的代码的既视感其实很强，不赘述了，我们又看到熟悉的 refreshContext() 函数。没错，这也是 Spring Boot 容器启动的核心，SpringApplication 对象内部包含了一个 ApplicatContext，调用 refreshContext() 最终会在 ApplicationContex 上调用 refresh()

## Random Thoughts

1. 每次点开一个类，总会伴随着一个非常庞大的 UML 关系图，一开始觉得贼吓人，直到一点点拆解之后才逐渐清晰起来。
2. 没有想到一个初始化容器的操作的背后能有这么精细&庞大的代码，结合上一点，我感受到了抽象对于构建大型项目的重要性，源码看到最后有几个方法我也没有点进去看，感觉意义不是特别大。
3. Spring 里面很多检查安全性 & 打印日志的代码量其实很大，看的时候跳过就好。
4. 之前看到一个问题说是如何看 Spring 源码，现在才发现这个问题最适宜的回答就是不要去看 Spring 源码，或者说只需要大致看一下 Spring 源码即可。一方面要清楚地带着问题/目的去有选择地读而不是事无巨细地钻进去，另一方面在阅读的过程中有一些抽象没有必要细究，细究就溺死在 if else， try catch 里面了，直接当一个黑盒即可，专注于大局才是正确的做法。

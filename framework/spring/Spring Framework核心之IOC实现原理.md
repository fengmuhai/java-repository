Spring Framework核心之IOC实现原理
====================


一、Spring框架的两大核心就是IOC控制反转，和AOP面向切面编程
====================

控制反转
--------------------
**控制反转**说的是应用运行过程中，对象之间的依赖从代码指定的方式，变为由容器来管理，通过依赖注入来实现。IOC是解耦组建之间关系的利器，把应用从复杂的依赖关系中解放出来，大大简化的应用的开发关系。

注入方式
--------------------
**一共有三种方式：接口注入、setter注入、构造器注入**，其中setter注入和构造器注入是主要的注入方式。


IOC容器的主要组件：BeanFactory和ApplicationContext
--------------------
**BeanFactory**实现了最基本的容器功能；

**ApplicationContext**应用上下文作为容器的高级形态存在。


二、IOC容器的初始化过程
=====================

**IOC容器的初始化是有refresh()方法来启动的**，这个方法标志着IOC容器的启动。具体启动包括：**BeanDefinition和Resource的定位、载入、和注册**三个基本过程。

**1、Resource定位过程：** 是指BeanDefinition的定位，它有ResourceLoader通过Resource接口完成。可以理解为是容器寻找数据的过程；

**2、BeanDefinition载入：** 把用户定义好的Bean文件转化为IOC容器内部定义的数据结构，而这个数据结果就是BeanDefinition。

**3、向IOC容器注册BeanDefinition：** 调用BeanDefinitionRegistry接口来实现注册，把载入过程解析得到的BeanDefinition注册到IOC容器中，具体是注册到一个HashMap中，IOC容器通过这个Map来持有BeanDefinition的数据。

**注意：初始化过程一般不包括依赖注入，在Spring Ioc中，Bean定义的载入和依赖注入是两个独立的过程。依赖注入一般发生在第一次通过getBean()向容器获取Bean的时候。但是如果指定了Bean的lazyinit属性，那么这个Bean的依赖注入在IOC容器初始化的时候就已经完成了。**


1.BeanDefinition的Resource定位
=====================
以FileSystemXmlApplicationContext为例分析
----------------------
1.FileSystemXmlApplicationContext实现了getResourceByPath()方法。这是一个模板方法，是为读取Resource服务的：


**FileSystemXmlApplicationContext.class**
```
  protected Resource getResourceByPath(String path) {
        if (path.startsWith("/")) {
            path = path.substring(1);
        }

        return new FileSystemResource(path);
    }
```
FileSystemXmlApplicationContext的**refresh()方法非常重要**，它是分析IOC容器初始化的重要入口，Ioc容器功能的相关实现分析由此开始：

IOC容器开始对BeanDefinition的定位是有refresh()触发的，而refresh的调用是由FileSystemXmlApplicationContext的构造方法启动的。

**FileSystemXmlApplicationContext.class**
```
    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
        super(parent);
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();
        }

    }
```

**在什么地方定义的BeanDefinition的读入器BeanDefinitionReader呢？接下来往下看refresh()源码：**

**AbstractApplicationContext.class**
```
pfix=org.springframework.context.support
pfix.FileSystemXmlApplicationContext
   ↳     pfix.AbstractXmlApplicationContext
         ↳     pfix.AbstractRefreshableConfigApplicationContext
                ↳     pfix.AbstractRefreshableApplicationContext
                      ↳     pfix.AbstractApplicationContext
                            ↳     pfix.DefaultResourceLoader
                                  ↳    pfix.ResourceLoader
                  
    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            ...

        }
    }

```
obtainFreshBeanFactory() -> refreshBeanFactory()
-------------------------
ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();

**AbstractApplicationContext.class**
```
    protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
        this.refreshBeanFactory();
        return this.getBeanFactory();
    }
```
this.refreshBeanFactory();

**AbstractRefreshableApplicationContext.class**
```
    protected final void refreshBeanFactory() throws BeansException {
        ...

        try {
            DefaultListableBeanFactory beanFactory = this.createBeanFactory();
            beanFactory.setSerializationId(this.getId());
            this.customizeBeanFactory(beanFactory);
            this.loadBeanDefinitions(beanFactory);
            Object var2 = this.beanFactoryMonitor;
            synchronized(this.beanFactoryMonitor) {
                this.beanFactory = beanFactory;
            }
        } 
        .....
    }
```

由obtainFreshBeanFactory -> refreshBeanFactory -> createBeanFactory 构建了容器，由loadBeanDefinitions(beanFactory)启动载入了BeanDefinition。
-------------------

1.构建了一个IOC容器供ApplicationContext使用，这个容器就是DefaultListableBeanFactory。**DefaultListableBeanFactory是很重要的一个Ioc实现**，XmlBeanFactory、ApplicationContext等这些类都是使用DefaultListableBeanFactory来作为基类的，

2.通过XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory，完成对BeanDefinition信息的读取；

**AbstractXmlApplicationContext.class**
```
    protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        beanDefinitionReader.setEnvironment(this.getEnvironment());
        beanDefinitionReader.setResourceLoader(this);
        beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
        this.initBeanDefinitionReader(beanDefinitionReader);
        this.loadBeanDefinitions(beanDefinitionReader);
    }
```

由this.loadBeanDefinitions(beanDefinitionReader)完成资源获取和读取
-------------------------
**AbstractXmlApplicationContext.class**
```
    protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
        Resource[] configResources = this.getConfigResources();
        if (configResources != null) {
            reader.loadBeanDefinitions(configResources);
        }

        String[] configLocations = this.getConfigLocations();
        if (configLocations != null) {
            reader.loadBeanDefinitions(configLocations);
        }

    }
```

获取资源：this.getConfigResources();通过DefaultResourceLoader实现
-----------------------------
**AbstractXmlApplicationContext.class**
```
    protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
        Resource[] configResources = this.getConfigResources();
        if (configResources != null) {
            reader.loadBeanDefinitions(configResources);
        }

        String[] configLocations = this.getConfigLocations();
        if (configLocations != null) {
            reader.loadBeanDefinitions(configLocations);
        }

    }
```
执行：reader.loadBeanDefinitions(configLocations);

**AbstractBeanDefinitionReader.class**
```
    public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
        Assert.notNull(locations, "Location array must not be null");
        int count = 0;
        String[] var3 = locations;
        int var4 = locations.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String location = var3[var5];
            count += this.loadBeanDefinitions(location);
        }

        return count;
    }
    
    public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
        Assert.notNull(locations, "Location array must not be null");
        int count = 0;
        String[] var3 = locations;
        int var4 = locations.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String location = var3[var5];
            count += this.loadBeanDefinitions(location);
        }

        return count;
    }
    
    public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
        return this.loadBeanDefinitions(location, (Set)null);
    }
    
    public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
        ResourceLoader resourceLoader = this.getResourceLoader();
        if (resourceLoader == null) {
            ...
        } else {
            int count;
            if (resourceLoader instanceof ResourcePatternResolver) {
                ....
            } else {
                Resource resource = resourceLoader.getResource(location);
                count = this.loadBeanDefinitions((Resource)resource);
                ...
                return count;
            }
        }
    }    
```

DefaultResourceLoader.class 执行Resource resource = resourceLoader.getResource(location);
----------------
最终会执行到his.getResourceByPath(location)，呼应了SystemFileXmlApplicationCOntext中重写的getResourceByPath()方法。
**DefaultResourceLoader.class**
```
    public Resource getResource(String location) {
        Assert.notNull(location, "Location must not be null");
        Iterator var2 = this.protocolResolvers.iterator();

        Resource resource;
        do {
            if (!var2.hasNext()) {
                if (location.startsWith("/")) {
                    return this.getResourceByPath(location);
                }

                if (location.startsWith("classpath:")) {
                    return new ClassPathResource(location.substring("classpath:".length()), this.getClassLoader());
                }

                try {
                    URL url = new URL(location);
                    return (Resource)(ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
                } catch (MalformedURLException var5) {
                    return this.getResourceByPath(location);
                }
            }

            ProtocolResolver protocolResolver = (ProtocolResolver)var2.next();
            resource = protocolResolver.resolve(location, this);
        } while(resource == null);

        return resource;
    }
```



2.BeanDefinition的载入和解析
=====================
**在完成BeanDefinition的Resource定位分析后，下面来了解BeanDefinition的信息载入过程。对IOC容器来说，这个载入过程就是对BeanDefinition的定义转化成Spring的内部数据结构的过程，这些信息保存在一个ConcurrentHashMap中。依赖注入就是通过这些数据结构信息来完成的。**

前面分析过BeanFactory的初始化过程，是从AbstractApplicationContext类中的refresh()开始的。BeanDefinition的入口也是在这里，具体是从AbstractRefreshableApplicationContext类的refreshBeanFactory()方法开始，如果创建IOC容器前，如果已经存在容器，那么需要把Beans和容器销毁，再创建新的，类似于计算机的重启。
```
    protected final void refreshBeanFactory() throws BeansException {
        if (this.hasBeanFactory()) {
            this.destroyBeans();
            this.closeBeanFactory();
        }

        try {
            DefaultListableBeanFactory beanFactory = this.createBeanFactory();
            beanFactory.setSerializationId(this.getId());
            this.customizeBeanFactory(beanFactory);
            //这里是重点
            this.loadBeanDefinitions(beanFactory);
            Object var2 = this.beanFactoryMonitor;
            synchronized(this.beanFactoryMonitor) {
                this.beanFactory = beanFactory;
            }
        } catch (IOException var5) {
            throw new ApplicationContextException("I/O error parsing bean definition source for " + this.getDisplayName(), var5);
        }
    }
```
载入过程在loadBeanDefinitions()方法开始
---------------------
**AbstractXmlApplicationContext.class**

```
    protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        beanDefinitionReader.setEnvironment(this.getEnvironment());
        beanDefinitionReader.setResourceLoader(this);
        beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
        this.initBeanDefinitionReader(beanDefinitionReader);
        this.loadBeanDefinitions(beanDefinitionReader);
    }


```

得到BeanDefinition信息的Resource的定位后，调用XmlBeanDefinitionReader来读取，**具体载入过程是委托给BeanDefinitionReader来完成的**。

因为BeanDefinition是通过XML来定义，所以使用XmlBeanDefinitionReader来载入BeanDefinition到容器中：
```
    protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
        Resource[] configResources = this.getConfigResources();
        if (configResources != null) {
            reader.loadBeanDefinitions(configResources);
        }

        String[] configLocations = this.getConfigLocations();
        if (configLocations != null) {
            reader.loadBeanDefinitions(configLocations);
        }

    }
```

```
    public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
        Assert.notNull(resources, "Resource array must not be null");
        int count = 0;
        Resource[] var3 = resources;
        int var4 = resources.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            Resource resource = var3[var5];
            count += this.loadBeanDefinitions((Resource)resource);
        }

        return count;
    }

```



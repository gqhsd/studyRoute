# Spring源码深度解析

## 容器的基础实现

### Spring容器核心类介绍

#### DefaultListableBeanFactory 默认加载bean

<img src="../../Desktop/assets/image-20230218123714795.png" alt="image-20230218123714795"  />

- AiasRegistry(接口)定义对 alias 的简单增删改等操作

- SimpleAliasRegitry 主要使用 map 作为 alias 的缓存，并对接口 AliasRegistry 进行实现

------

- SingletonBeanRegistry(接口) ：定义对单例bean的注册及获取

- DefaultSingletonBeanRegistry: 对接口SingletonBeanRegistry 各函数的实现。
- FactoryBeanRegistrySupport: 在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。

------

- BeanFactory(接口) ：定义获取 bean和bean的各种属性
- ConfigurableBeanFactory: 提供配置Factory 的各种方法。

- HierarchicalBeanFactory(接口): 继承BeanFactory,也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持。
- ConfigurableListableBeanFactory(接口): BeanFactory 配置清单，指定忽略类型及接口等。
- ListableBeanFactory(接口): 根据各种条件获取bean的配置清单。
- AbstractBeanFactory: 综合FactoryBeanRegistrySupport和ConfigurableBeanFactory 的功能。
- AutowireCapableBeanFactory(接口): 提供创建bean、自动注入、初始化以及应用bean 的后处理器。
- AbstractAutowireCapableBeanFactory:综合AbstractBeanFactory 并对接口Autowire CapableBeanFactory进行实现。

- ------

  BeanDefinitionRegistry(接口): 定义对BeanDefinition的各种增删改操作。

- **DefaultListableBeanFactory**: 综合上面所有功能，主要是对bean注册后的处理。

  ------

  **XmlBeanFactory**对DefaultListableBeanFactory 类进行了扩展，用于**从XML文档中读取BeanDefinition**,对于注册及获取bean还是使用从父类DefaultI istableBeanFactory继承的方法去实现，而唯独**与父类不同的个性化实现**就是**增加了XmlBeanDefinitionReader 类型的reader**属性。在XmlBeanFactory中主要**使用reader属性对资源文件进行读取和注册**。

#### XmlBeanDefinitionReader

常见类:

![image-20230218194935598](../../Desktop/assets/image-20230218194935598.png)

- ResourceLoader(接口):定义资源加载器，主要应用于根据给定的资源文件地址返回对应的Resource。
- BeanDefinitionReader(接口):主要定义资源文件读取并转换为BeanDefinition的各个功能。
- EnvironmentCapable(接口)定义获取Environment方法。
- DocumentLoader(接口):定义从资源文件加载到转换为Document的功能。
- AbstractBeanDefinitionReader:对EnvironmentCapable、BeanDefinitionReader 类定义的功能进行实现。
- BeanDefinitionDocumentReader(接口): 定义读取Docuemnt并注册BeanDefinition 功能。
- BeanDefinitionParserDelegate: 定义解析Element的各种方法。

工作流程:

1.通过继承自AbstractBeanDefinitionReader 中的方法，来使用ResourLoader 将资源文件路径转换为对应的Resource 文件。
2.通过DocumentLoader对Resource文件进行转换，将Resource 文件转换为Document文件。
3.通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析。


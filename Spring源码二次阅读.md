### 



#  Spring源码二次阅读

# 一. spring概述：

## 1. 基本概念,名词解释等等

BeanDefinitionReader: 用于读取bean定义信息文件的读取器.

**spring中Bean的类型:**

- singleton
- protptype
- request
- session



**后置处理器:**

BeanFactoryPostProccessor:修改beanDefinition信息

BeanPostProccessor:修改bean信息

**创建bean的基础流程(前期简单理解下):**

![image-20220126000540938](/Users/wzw/Desktop/assets/image-20220126000540938.png)

**实例化:**

在堆中分配一块空间->所有值都是默认值

初始化: 给属性设置值:  1. 填充属性 2. 执行初始化方法



**Aware到底是干什么的?**

spring框架用于获取到bean的相关信息的接口,当某个bean实现了aware的相关接口,spring框架初始化这个bean的时候,会注入指定的信息,到这个bean的指定属性里面去.

spring在不同的阶段分别要做不同的事情,是怎么实现的?

->观察者模式:监听器,监听事件,广播器



**Enviroment接口**

在容器启动之前,spring就回获取到系统的各种值,方便后续使用而不是每次要去读取

**FactoryBean和BeanFactory的区别**

1. 都是用来创建对象的对象工厂

当使用BeanFactory的时候,必须遵循完整的创建过程,这个过程由spring控制

使用FactoryBean只需要调用getObject就可以返回具体对象,这个创建过程由用户自己控制,更加灵活.  

创建过程区别:

实现了FactoryBean的对象还是正常流程创建的,而getObject方法是在getBean方法中调用的,不是直接创建的.并且通过getObject()得到的bean放在另外一个集合中.但是两个bean的名字都是实现了FactoryBean的对象名称

![image-20220216151046281](/Users/wzw/Desktop/assets/image-20220216151046281.png)



**BeanFactory:bean工厂的顶层接口**

ListableBeanFactory:List性质的bean工厂,可以吧bean对象像枚举一个一个个列出来,而不是像getBean(name)这种形式

ConfigureBeanFactory:配置性质的bean工厂,可以配置这个工厂的各种属性值.

DefaultListableBeanFactory:用的最多的bean工厂



RootBeanDefinition:

GenericBeanDefinition:

MegerdBeanDefinition:

AbstractBeanDefinition:

一级缓存:成品

二级缓存:实例化但没有初始化的对象(包括这个bean的普通对象和代理对象)

三级缓存:用于生成代理对象的lambda表达式



**三级缓存:**

![image-20220126170026556](/Users/wzw/Desktop/assets/image-20220126170026556.png)



beanPostProccessor及其实现

![image-20220215164417788](/Users/wzw/Desktop/assets/image-20220215164417788.png)

- BeanPostProccessor:bean初始化前后分别调用before,after方法
- DestructionAwareBeanPostProccessor方法,包含用于销毁容器之前调用的方法
- MergeBeanDefinitionPostProccessor:要处理合并的beanDefinition(和父类合并)的时候,进行的处理操作.
- InstantiationBeanPostProccessor:包含实例化前后分别要调用的方法
- SmartInstantiationBeanPostProccessor:查找构造器,预测bean类型,解决循环依赖





spring中的观察者模式:

![image-20220215183523550](/Users/wzw/Desktop/assets/image-20220215183523550.png)

![image-20220215183655839](/Users/wzw/Desktop/assets/image-20220215183655839.png)



函数式接口:

![image-20220216174147650](../../Desktop/assets/image-20220216174147650.png)



创建一个bean 的完整过程:

![image-20220216174741021](../../Desktop/assets/image-20220216174741021.png)





lookUpMethod:用于解决单例bean引用原型bean的情况,CommandManager是抽象类,实际的bean是动态代理类,并且注册了拦截器,调用createCommond的时候执行拦截方法,返回指定的bean(command)

如图配置

<img src="../../Desktop/assets/image-20220216215426317.png" alt="image-20220216215426317" style="zoom:50%;" />



![image-20220216215059279](../../Desktop/assets/image-20220216215059279.png)



spring中创建bean 的各种方式:

<img src="../../Desktop/assets/image-20220217142932931.png" alt="image-20220217142932931" style="zoom:50%;" />

使用supplier的方式创建bean,在createBeanInstance方法中使用到

![image-20220217154838279](../../Desktop/assets/image-20220217154838279.png)

factoryMethod方式:

![image-20220217160036531](../../Desktop/assets/image-20220217160036531.png)

![image-20220217160059168](../../Desktop/assets/image-20220217160059168.png)

源码中的流程

![image-20220217163929821](../../Desktop/assets/image-20220217163929821.png)



![image-20220217164004639](../../Desktop/assets/image-20220217164004639.png)

解决循环依赖的问题:

![image-20220219160642175](../../Desktop/assets/image-20220219160642175.png)

![image-20220219161716858](../../Desktop/assets/image-20220219161716858.png)

![image-20220219161834841](../../Desktop/assets/image-20220219161834841.png)

![image-20220219161758793](../../Desktop/assets/image-20220219161758793.png)

为什么一定要设置三级,而不是二级?

如果没有三级缓存,只有二级,在Bpp的after()方法,其中有一个处理器为： `AnnotationAwareAspectJAutoProxyCreator` 其实就是加的注解切面，会跳转到 `AbstractAutoProxyCreator 类的 postProcessAfterInitialization 方法`

![8eb578da6a02b6cdd5e7eca1fdf2b0cd.png](../../Desktop/assets/8eb578da6a02b6cdd5e7eca1fdf2b0cd.png)

![image-20220219163007241](../../Desktop/assets/image-20220219163007241.png)

## 2. 基本启动流程

### 1) refresh函数

#### 1. prepareRefresh():   最前面的准备工作

1. 状态标志 close,active的设定

2. initPropertySource():允许子类扩展,本身什么都不做

   拓展:比如说重写这个方法,在里面设置上系统的属性,然后下一步就可以获取到此处设置的值了

3. getEnviroment().validateRequiredProperties():

   1. 很复杂,new了一个StanderdEnviroment对象,在构造方法中获取了各种系统属性值

   2. 验证属性资源

4.  准备存放监听器和需要发布的事件的集合.



#### 2. ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

概述: 获得刷新的容器对象.

1. refreshBeanFactory( )

刷新一下内部存在的bean工厂

2. 销毁bean并关闭原来的bean工厂

1. 读取bean的定义信息(比较复杂),最终一直到bean的定义信息(包括bean名称,别名被注册 )

   1. 如图,首先是创建一个适配器,实际操作的还是beanFactory

   ![image-20220127160227880](/Users/wzw/Desktop/assets/image-20220127160227880.png)

   2. 设置环境对象和设置资源加载器,生成一个读取配置文件的解析器(本来是要从官网取的,现在本地就可以读取)

   3. 设置配置文件是否要验证

   4. loadBeanDefinitions():

      ![image-20220128122936967](/Users/wzw/Desktop/assets/image-20220128122936967.png)

      然后找到如下重载方法:

      ![image-20220128130440052](/Users/wzw/Desktop/assets/image-20220128130440052.png)

      然后继续进入重载方法:

      ![image-20220128130740045](/Users/wzw/Desktop/assets/image-20220128130740045.png)直到:  

      此时当前线程还没有加载资源,因此currentResouces为0

      ![image-20220128130654999](/Users/wzw/Desktop/assets/image-20220128130654999.png)

      如图处理后,进行实际处理:

      ![image-20220128131603239](/Users/wzw/Desktop/assets/image-20220128131603239.png)

      ​	以下是具体解析逻辑

      ​		![image-20220128155513362](/Users/wzw/Desktop/assets/image-20220128155513362.png)

      ​		![image-20220128155639161](/Users/wzw/Desktop/assets/image-20220128155639161.png)

      **得到Document类型后,开始解析,把东西放到BeanDefinition里面去**

      

      拓展:解析Document的时候是可以解析自定义标签的,具体流程如下

      ![image-20220129165216293](/Users/wzw/Desktop/assets/image-20220129165216293.png)

      

      得到一个解析Document对象的reader对象

      ![image-20220128162713926](/Users/wzw/Desktop/assets/image-20220128162713926.png)

      **略过一些具体的解析步骤代码(从Doc对象的根节点开始,一步步向下解析,其中会根据节点的标签类型,选择具体的解析方式),如图是bean标签类型的解析方法:**

      ![image-20220128163903364](/Users/wzw/Desktop/assets/image-20220128163903364.png)

      	1. parseBeanDefinitionElement方法:

      ![image-20220128164912154](/Users/wzw/Desktop/assets/image-20220128164912154.png)

      2. decorateBeanDefinitionIfRequired():看看是否有属性需要修饰,有的话进行一个修饰工作

      3. 向IOC容器进行一个注册.这里面会进行一些检验工作,具体的检验看源码

         ![image-20220129154534202](/Users/wzw/Desktop/assets/image-20220129154534202.png)

         然后进入默认的bean工厂,找到实现方法

         ![image-20220129154642806](/Users/wzw/Desktop/assets/image-20220129154642806.png)

         1. 如果已经有了这个bean,并且上面有个配置项(同名beanDdefinition能否被覆盖是false的话,就抛出异常,否则进行一些比较并打日志

         2. 判断bean的创建工作是否开始启动

            1. 是:还没看

            2. 否:

               ![image-20220129154132818](/Users/wzw/Desktop/assets/image-20220129154132818.png)

1. return getBeanFactory();

#### 3.prepareBeanFactory():bean工厂的一些属性设置

![image-20220129183612428](/Users/wzw/Desktop/assets/image-20220129183612428.png)

 1. 设置类加载器,就是线程初始化的时候得到一个类加载器

 2. 获取一个EL标签的解析器

 3. 增加一个属性编辑器,对bean的属性进行管理,在后面给bean设置属性的时候会用到

    如图的registerCustomEditors方法

    ![image-20220129184124502](/Users/wzw/Desktop/assets/image-20220129184124502.png)

    自定义属性编辑器:

    

    1. 实现具体的编辑器类,就是工作的类

       ![image-20220130212954922](/Users/wzw/Desktop/assets/image-20220130212954922.png)

    2. 实现注册器,用于向spring注册对应的编辑器

       ![image-20220130213035072](/Users/wzw/Desktop/assets/image-20220130213035072.png)

    3. 让spring识别到这个注册器
    
       ![image-20220207223102462](/Users/wzw/Desktop/assets/image-20220207223102462.png)
    
 4. 新增一个beanProccessor

    ![image-20220207230917928](/Users/wzw/Desktop/assets/image-20220207230917928.png)

    我目前的理解是,beanProccessor像是aop,会在设置属性值的前后分别去做一些事情,而这个applicationContextAwareProccessor就是spring内置的一个beanProccessor,用于判断是否实现了指定的几种aware接口,实现了的话,注入aware

 5. 这些bean会用上面的方式注入

    ![image-20220207231354085](/Users/wzw/Desktop/assets/image-20220207231354085.png)

 6. 相当于@primry

    ![image-20220208162208568](/Users/wzw/Desktop/assets/image-20220208162208568.png)

 7. 在注册一个beanProccessor

    ![image-20220208162411484](/Users/wzw/Desktop/assets/image-20220208162411484.png)

 8. 

    ![image-20220208162448641](/Users/wzw/Desktop/assets/image-20220208162448641.png)

 9. 

    ![image-20220208162854537](/Users/wzw/Desktop/assets/image-20220208162854537.png)

#### 4. postProccessBeanFactory(空的,允许扩展)

接下来是实例化,初始化的准备工作:

#### 5. invokeBeanFactoryPostProccessor():实例化,并执行所有BeanFactoryPostProccessor:这个方法在单例对象实例化之前一定要执行

![image-20220208170606191](/Users/wzw/Desktop/assets/image-20220208170606191.png)

1. 使用委托类,实例化并调用beanFactoryPostProcessor.

   整体流程:

   ![image-20220208173027675](/Users/wzw/Desktop/assets/image-20220208173027675.png)

   1. 创建两个集合分别存放BeanFactoryPostProccessor和BeanDefinitionRegistryPostProcessor

      对方法传入的beanFactoryPostProcessor列表遍历判断,如果是BeanDefinitionRegistryPostProcessor类型,直接执行

      ![image-20220208192554498](/Users/wzw/Desktop/assets/image-20220208192554498.png)

      然后根据类型找到BeanDefinitionRegistryPostProcessor类型**,如果实现PriorityOrdered接口,则准备执行**

      ![image-20220208192843087](/Users/wzw/Desktop/assets/image-20220208192843087.png)

      然后基本一样,实现Ordered接口

      **注意:**

      **重新获取一遍BeanDefinitionRegistryPostProcessor是因为前面执行的过程中可能会新增新的BeanDefinitionRegistryPostProcessor.**

      **但是这个过程中又新增了新的BeanDefinitionRegistryPostProcessor类型且实现PriorityOrdered接口怎么办?**

      **spring内部不包含这种情况,这种情况只会在用户操作中产生,那么应该自定义去扩展,比如下面的递归**

      ![image-20220208194402141](/Users/wzw/Desktop/assets/image-20220208194402141.png)

      最后处理没有实现这两个接口的BeanDefinitionRegistryPostProcessor类型,

      ![image-20220208202223908](/Users/wzw/Desktop/assets/image-20220208202223908.png)

#### 6. registerBeanPostProccessors():实例化并注册所有BeanPostProccessor	

![image-20220215133207758](/Users/wzw/Desktop/assets/image-20220215133207758.png)如上图:得到实现了BeanPostProcessor接口的类(这里的类是在前面获取beanDefinition的时候,里面BeanDefinitionParser实现类,有内置的解析器手动添加了各种类的beanDefinition,因此这里可以获取到)



![image-20220215132503903](/Users/wzw/Desktop/assets/image-20220215132503903.png)

根据BeanPostProcessor的类型分别放到不同的集合里面去,然后排序,然后注册.

![image-20220215170441579](/Users/wzw/Desktop/assets/image-20220215170441579.png)

重新注册ApplicationListenerDetector是为了把它放到BeanPostProcessor的末尾.

#### 7. initMessageSource():初始化消息来源(国际化)

#### 8. initApplicationEventMuticaster():初始化广播器



![image-20220215184511299](/Users/wzw/Desktop/assets/image-20220215184511299.png)

SimpleApplicationEventMulticaster的父类SimpleApplicationEventMulticaster存放了对应的组件,如监听器集合

![image-20220215190224461](/Users/wzw/Desktop/assets/image-20220215190224461.png)

其中的内部类ListenerRetriever中实际存放监听器集合

![image-20220215190242507](/Users/wzw/Desktop/assets/image-20220215190242507.png)

#### 9. onRefresh():也是空的允许扩展

#### 10.registerListeners():注册监听器

![image-20220215190616910](/Users/wzw/Desktop/assets/image-20220215190616910.png)

1. 向多播器注册监听器
2. 

正式实例化:

#### 11. finishBeanFactoryInitialization():  初始化所有非懒加载的单例bean

1. 设置当前上下文 类型转换工作的一个service

   ![image-20220126164044118](/Users/wzw/Desktop/assets/image-20220126164044118.png)

   默认不会进这个if语句,除非自定义这个bean,实现方式:

   ![image-20220216121957028](/Users/wzw/Desktop/assets/image-20220216121957028.png)

   用途:类似与前面的属性编辑器,但是更加灵活,原类型和目标类型甚至可以多对多.(分别是Converter 1:1,GenericConverter 1: n,ConverterFactory n:n)

2. 判断当前beanFactory有没有内置的值处理器,有的话设置一个默认的值处理器

   ![image-20220126164209349](/Users/wzw/Desktop/assets/image-20220126164209349.png)

   这个时候是不会进入if语句的,因为这个类实现了BeanFactoryPostProccessor接口,前面已经实例化并且调用过,作为beanFactory的成员变量了

3. 处理AOP中织入的相关处理工作

   ![image-20220126164418197](/Users/wzw/Desktop/assets/image-20220126164418197.png)

4. 设置临时类读取器(null)

   ![image-20220126164506415](/Users/wzw/Desktop/assets/image-20220126164506415.png)

5. 冰冻配置

   让类的定义信息不允许修改

   ![image-20220126164658575](/Users/wzw/Desktop/assets/image-20220126164658575.png)

6. 真正实例化的操作

   ![image-20220126164717148](/Users/wzw/Desktop/assets/image-20220126164717148.png)

   1. 创建一个bean定义信息的名字的集合

      ![image-20220126164906565](/Users/wzw/Desktop/assets/image-20220126164906565.png)

   2. 循环创建对应名字的单例对象

      1. 返回一个合并好的RootBeanDefinition,遍历父级BeanDefinition,进行一个整合

         ![image-20220126165428535](/Users/wzw/Desktop/assets/image-20220126165428535.png)

      2. 判断非抽象且单例且非懒加载

      3. 判断是否实现FactoryBean接口

         1. 是:

         2. 否,进入doGetBean方法:

            1. 转换bean的名字

               ![image-20220126165837225](/Users/wzw/Desktop/assets/image-20220126165837225.png)

            2. 判断缓存中已经有对应的bean,有的话,直接拿出来用

7. 最后再刷新一下

#### 12. finishRefresh();

## 3. Spring启动流程细节

1. 在创建对应的applicationContext上下文的时候,父类的构造器会创建好需要的各种类,再执行其他工作

   举例:ClassPathXmlApplicationContext读取配置文件的时候,就会先创建好多个父类,,然后解析配置文件,解析的时候用到resoulvePath函数,获取到Enviroment(会获取当前系统环境的各种环境之,属性值),然后解析路径

   ![image-20220127140438047](/Users/wzw/Desktop/assets/image-20220127140438047.png)



### ConfigurationClassPostProccessor

概述:用于处理@Configuration,@ComponentScan@ComponentScans,@Import注解.

类图:![image-20220209130349766](/Users/wzw/Desktop/assets/image-20220209130349766.png)

1. postProcessBeanDefinitionRegistry方法:

   ![image-20220209130423747](/Users/wzw/Desktop/assets/image-20220209130423747.png)

   **processConfigBeanDefinitions方法:**

   1. 对注册的所有beanDefinition做一个筛选

      ![image-20220209161530617](/Users/wzw/Desktop/assets/image-20220209161530617.png)

   2. 排序

      ![image-20220209161559805](/Users/wzw/Desktop/assets/image-20220209161559805.png)

   3. 设置命名规则

      ![image-20220209161703909](/Users/wzw/Desktop/assets/image-20220209161703909.png)

   4. 创建环境和解析器

      ![image-20220209161747935](/Users/wzw/Desktop/assets/image-20220209161747935.png)

   5. 解析

      上述筛选完的beanDefinition每个分别进入parse方法,直到进入processConfigurationClass()方法

      1. 判断是否跳过解析

      2. 判断是否已经被处理过,是的话进行合并

         ![image-20220209162232685](/Users/wzw/Desktop/assets/image-20220209162232685.png)

      3. 对着及其父类进行循环解析,具体解析逻辑如下

         ![image-20220209162336380](/Users/wzw/Desktop/assets/image-20220209162336380.png)

         doProcessConfigurationClass方法:

         1. 递归得到其中的@Component修饰的类,分别调用上面的processConfigurationClass方法

            ![image-20220209162727188](/Users/wzw/Desktop/assets/image-20220209162727188.png)

         2. 判断是否有@PropertySource修饰

            ![image-20220209162805601](/Users/wzw/Desktop/assets/image-20220209162805601.png)

         3. 处理@ComponentScan或者@ComponentScans注解

            ![image-20220209163110916](/Users/wzw/Desktop/assets/image-20220209163110916.png)

         4. 处理@Import

            ![image-20220209171818380](/Users/wzw/Desktop/assets/image-20220209171818380.png)

            如SpringBoot中的自动装配就是这样实现的,在@SpringApplication中就有@Import注解

            找到@Import({AutoConfigurationImportSelector.class}

            ![image-20220216152329222](../../Desktop/assets/image-20220216152329222.png)

            这个类的selectImports方法中找到this.getAutoConfigurationEntry()->this.getCandidateConfigurations()->this.getSpringFactoriesLoaderFactoryClass(),最终到spring.facortries文件找到要自动装配的文件.

            **上述过程有待确认**

            这个类实现了DeferredImportSelector接口.

            ![image-20220216154152085](../../Desktop/assets/image-20220216154152085.png)

            在处理@import注解的时候,会进行进行判断

            1. ImportSelector的子类,如果不是DeferredImportSelector类型,向上递归,如果是,最终统一在下图处理

               ![image-20220216155402278](../../Desktop/assets/image-20220216155402278.png)

            2. 实现了ImportBeanDefinitionRegistrar接口

               ![image-20220216154745535](../../Desktop/assets/image-20220216154745535.png)

            3. 当做普通@configration类进行处理

         5. 处理@ImportResource注解

            ![image-20220209172034440](/Users/wzw/Desktop/assets/image-20220209172034440.png)

         6. 处理加了@Bean注解的方法

            ![image-20220209172100971](/Users/wzw/Desktop/assets/image-20220209172100971.png)

         7. 解析父类

            ![image-20220209172223708](/Users/wzw/Desktop/assets/image-20220209172223708.png)

### 具体的实例化过程:

注意点:



#### 具体流程讲解:

1. 初始化过程

   ![image-20220216134516715](/Users/wzw/Desktop/assets/image-20220216134516715.png)

   1. 得到合并父类的MergedBeanDefinition

      ![image-20220216134758220](/Users/wzw/Desktop/assets/image-20220216134758220.png)

      注意:此时get了,前面一定有一个地方把mergedBeanDefinitions填充了,在哪里?

      答: 前面执行的getBeanNamesForType()方法内部已经调用过getMergedLocalBeanDefinition()方法,此时这个集合里面都是空的,就会进入getMergedBeanDefinition方法.,进入到下图

      ![image-20220216135449634](/Users/wzw/Desktop/assets/image-20220216135449634.png)

      如果没有父类,直接包装成RootBeandefinition并返回

      如果有,递归调用当前方法,直到顶层,此时本身及所有父类都放入到这个mergedBeanDefinitions集合中了.

   2. 开始创建,如果非抽象,单例,非懒加载,则进入创建.
      1. 是FacortyBean类型,执行getBean,进行后续判断
      2. 普通bean,直接调用getBean()

2. 初始化回调过程

![image-20220216134557058](/Users/wzw/Desktop/assets/image-20220216134557058.png)

#### 具体方法详解

1. doGetBean()

   1. 判断一级缓存中是否有这个bean

      ![image-20220216170605317](../../Desktop/assets/image-20220216170605317.png)

   2. 没有的话

      准备开始解决循环依赖,先进行一个判断,如果是原型模式,且在当前线程中正在创建,则抛出异常

      ![image-20220216171053481](../../Desktop/assets/image-20220216171053481.png)

      获取父容器,由于spring中父容器都是空的,因此跳过,springMVC中有相关概念.

      <img src="../../Desktop/assets/image-20220216171233379.png" alt="image-20220216171233379" style="zoom:50%;" />

      表示当前bean已经创建了(为什么不加个标志位表示已经创建而是放入一个集合?->防止每个对象遍历,提高效率)

      ![image-20220216171603105](../../Desktop/assets/image-20220216171603105.png)

      得到MergedBeanDefinition信息,并且进行合法性判断(目前就是抽象的话抛异常)

      ![image-20220216173701611](../../Desktop/assets/image-20220216173701611.png)

      判断是否有依赖的bean,有的话先进行初始化

      ![image-20220216173827751](../../Desktop/assets/image-20220216173827751.png)

      开始创建bean

      ![image-20220217134302535](../../Desktop/assets/image-20220217134302535.png)

2. createBean():

   ![image-20220216200634867](../../Desktop/assets/image-20220216200634867.png)

   ![image-20220217134643898](../../Desktop/assets/image-20220217134643898.png)

**resolveBeanClass**:设置当前bean的beanClass属性,从String类型转变成Class类型

**prepareMethodOverrides**():判断如果bean有replace或者lookup方法,设置一个标志位

![image-20220217135002956](../../Desktop/assets/image-20220217135002956.png)

**Object bean = resolveBeforeInstantiation(beanName, mbdToUse)**:

![image-20220217135100635](../../Desktop/assets/image-20220217135100635.png)

在最上面名词解释有一个InstantiationAwareBeanPostProcessor的beanPostProccessor:就是在实例化前后调用的,此时在此方法中创建这个bean的话,就是不进行后续操作,直接返回.

具体扩展如下:

![image-20220217140637734](../../Desktop/assets/image-20220217140637734.png)





3. doCreateBean

   ![image-20220217144102642](../../Desktop/assets/image-20220217144102642.png)

   ![image-20220217151451141](../../Desktop/assets/image-20220217151451141.png)

   ![image-20220217151510992](../../Desktop/assets/image-20220217151510992.png)

   ![image-20220217151525059](../../Desktop/assets/image-20220217151525059.png)

   ![image-20220217151538914](../../Desktop/assets/image-20220217151538914.png)

   **BeanWrapper instanceWrapper = null;**:bean的包装类

   **instanceWrapper = createBeanInstance(beanName, mbd, args);**:创建实例

   

4. createBeanInstance():创建实例

   ![image-20220217151909579](../../Desktop/assets/image-20220217151909579.png)

   ![image-20220217151947525](../../Desktop/assets/image-20220217151947525.png)

   ![image-20220217152003824](../../Desktop/assets/image-20220217152003824.png)

   使用supplier方式创建bean

   ![image-20220217155107001](../../Desktop/assets/image-20220217155107001.png)

   使用工厂方式创建bean:

   ![image-20220217164443003](../../Desktop/assets/image-20220217164443003.png)

   接下来准备通过反射创建bean

   **下图一般是原型情况才会用到,在第一次创建后会给这些属性赋值,后续从创建的时候可以直接拿过来用,其中获取构造方法的逻辑比较复杂,会根据构造方法的重载情况进行判断,根据参数排序,选中合适的构造方法.**

   ![image-20220217165618727](../../Desktop/assets/image-20220217165618727.png)

   **使用beanPostProccessor,在名词解释中的SmartInstantiationBeanPostProccessor的实现类中找**![image-20220218131750319](../../Desktop/assets/image-20220218131750319.png)

   如AutowiredAnnotationBeanPostProcessor

   ![image-20220218133149850](../../Desktop/assets/image-20220218133149850.png)

   得到@Primary注解修饰的构造器,否则得到全部,并再次进入)autowireConstructor方法进行处理
   
   ![image-20220218134156225](../../Desktop/assets/image-20220218134156225.png)
   
   ![image-20220218134129799](../../Desktop/assets/image-20220218134129799.png)
   
   使用无参构造器进行实例化
   
   ![image-20220218134351110](../../Desktop/assets/image-20220218134351110.png)
   
   下图会使用默认的cglib的策略实例化bean
   
   ![image-20220218140555598](../../Desktop/assets/image-20220218140555598.png)
   
   其中一共有五种方式:
   
   Simple方式的话直接通过反射的方式得到构造方法,cglib方式的话,通过生成bean的代理对象,得到构造方法,然后生成实例
   
   ![image-20220218141658611](../../Desktop/assets/image-20220218141658611.png)
   然后包装成BeanWrapper
   
   ![image-20220218143539889](../../Desktop/assets/image-20220218143539889.png)
   
   ![image-20220218143843853](../../Desktop/assets/image-20220218143843853.png)
   
   因此这个beanWrapper同时有前面说的属性编辑器的注册器和类型转换器的特性,因此可以对包装在里面的bean的属性进行类型转换和属性编辑
   
   ![image-20220218144528405](../../Desktop/assets/image-20220218144528405.png)

# 结尾


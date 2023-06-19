# spring 源码

## 1.BeanDefinition和BeanFactroyPostProcessor

执行一个main方法的时候,spring会调用jvm,读取class文件

1. 但是不会第一时间实例化这些class文件的对象,因为还有很多属性不清楚,比如是否是懒加载,是否只是其他类的property,是否有dependsOn注解等
2. 因此会先生成  BeanDefinition对象,这个对象包含要实例化的对象的所有信息
3. 然后把这个 BeanDefinition对象放到一个map中,(还会把BeanDefinition代表的bean的名字放到一个list里面,后期new的时候根据list遍历,去map里面取)
4. 在对 BeanDefinition对象验证完成后,才进行new ,放到单例map中(注意是单例,不是原型对象,原型对象在getbean的时候new)



---

注意:程序员如果要对这个部分进行扩展,就要用到BeanFactroyPostProcessor接口,使用这个接口的实现类,那么程序就会在,把

BeanDefinition对象所在的map验证完之后,new之间,调用BeanDefinition实现类里面的方法,如图所示

<img src="/Users/wzw/Desktop/assets/image-20201126191654722.png" alt="image-20201126191654722" style="zoom:33%;" />

## 实例化bean的过程

解释一下三个map:

(singletonObject:我们平常理解的spring容器,创建完的.)

singletonFactories ： 单例对象工厂的cache
earlySingletonObjects ：提前暴光的单例对象的Cache
singletonObjects：单例对象的cache

1. 首先根据BeanDefinition所在的map获取bean的名字列表,然后根据名字,进行非lazy的bean的实例化,在这个过程中会进行很多的验证,确定这个类是单例的,非lazy的等等,然后进行普通bean的实例化(getBean方法,但是里面其实也会进行很多的验证比如名字不非法等等),

2. 这个getbean  会直接调用doGetBean方法,这个doGetBean方法里面又有很多方法,其中transformedBeanName方法就用来检测名字是否是合法的,然后就是getSingleton方法(**第一次**  这个方法就是取spring单例池 验证现在要实例化的bean是否已经被实例化,如果是,从一级缓存拿,如果不是,再验证是否正在创建(isPrototypeCurrentlyInCreation方法),如果是,从earlySingletonObjects,如果得到的是null,就去从三级缓存拿  得到工厂->从工厂拿  然后放到二级缓存中,再把工厂里的这个对象移除)

**注意:这个时候为什么要getSingleton方法,明明正在实例化,肯定没有被实例化过()**

​		**为什么需要从二级缓存清除?		:**

​		**为什么不直接从工厂拿,而是先要去二级缓存拿呢?   :从工厂里面取,会进行一系列耗时的步骤,浪费性能,所以把处理好的对象放到二级性能可以节约性能,节省时间**



​		

3. 然后还是各种验证,比如是否dependsOn,是否有方法注入等,很多

4. 验证是单例对象,在这次调用getBean方法时,是会进入的,这里会调用第二次getSingleton方法,如图所示,真正进入前,先进行createBean方法(注意,决定创建这个bean,在这之前,做了个前置处理,把它放到singleCurrentlyInCreation里面,结束后,他取出放到一级缓存也就是单例池里面)),从这里开始,才是这个 bean的生命周期的开始

<img src="/Users/wzw/Desktop/assets/image-20201127160830681.png" alt="image-20201127160830681" style="zoom:33%;" />

1. 先获取这个bean的类型,然后进行一些其他的步骤,比如验证等等,这里没有深入讲,然后进行第一次调用后置处理器(一共会调用九次),然后调用docreatebean方法,进行实例化对象(注意,只是对象不是bean,根据反射获取一个合适的构造方法进行构造,因为此时还没有进行aop的代理,生命周期的初始化回调方法的调用,属性的注入等等)

2. 在第三次调用后置处理器之后,进行一次判断,判断是否允许循环依赖 ,一般情况下是成立的,然后第四次调用后置处理器,判断是否需要进行循环依赖,如果需要,就把他放到一个工厂里面去,然后把这个工厂放到一个map里面

   <img src="/Users/wzw/Desktop/assets/image-20201129141248891.png" alt="image-20201129141248891" style="zoom:33%;" />

   <img src="/Users/wzw/Desktop/assets/image-20201129141248891.png" alt="image-20201129141248891" style="zoom:33%;" />

3. 判断是否需要进行属性的注入,如果需要,**进行属性的注入**,这个过程中进行第五第六次的后置处理器,最后一共进行九次后置处理器的调用,最后放到spring单例池中,才算结束

4. 在属性注入结束后,会进行生命周期回调方法和aware的回调方法,其中初始化回调方法有三种方式:
   注解@postConstruct
   实现Init....Bean(就是初始化bean)这个接口然后重写一个方法
   在xml文件的bean标签里面指定初始化方法

   **优先级从高到低**<img src="/Users/wzw/Desktop/assets/image-20201201180843367.png" alt="image-20201201180843367" style="zoom:33%;" />

5. 接着就是aop了,

   <img src="/Users/wzw/Desktop/assets/image-20201201181009782.png" alt="image-20201201181009782" style="zoom:50%;" />













## 属性注入的过程 populateBean,有网址,挺6

进行属性注入的时候,方法中有一个getBeanPostProcssor方法,用来获取所有的后置处理器,然后用for循环进行遍历,分别调用,完成他们的职能,比如进行代理,进行aop等等,在进行属性注入的过程中,会先把这个要注入的属性去容器中找,如果还没有被实例化,就在进行一次上面的bean的实例化过程,在那个getSingleton方法中,会进行验证,是否正在创建,如果是的从三级缓存去拿,如果不是,继续进行下去,以此递归

可参考以下博客

https://louyuting.blog.csdn.net/article/details/77940767?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control

## 重要的问题

1. **spring是不是默认支持循环依赖:**是,源码中有一个一allowCircularReferences默认为true,表示打开,会在初始化bean生命周期的时候进行一次判断时用到,可以通过几个方法来关闭循环依赖

   1. 修改源码

   2. 在手动初始化容器时,调用初始化方法的默认构造方法,然后手动注册配置文件,修改一些需要的值,再手动refresh()
      如图,非默认的构造方法:

      <img src="/Users/wzw/Desktop/assets/image-20201128145850044.png" alt="image-20201128145850044" style="zoom:33%;" />

      手动修改的过程:

      <img src="/Users/wzw/Desktop/assets/image-20201128145956891.png" alt="image-20201128145956891" style="zoom:33%;" />

   3. 扩展spring

2. **@resource和@Autowired的区别**:进行属性注入的时候,对这两个注解处理的后置处理器不一样

   @resource的后置处理器:CommonAnnotationBeanPostProccessor

   @Autowired的后置处理器:AutowiredAnnotationBeanPostProccessor

3. 三个map的作用:
   


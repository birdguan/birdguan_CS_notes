# Spring面试题

## **Spring框架的七大模块**

* Spring Core：框架的最基础部分，提供IoC容器，对bean进行管理
* Spring Context：继承BeanFactory，提供上下文信息，拓展出JNDI, EJB, 电子邮件，国际化等功能
* Spring DAO：提供了JDBC的抽象层，还提供了声明性事务管理方法
* Spring ORM：提供了JPA, JDO, Hibernate, MyBatis等ORM映射层
* Spring AOP：集成了所有AOP功能
* Spring Web：提供了基础的Web开发的上下文信息，现有的web框架，如JSF, Tapestry, Structs等，提供了集成
* Spring Web MVC：提供了Web应用de Model-View-Controller全功能实现

## **Bean定义5种作用域**

1. singleton（单例）
2. prototype（原型）
3. request
4. session
5. global session

## **Spring IoC初始化流程**

☞ 第一步 Resource定位

Resource是Spring中用于封装I/O操作的接口。正如前面所见，在创建spring容器时，通常要访问XML配置文件，除此之外还可以通过访问文件类型、二进制流等方式访问资源，还有当需要网络上的资源时可以通过访问URL，Spring把这些文件统称为Resource，常用的Resource资源类型如下：

* FileSystemResource：以文件的绝对路径方式进行访问资源，效果类似于Java中的File
* ClassPathResource：以类路径的方式访问资源，效果类似于this.getClass\(\).getResource\("/\).getPath\(""\)
* ServletContextResource：web应用根目录的方式访问资源，效果类似于request.getServletContext\(\).getRealPath\(""\)
* UrlResource：访问网络资源的实现类，如file: http: ftp: 等前缀的资源对象
* ByteArrayResource：访问字节数组资源的实现类

Spring提供了ResourceLoader接口用于实现不同的Resource加载策略，该接口的实例对象中可以获取一个resource对象，也就是说将不同的Resource实例的创建交给ResourceLoader的实现类来处理。

☞ 第二步 通过返回的Resource对象，进行BeanDefinition的载入

1. 什么是BeanDefinition? BeanDefinition与Resource的联系？

官方文档中对BeanDefinition的解释如下：

A BeanDefinition describes a bean instance, which has property values, constructor argument values, and further information supplied by concrete implementations.

Load bean definitions from the specified resource.

总之，BeanDefinition相当于一个数据结构，这个数据结构的生成过程是根据定位的Resource资源对象中的bean而来的，这些bean在Spring IoC容器内部表示成了BeanDefinition这样的数据结构，IoC容器对Bean的管理和依赖注入的实现都是通过操作BeanDefinition来进行的。

2. 如何将BeanDefinition载入到容器？

在Spring中配置文件主要格式是XML，对于用来读取XML型资源文件来进行初始化的IoC容器而言，该类容器会使用到AbstractXmlApplicationContext类，该类定义了一个名为loadBeanDefinitions\(DefalutListableBeanFactory beanFactory\)的方法用来获取BeanDefinition。

☞ 第三步 将BeanDefinition注册到容器中

最终Bean配置会被解析成BeanDefinition并与beanNames, Alias以同封装到BeanDefinitionHolder类中，之后beanFactory.registerBeanDefinition\(beanName, bdHolder.gerBeanDefinition\(\)\)注册到DefaultListableBeanFactory.beanDefinitionMap中。之后客户端如果要获取Bean对象，Spring容器会根据注册的BeanDefinition进行实例化。

## **BeanDefinition加载流程**

定义BeanDefinitionReader解析xml的document，BeanDefinitionDocumentReader解析document成beanDefinition

## **DI依赖注入流程（实例化，处理Bean之间的依赖关系）**

过程在IoC初始化后，依赖注入的过程是用户第一次向IoC容器索要Bean时触发。

* 如果设置lazy-init=true，会在第一次getBean的时候才初始化Bean，lazy-int=false时，会在容器启动的时候直接初始化（singleton bean）
* 调用BeanFactory.getBean\(\)生成bean
* 生成bean过程运用装饰器模式产生的bean都是beanWrapper \(bean的增强\)

☞ 依赖注入如何处理Bean之间的依赖关系

在beanDefinition载入时，如果bean有依赖关系，通过占位符来代替，在调用getBean时候，如果遇到占位符，从IoC里获取bean注入到本实例来。

## **Bean的生命周期**

* 实例化Bean：IoC容器通过获取BeanDefinition对象中的信息进行实例化，实例化对象被包装在BeanWrapper对象中
* 设置对象属性\(DI\)：通过BeanWrapper提供的设置属性的接口完成属性依赖注入
* 注入Aware接口（BeanFactoryAware，可以用这个方法来获取其它Bean，ApplicationContextAware）：Spring会检测该对象是否实现了xxxAware接口，并将相关xxxAware实例注入给bean
* BeanPostProcessor：自定义的处理（分前置处理和后置处理）
* InitializingBean和init-method：执行我们自定义的初始化方法
* 使用
* destory：bean的销毁

## **Spring的IoC注入方式**

* 构造器注入
* setter方法注入
* 注解注入
* 接口注入

## **Soring的循环依赖**

☞ 什么是循环依赖？

循环依赖其实就是循环引用，也就是两个或者两个以上的bean相互持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C依赖于A，如图：

![&#x8FD9;&#x91CC;&#x5199;&#x56FE;&#x7247;&#x63CF;&#x8FF0;](https://img-blog.csdn.net/20170912082357749?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDg1MzI2MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

 Spring中循环依赖场景有：

1. 构造器的循环依赖
2. field属性的循环依赖

☞ 怎么检测是否存在循环依赖？

Bean在创建的时候可以给Bean打标，如果递归调用回来发现正在创建中的话，即说明存在循环依赖。

☞ 如何解决循环依赖？

Spring的单例对象的初始化主要分为三步：（1）CreateBeanInstance实例化（调用对象的构造方法实例化对象）；（2）populateBean填充属性（多bean的依赖属性进行填充）；（3）InitializeBean初始化（调用spring xml中的init方法）

从上面单例bean初始化步骤可以看到，循环依赖主要发生在第一、第二部，也就是构造器循环依赖和filed循环依赖。

在Spring容器整个生命周期内，有且只有一个对象，对象应该存在Cache中，Spring为了解决单例的循环依赖问题，使用了三级缓存。

这三个缓存分别指：

1. singletonObjects：第一级缓存，里面放置的是实例化好的单例对象；
2. earlySingletonObjects：第二级缓存，里面存放的是提前曝光的单例对象
3. singletonFactories：第三级缓存，里面存放的是要被实例化的对象的对象工厂

创建bean的时候Spring首先从一级缓存singletonObjects中获取，如果获取不到，并且对象正在创建中，就再从二级缓存earlySingletonObjects中获取，如果还是获取不到就从三级缓存singletonFactories中取（Bean调用构造函数进行实例化后，即使属性还为填充，就可以通过三级缓存向外提前暴露依赖的引用值，根据对象引用能定位到堆中的对象，其原理是基于Java的引用传递），取到后从三级缓存移动到二级缓存，完全初始化之后将自己放入到一级缓存中供其他使用。

因为加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决。

## **Spring中使用了哪些设计模式？**

* 工厂模式：spring中的BeanFactory就是简单工厂模式的体现，根据传入唯一的标识来获得bean对象；
* 单例模式：提供了全局的访问点BeanFactory
* 代理模式：AOP功能的原理就是使用了代理模式
* 装饰器模式：依赖注入就需要使用BeanWrapper
* 观察者模式：Spring中Observer模式常用的地方是listener的实现，如ApplicationListener
* 策略模式：Bean实例化的时候决定采用何种方式初始化bean实例（反射或者GCLIB动态字节码生成）

## **AOP核心概念**

* 切面（aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象
* 横切关注点：对那些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点
* 连接点（joinpoint）：被拦截到的点，因为spring只支持方法类型的连接点，所以在Spring中连接点指的就是拦截到的方法，实际上连接点还可以是字段或者构造器
* 切入点（pointcut）：对连接点进行拦截的定义
* 通知（advice）：所谓通知指的就是拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类
* 目标对象：代理的目标对象
* 织入（weave）：将切面应用到目标对象并导致代理对象创建的过程
* 引入（introduction）：在不修改代码的前提下，引入可以在运行期为类动态地添加方法或字段

## **解释一下AOP**

传统OOP开发代码逻辑是自上而下地，这个过程中会产生一些横切性问题，这些问题与我们主业务逻辑关系不大，会散落在代码地各个地方，造成难以维护，AOP的思想就是把业务逻辑与横切的问题进行分离，达到解耦的目的，提高代码重用性和开发效率

## **AOP主要应用场景**

* 记录日志
* 监控性能
* 权限控制
* 事务管理

## **AOP源码分析**

* @EnableAspectAutoProxy给容器（beanFactory）中注册一个AnnotationAwareAspectAutoJProxyCreator对象
* AnnotationAwareAspectJAutoProxyCreator对目标对象进行代理对象的创建，对象内部，是封装JDK和CGlib两个技术，实现动态代理对象的创建（创建代理对象的过程中，会先创建一个代理工厂，获取到所有的增强器（通知方法），将这些增强器和目标类注入到代理工厂，再用代理工厂创建对象）
* 代理对象执行目标方法，得到目标方法的拦截器链，利用拦截器的链式机制，依次进入每一个拦截器进行执行

## **AOP使用哪种代理对象**

* 当bean的实现中存在接口或者是Proxy的子类，使用JDK动态代理；不存在接口，Spring采用CGLIB来生成代理对象
* JDK动态代理主要涉及到java.lang.reflect包中的两个类：Proxy和InvocationHandler
* Proxy利用InovationHandler（定义横切逻辑）接口动态创建目标类的代理对象

## **JDK动态代理**

* 通过bind方法建立代理与真实对象关系，通过Proxy.newProxyInstance\(target\)生成代理对象
* 代理对象通过反射invoke方法实现调用真实对象的方法

## **静态代理与动态代理区别** 

* 静态代理：程序运行前代理类的.class文件就存在了
* 动态代理：在程序运行时利用反射动态创建代理对象（复用性、易用性、更加集中）

## **CGLIB和JDK动态代理区别**

* JDK动态代理必须提供接口才能使用
* CGLIB不需要，只要一个非抽象类就能实现动态代理


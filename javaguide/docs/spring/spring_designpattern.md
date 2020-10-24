## 工厂模式
### BeanFactory
一个工厂，里面都是跟获得对象相关的方法
getBean，isType。
Bean factory implementations should support the standard bean lifecycle interfaces as far as possible.
这句话什么意思？
bean lifecycle interface应该是说一个bean的生命周期里面，“从生到死”，我们可以对它进行的一些处理。
就比如，给他做一些初始化，做一些代理，之类的。
主要是一些aware接口，还有initialize，还有beanpostprocessor

### ListableBeanFactory
BeanFactory是一个一个的，这个ListableBeanFactory是一组一组的。
比如说把这个type的所有的bean都取出来，类似的
并且感觉这个ListableBeanFactory新加的方法，除了一个是跟bean相关的，其他都是跟beandefinition相关的
不明白作为一个beanfactory，为什么要跟抽象的beandefinition相关。
因为作为用户来说，beandefinition是对他透明的。
一个解释是，这个用户是spring internal的类，而不是外界用户。

### HierarchicalBeanFactory
多两个方法，一个是找父beanfactory，一个是containsLocalBean，看某个bean是否在自己这层（有啥用）。

* SingletonBeanRegistry
    * 不知道它和beanfactory的差别
    * 感觉像是它是一个特殊的bean的类别的管理，singleton嘛。
    * 而beanfactory是最高抽象层次的，所有bean。

* ConfigurableBeanFactory


* AutowireCapableBeanFactory

* AnnotationConfigRegistry
    * 这种registry都是注册中心
    * 肯定要有的方法是注册，还有必定有容器来维护所有注册进来的东西
* BeanDefinitionRegistry
    * 我觉得这种registry都跟list差不多，跟container差不多
    * 就是一个放东西，取东西的地方
    
* ApplicationContext
    * 像这种什么Context，应该是说，里面有很杂的功能。
    * 感觉像是什么都有
    

通过容器初始化bean的过程，重新梳理逻辑
和里面的一些模式
来理解整个spring的框架
再反过来通过面试题来做一些巩固和查缺补漏

org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory
整个主要是对beanfactory做一些初始化工作，
比如加以下的ApplicationContextAwareProcessor


beanFactory的类型在什么时候确定的
	public GenericApplicationContext() {
		this.beanFactory = new DefaultListableBeanFactory();
	}
这个factory跟这个context是静态绑定的


* DefaultListableBeanFactory： spring默认的beanfactory

org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons
开始初始化所有的singleton
List<String> beanNames = new ArrayList<>(this.beanDefinitionNames); 
//beanfactory里是怎么有这个beandefinition的
应该是GenericApplicationContext在new这个beanfactory的时候，或者后面，在哪里给他塞进去的
org.springframework.context.support.GenericApplicationContext#registerBeanDefinition
是在applicationcontext一开始初始化去读那些beandefinition的时候，一个一个加进beanfactory里的

DefaultListableBeanFactory: getBean(beanName);
org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)
接下来就是对定义里的每个bean，去做beanFactory的getBean操作。
org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
这个doGetBean后面主要就是和DefaultSingletonBeanRegistry的getSingleton通信
其实他们之间都不是组合关系，是继承关系。是紧耦合。就是DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory，
AbstractAutowireCapableBeanFactory通过很多层继承了DefaultSingletonBeanRegistry。
是继承的紧耦合关系。

DefaultSingletonBeanRegistry: getSingleton
这个getSingleton主要有两个参数，
getSingleton(String)
getSingleton(String, ObjectFactory<?>)

getSingleton(String)
从earlySingletonObjects里拿，没有就从singletonFactories里造。
都没有那就没有了。

getSingleton(String, ObjectFactory<?>)
从singletonFactories里造

这里面这个多级缓存主要想解决的是循环依赖
循环依赖的话，这个bean必须要有创建和属性设置要分开的这个前提。
也就是当这个bean A被创建出来后，我先放半成品缓存里，然后就去设置它的属性。
因为走它的属性B的创建过程的时候，也要用到之前这个bean，那它暂时就可以从缓存里面拿出来，再设到自己的属性里。
这样再递归回来的时候，这个属性B已经被创建成功了。所以A就把属性设置上，然后把自己放成品缓存里。
主流程是在AbstractAutowireCapableBeanFactory#createBean
    instanceWrapper = createBeanInstance(beanName, mbd, args); //无参新建实例
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean)); //加进factory缓存
    populateBean(beanName, mbd, instanceWrapper); //
    exposedObject = initializeBean(beanName, exposedObject, mbd);
    
这个AbstractAutowireCapableBeanFactory的createBean就包含了一个bean生命周期里的全部东西，实例化啊，aware啊，beanpostprocessor啊之类的。populateBean, initializeBean


这里面有一个问题就是，为什么要有三级缓存，而不是只有两级。
我们想下如果不要early缓存或者不要factory缓存。
* 只有early缓存
也就是说，在创建A的时候，先直接用factory的create创建出来，放early里面。
然后去填充自己的属性。然后自己的属性B在创建的时候，不是也要依赖A嘛，就去early拿。然后自己好了后就放在一级缓存里。
然后A就填充完属性，也把自己放在一级缓存里，然后把自己从early缓存里拿掉。
这个不是，完美吗。要啥factory缓存？？






































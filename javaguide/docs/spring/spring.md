### autowire
* 指的是这个类的fields的inject方式
    * 比如两个类A和B，然后A的一个成员变量是B
    * 如果是autowire的话，在做A的这个类的singleton的时候，就会把B自动set进去
* 所以spring里的AutowireCapableBeanFactory
    * 指的应该就是用这个beanfacotory create出来的类，它的fields都是自动填充进去的
    
    
### ApplicationContext vs BeanFactory
* 当你有一个配置文件，你可以用来实例化AC，或者直接实例化BeanFactory也行
* 区别
    * BeanFactory就是很轻量级的，一个生产的工厂
    * ApplicationContext其实是BeanFactory的一个丰富的实现
* 具体不同
    * BeanFactory就可以是lazy loading，AC就是全部先给你准备好
    * BeanFacotory就两种scope，AC有很多种
    * BeanPostProcessor
        * AC可以autodetect所有的BeanPostProcessor bean
        * BF需要自己去注册
    * 多个beanpostprocessor，AC可以用Ordered来设置顺序，BF不行
    

### 常用的BeanPostProcessor
* 类别
    * InstantiationAwareBeanPostProcessor
        * 在instantiate前后做点事情
        * 注意BeanPostProcessor是在initialize前后做点事情
    * SmartInstantiationAwareBeanPostProcessor
        * 没有太多资料
        * 但好像spring的aop的proxy就是通过实现这个接口
    * MergedBeanDefinitionPostProcessor
        * 改beandefinition的，比如可以把init-method改掉
* 在spring中的位置
    * InstantiationAwareBeanPostProcessor
        * postProcessBeforeInstantiation
            * 是在createBean中，doCreateBean之前
            * resolveBeforeInstantiation方法里
                * 调用postProcessBeforeInstantiation
                * 如果有结果，就直接返回了。
                    * （也就是说，这个方法可以拦截掉后面所有的spring bean生命周期流程）
                * 如果没结果，就去老实做doCreate
        * postProcessAfterInstantiation
            * 在populateBean里面,propertyset之前
            * doc说可以用来做field injection
            * 理解上，既然是postAfterInstantiate，
                * 也就意味着，在你无参构造实例化之后，然后下一步（properties set）之前
    * MergedBeanDefinitionPostProcessor
        * 在doCreate中
        * instantiate之后，populateBean和initializeBean之前
    * SmartInstantiationAwareBeanPostProcessor
        * getEarlyBeanReference
        * 是在populateBean和initilizeBean之前，会放一个factory进缓存。三级缓存把应该是
        * 主要是为了解决循环依赖。要暴漏出一个半成品bean
        * 但是这个bean如果是原始object的话，后面别人依赖到你就会有问题
        * 所以放一个factory，这个factory的get reference方法是会去调用SmartInstantiationAwareBeanPostProcessor的
        * 比如如果是AOP的话，在这里面，暴漏出去的就是一个AOP已经包装过了的bean
        * AbstractAutoProxyCreator是它的一个实现类


### AbstractBeanFactory.doGetBean
* //依次从一，二，三级缓存中去找这个bean
* sharedInstance = getSingleton(beanName, true); //详细方法见下主题
* if not null //在一，二，三级缓存中有拿到这个bean
    * 就拿出来用
    * 问题：你这样不怕拿出来给用户用的是半成品吗
        * 如果是一级缓存中拿出来的，那就可以直接用，管你
        * 如果是二三级缓存中拿出来的
            * 想想，有可能是用户要用吗
            * 如果是二三级缓存（假设这个类是A），
                * 证明说，A的这个instance肯定是初始化到一半
                * 什么情况下初始化一半又跳到初始化其他实例？
                    * 那肯定是循环依赖
                * 也就是说，是其他的实例的初始化过程，调到这个过程。
            * 所以这个调用过程不会是暴漏给用户，只会是给其他类
* if null //在一，二，三级缓存中没有拿到这个bean
    * 检查dependsOn，有的话，就去处理
    * sharedInstance = getSingleton(beanName, creatBean的ObjectFactory );


#### getSingleton(String beanName, ObjectFactory<?> factory)
* 推断的话，这个方法应该就是用factory的方法去产生这个bean，然后返回
* singletonObject = factory.getObject();
    * 调用的就是底下的createBean方法
* addSingleton(beanName, singletonObject);
    * 从二，三缓存中移除
    * 加进一级缓存
  
  
#### getSingleton(String beanName, boolean allowEarlyReference)   
* in DefaultSingletonBeanRegistry
* 从一级缓存中拿，拿到就返回
* 从二级缓存中拿，拿到就返回
* 从三级缓存（singletonFactories）中拿
    * 加进二级缓存
    * 从三级缓存中删除
    * 返回
* 都没有就返回null


#### createBean
* in class AbstractAutowireCapableBeanFactory
* resolveBeforeInstantiation
    * 调用InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation
* doCreateBean


#### doCreateBean
* instantiate
    * 推断构造函数，比如说用无参构造器反射实例化一个
* applyMergedBeanDefinitionPostProcessors
    * MergedBeanDefinitionPostProcessor
    * 可以修改beandefinition
    * 比如说你可以自己实现一个这个类，把beanname是A的init-method换掉
* addSingletonFactory
* populateBean 
    * 执行InstantiationAwareBeanPostProcessor的
        * postProcessAfterInstantiation
        * postProcessProperties
    * 然后applyPropertyValues:真正把属性注入进去
* initializeBean
    * initilizingBean
    * init-method


#### addSingletonFactory
* 构造一个ObjectFactory放进去
    * ObjectFactory就是用来生产要的这个object的工厂
        * 传进去的参数bean
        * 调用SmartInstantiationAwareBeanPostProcessor.getEarlyReference(bean)
    * AOP的动态代理应该就是在这里做的，不然就没地方做了
* 这个方法主要是让循环依赖的对象能够提前暴漏出来
    * 假设A，B相互依赖，A提前暴漏
    * 这样B在生成的时候，能够从中去拿半成品A，填到自己的属性中。即使A是半成品
        * 半成品的意思是说，还没populatebean，也没initilizebean
        * 但是B持有的这个A的reference的引用是不会变的。
            * 所以B可以先持有，后面A自己去populatebean和initialize的时候，也就是做了B中的那个A的实例
    * 为什么不直接放工厂生成出来的object
        * 因为如果直接放工厂生成出来的object
            * 也是代理过的（SmartInstantiation）
        *


#### populateBean
* InstantiationAwareBeanPostProcessor
    * postProcessAfterInstantiation
* 准备PropertyValues
    * 如果是配置文件中配的会是RunTimeBeanReference
    * 如果是autowire的会是真正的object(bean)
* InstantiationAwareBeanPostProcessor
    * //因为要对property做些手脚，当然要放在这个位置
    * //也就是property准备好后，并且装配进bean之前
    * postProcessPropertyValues 
* applyPropertyValues
    * resolveValueIfNecessary
        * 如果前面是autowire的，那propertyvalue里面直接就是bean了
        * 如果前面是直接在配置文件里配的，那property里是RunTimeBeanReference
            * 用当前的beanfactory去getBean，返回这个bean
    * bw.setPropertyValues //BeanWrapper
    * 用的是PropertyAccessor.setPropertyValues
        * BeanWrapper本身就是PropertyAccessor的实例
        * 这个方法就是把property set进wrappedObject里
    * 这些property还不就是用的浅拷贝。
        * 不同的bean引用beanA，他们的beanA是同一个实例
        * 就是shallow copy，哪有什么deep copy。
        * @?


#### initializeBean
* invokeAwareMethods
    * 某某Aware的意思是，要让这个bean持有某某的引用
    * 比如BeanNameAware, BeanClassLoaderAware, BeanFactoryAware
    * 一般都是执行set**的方法
* BeanPostProcessor
    * postProcessBeforeInitialization
    * 找到注册的所有BPP，调用beforeinit方法
* invokeInitMethods
    * InitializingBean
        * afterPropertiesSet方法
    * initMethodName
        * 反射来调用
* BeanPostProcessor
    * postProcessAfterInitialization
    * 找到注册的所有BPP，调用afterinit方法
    * AOP的动态代理也是在这里做的
        
        
### 循环依赖解决
### Spring循环依赖
* doGetBean
    * 从缓存池中拿
        * 一级缓存有，返回
        * 二级缓存有，返回
        * 三级缓存有
            * 加进二级缓存
            * 从三级换种删除
            * 返回
    * 从缓存池中拿，没有，去造(doCreateBean)
        * instantiate
        * MergedBeanDefinitionPostProessor。改变BeanDefinition
        * addSingletonFactory
            * 这个fatory生产bean的方法是用SmartInstantiationAwareBeanPostProcessor来做
            * //这个方法就提前暴露了这个Bean
        * populateBean
        * initializeBean
        * 返回到doCreateBean
        * 然后从二三缓存中移除，加入第一缓存

* 指导思想
    * 在bean还没创建完，准确来说是populateBean前，提前暴露
        * 在populateBean之前，是因为populateBean要生成的这个bean会依赖我当前的bean
        * 所以一定要在populateBean之前创建
    * 暴露的这个bean必须是经过SmartInstantiationBeanPostProcessor处理的
        * 因为不然你拿的是原始bean的引用，是错误的

* 问题：为什么要用三级缓存，不是二级缓存就够了吗
    * 注意，三级缓存是缓存一个工厂，也就是说，缓存产生bean的**方法**，而不是这个bean本身
    * 如果不用三级缓存，而只用二级缓存，缓存的就得是这个bean本身
        * 也就是每个bean都要先代理出来（SmartInstantiation），再去populateBean
    * 区别
        * 本来是每个bean都要缓存objectfactory，现在是每个bean都要缓存代理后的bean
            * 其实不大知道两者的cost有什么差别


### InitializingBean, init-method, @PostConstruct
* InitializingBean, init-method="myMethod", @PostConstruct
* 顺序
    * 逻辑
        * postconstruct先。因为它是唯一一个跟construct有关系的。当然是construct先。因为逻辑是construct然后才initilize
        * InitializingBean先于init-method。这个没什么好说的
    * 代码
        * CommonAnnotationBeanPostProcessor
        * @PostConstruct是在这个BeanProcessor的postProcessBeforeInitialization里，当然在你们其他的init之前
    
    
### dependency-check
* getDependencyCheck in AbstractBeanDefinition
* dependency check在spring3就被弃用了。
* 本来是用来检查你的property是不是有没有set值的
* 那被弃用了，第一行那个代码是干嘛的

### depends-on
* 可以用来调整bean的启动顺序










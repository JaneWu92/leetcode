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



### 生命周期
* 简要周期：AbstractAutowireCapableBeanFactory#doCreateBean
    * instantiate
        * 推断构造函数，比如说用无参构造器反射实例化一个
    * applyMergedBeanDefinitionPostProcessors
        * MergedBeanDefinitionPostProcessor
        * 可以修改beandefinition
        * 比如说你可以自己实现一个这个类，把beanname是A的init-method换掉
    * addSingletonFactory
    * populateBean 
        * 执行InstantiationAwareBeanPostProcessor的
            * postProcessBeforeInstantiation
            * postProcessAfterInstantiation
        * 然后applyPropertyValues:真正把属性注入进去
    * initializeBean
        * initilizingBean
        * init-method

### populateBean
* 准备PropertyValues
    * 
* applyPropertyValues

### 循环依赖解决






### InitializingBean, init-method, @PostConstruct
* InitializingBean, init-method="myMethod", @PostConstruct
* 顺序
    * 逻辑
        * postconstruct先。因为它是唯一一个跟construct有关系的。当然是construct先。因为逻辑是construct然后才initilize
        * InitializingBean先于init-method。这个没什么好说的
    * 代码
        * CommonAnnotationBeanPostProcessor
        * @PostConstruct是在这个BeanProcessor的postProcessBeforeInitialization里，当然在你们其他的init之前
    
    













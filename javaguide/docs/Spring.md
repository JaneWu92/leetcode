

## 杂
* 无参构造器
```aidl
bean一定要显式地写无参的构造器。
其实不是强制，只是说，如果没有无参构造器，spring就不能通过反射并且不设参地来初始化这个类。
因为如果你没写，其实编译器会默认给你加一个。但这个是在你什么构造器也没写的情况下。
也就是说如果你一旦写了任意一个构造器，那这个默认的无参构造器就没了。
这里涉及到一个问题，spring为什么在反射的时候调用我的无参构造器，我这么多有参的构造器它就不能调这些吗。
这里就是一个问题，在配置bean的情况下，有两种方法来设置参数，一个是有参的构造器，直接constructor来写，一个是通过setter。
也就是说，如果你没写无参构造器，你只能用constructor的方法来构造你的类。也就丢失了setter这种的易用性。就是对你在配置的时候就有限制了。
你就显式写一个无参构造器，配置的时候你就爱咋咋地了。
```
* Spring优势
    * Spring的话主要有两个feature，IOC和AOP
        * IOC
            * 把对象创建和对象的依赖关系的这部分工作交给容器。做够做到很大程度上的解耦
        * AOP
            * 能够解决一个问题：有一些同质的需求要散落到垂直的各个service上，能够有一种方式能够把他们统一起来。对开发和维护有好处。
        * 声明式事务的支持

## SSM框架
* SSM框架
    * Spring
        * DI依赖注入
    * Spring MVC
        * Model
            * 一些service：用户登录.java，订单.java
            * 数据库层：DAO
        * View
            * JSP
        * Controller
            * 不同的Url要mapping到Controller里的哪些Service
    * MyBatis
* 上面的都归Spring容器管理，都是Singleton
* 不归Spring管的，比如pojo，就是prototype的（既然不归spring管，哪还有prototype的概念）

## Spring循环依赖问题
https://zhuanlan.zhihu.com/p/84267654
https://www.cnblogs.com/tiger-fu/p/8961361.html
* 最主要的就是一个问题就是，你的dependency是在实例化的时候就需要，还是可以先no-argument实例化后，再去set这些dependency的field
* 在Spring里，有几个要关注的事情
    * constructor set field: 实例化的时候就需要dependency
    * setter set field：实例化的时候直接no-arg构造器直接实例化起来，然后再去set field
    * singleton： 实例化出来的对象会被放在缓存池里，所以其他人看得到
    * prototype： 实例化出来的对象不会放在缓存池，都是自给自足。
    * 循环依赖的类，只要有一个对象能够被实例化出来，这个环就能被打破。
* 所以，在这个循环依赖里，只要有一个sigleton的类，并且它的构造是用setter的形式的。这些个循环依赖就能被打破。
    * A用无参构造函数构造了，就放在缓存池里。
    * 然后去set它的property，发现有一个B，先去缓存池找，没有，就去实例化B，也是用无参，然后放缓存池。
    * 然后去set B的property，发现是A，去缓存池找，找到了，就set进来。
    * 然后返回给上面调我的A，然后A又set进去。
    * 完成。
* 否则，如果全是prototype，不管你用setter还是constructor去构造，都不能成。
    * 因为prototype即使你用setter去构造，类是实例化出来的，但是它没有这个缓存池的机制。
    * 因为prototype根本不是共享的，是独立的，所以没有缓存池这一说。所以它会一直去调一直去调它的dependency的构造函数。形成一个环。

## 

## 问题
```aidl
connection和threadlocal的关系。
spring里的singleton多线程共享的问题
有状态数据，无状态数据，怎么做。
```

## 反射和代理
### 反射
* 反射是java提供的一个API.
* 能够动态地拿到一个类的所有信息：方法，成员变量，修饰符之类的
* 有什么用？
```aidl
import java.lang.reflect.*;
public class TestMain {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class test = Test.class;
        Field[] fields = test.getFields();
        Method[] methods = test.getMethods();
        Constructor[] constructor = test.getConstructors();
        Test testO = (Test)test.newInstance();
        Method m = test.getMethod("public_method", Integer.class, String.class);
        m.invoke(testO,5,"sdf");
        System.out.println(test);
    }
}
class Test{
    private String private_field;
    public String public_field;
    public Test(){
    }
    private String private_method(Integer i){
        return String.valueOf(i);
    }
    public String public_method(Integer i, String ii){
        System.out.println("in the public method");
        return String.valueOf(i);
    }
}
```
### 代理
* 静态代理和动态代理
* 静态代理是为每个类，每个方法都套一层，有点像装饰模式。其实更准确一点说是每个接口。每个接口都要有一个静态代理类。
* 动态代理是，所有人对我来说，都是“方法”。
* 我可以动态地建类，只要你给我名字。然后我可以在你想要的所有方法前都调用某一个方法。
* 但是这个怎么实现。
#### 静态代理
* 这里用的都是类。但是更准确的应该是Student和StudentProxy都实现一个接口，比如说People。这样StudentProxy和Student就是同质的。
```aidl
package com.jane.learning.spring;
public class StaticProxy {
    public static void main(String[] args) {
        Student s = new Student();
        StudentProxy sp = new StudentProxy(s);
        sp.eat();
        sp.work();
    }
}
class StudentProxy{
    Student s;
    public StudentProxy(Student s){
        this.s = s;
    }
    public void eat(){
        System.out.println("before the action, i caught you!!");
        s.eat();
        System.out.println("after the action, i caught you!!");
    }
    public void work(){
        System.out.println("before the action, i caught you!!");
        s.work();
        System.out.println("after the action, i caught you!!");
    }
}
class Student{
    public void eat(){
        System.out.println("student eat in dining hall");
    }
    public void work(){
        System.out.println("student work in classroom");
    }
}
```
#### 动态代理
* 思考
* 要解决的应该是静态代理的问题：要为每个接口都定制一个proxy类。即使我这个proxy做的事情可能都是一样的，比如说是记日志。
* 静态代理就还是垂直的，而我们现在想要要给水平的，“记日志”这个东西把它抽象出来。
* 看了下面的“JDK动态代理源码”你就可以理解。它是这么做的：
    * Proxy这个类就是接受一个InvocationHandler做参数，这个InvocationHandler里面就是你要做的“记日志”这个抽象。
    * 这个InvocationHandler做的就是在你本来要装饰的那个对象的任意方法的前后，都加上“记日志”的操作
        * 但这里我觉得有点奇怪的是，它必须有“要装饰的那个对象”。但是它作为一个接口是没有成员属性的。设计上就有点费解。
        * 在自己实现这个InvocationHandler这个接口的时候就一定要有持有“要装饰的那个对象”
    * 然后java能够根据你的UserService的接口，和proxy这个类，给你自动写出一个实现类。
    * 你就可以调用它帮你自动化写出的这个实现类，去调用你的work方法或者eat方法。因为在这个类里面，调的方法都是去调invocationhander的方法，就能够用反射，去。。。
    
### JDK动态代理源码
* newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
    * Class<?> cl = getProxyClass0(loader, intfs);
        * ProxyClassFactory: apply(ClassLoader loader, Class<?>[] interfaces)
            * ProxyGenerator去把interfaces变成byte[]的proxy类
            * 然后native Class<?> defineClass0，去把这个byte[]文件加载成类对象放在jvm里。（不知道是怎么实现的，但是就是有这个类）
    * 到了这里，你要代理的那个interface，系统已经帮你实现了一个class proxy implements interface。
    * final Constructor<?> cons = cl.getConstructor(constructorParams); //注意这里的constructorParams是类型，不是实例
    * cons.newInstance(new Object[]{h});
    * 到了上面这里，就是去拿那个defineclass出来的class proxy implements interface，然后去拿constructor去实例化它。并且传进去参数是我们在newProxyInstance时候，传进去的InvocationHandler
    * 这样到这里，就拿到了一个新的对象（注意不是类， 是对象实例）。
    * 这个对象是java帮你自动创建出来的，继承了你想要的接口，并且在接口实现上，是调用你传进去的InvocationHandler。
* 几个问题
    * Proxy类和对象是怎么出来的
        * 类是通过ProxyGenerator.generateProxyClass。会帮你实现一个类。（具体细节未知）
        * class myOwnClass extends Proxy implements UserService
        * protected Proxy(InvocationHandler h) 
        * work: super.h.invoke(this, m3, (Object[])null);
        * ======================
        * 以上是帮忙自动化出来类。
        * 对象的话就是反射，调用构造函数新建出来一个实例。
```aidl
package com.jane.learning.spring.dynamicproxy;
import sun.misc.ProxyGenerator;
import java.io.File;
import java.io.IOException;
import java.lang.reflect.Proxy;
import java.nio.file.Files;
import java.nio.file.Path;

public class DynamicProxy {
    public static void main(String[] args) throws IOException {

        UserService user = new UserServiceImpl();
        MyProxyFactory f = new MyProxyFactory(user, UserService.class);
        UserService p = (UserService) f.getProxy();
        p.work();

        AcountService acount = new AccountServiceImpl();
        MyProxyFactory f2 = new MyProxyFactory(acount, AcountService.class);
        AcountService a = (AcountService) f2.getProxy();
        a.dec();
//        createProxyClass();
    }

    public static void createProxyClass() throws IOException {
        byte[] myOwnClasses = ProxyGenerator.generateProxyClass("myOwnClass", new Class[]{UserService.class});
        Path write = Files.write(new File("my.class").toPath(), myOwnClasses);
    }
}

class MyProxyFactory {
    private Object target;
    private Class<?> clz;
    public MyProxyFactory(Object t, Class<?> c) {
        target = t;
        clz = c;
    }

    public Object getProxy() {
        Object proxyInstance = Proxy.newProxyInstance(DynamicProxy.class.getClassLoader(), new Class[]{clz}, (proxy, method, ars) -> {
            System.out.println("before ...");
            method.invoke(target, ars);
            System.out.println("after ...");
            return null;
        });
        return proxyInstance;
    }
}

interface UserService {
    public void work();
}

class UserServiceImpl implements UserService {
    @Override
    public void work() {
        System.out.println("user works");
    }
}

interface AcountService {
    public void dec();
}

class AccountServiceImpl implements AcountService {
    @Override
    public void dec() {
        System.out.println("Acount dec");
    }
}
```       
```aidl
public final class myOwnClass extends Proxy implements UserService {
    private static Method m1;
    private static Method m3;
    private static Method m4;
    private static Method m2;
    private static Method m0;

    public myOwnClass(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void work() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void eat() throws  {
        try {
            super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.jane.learning.spring.dynamicproxy.UserService").getMethod("work");
            m4 = Class.forName("com.jane.learning.spring.dynamicproxy.UserService").getMethod("eat");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```

### 循环依赖
#### 普通java
* 以下代码会stackoverflow，因为会一直死循环调用对方的constructor
```aidl
public class UserService {
    IndexService indexService;
    public UserService(){
        indexService = new IndexService();
    }

    public static void main(String[] args) {
        UserService s = new UserService();
    }
}
class IndexService{
    UserService userService;
    public IndexService(){
        userService = new UserService();
    }
}
```
* userful blog
    * https://blog.csdn.net/java_lyvee/article/details/101793774


#### Spring三级缓存
* https://www.cnblogs.com/grey-wolf/p/13034371.html

### spring接口增强
* aware接口
BeanNameAware, BeanFactoryAware, ApplicationContextAware
这个从名字上来看，应该是回调。spring的框架在准备你这个bean的时候，在某个过程中，会调用你的这些方法，去传给你一点东西。
这个就是所谓的回调。

* BeanPostProcessor
这个是spring提供的一个接口。也算是回调把。
实现了这个接口的所有类，只要是在spring容器中的。就都会被spring收集起来
然后呢，这些被收集起来的bean post processor，在每个bean的准备过程中，都会被调用，来对bean进行增强。比如aop就是用这个beanpostprocessor做的。

* InitializingBean and DisposalbeBean
前者是在bean属性已经注入后调用，后者是在bean被销毁的时候被调用

* @PostConstruct 和 @PreDestroy
JSR-250

* init-method 和 destroy-method
Spring-specific interfaces

注意，postconstruct是先于InitializingBean的，从InitializingBean的方法afterPropertiesSet就可以看出来。
前者是构造函数调用后，后者是属性注入前。



### Spring AOP

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

instanceWrapper = createBeanInstance(beanName, mbd, args);
//原生对象
exposedObject = initializeBean(beanName, exposedObject, mbd);
//代理对象

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
//到这里有了cglib代理对象


//BeanPostProcessor是什么加入的
org.springframework.context.support.AbstractApplicationContext#registerBeanPostProcessors
org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, org.springframework.context.support.AbstractApplicationContext)
org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanPostProcessor>)
org.springframework.beans.factory.config.ConfigurableBeanFactory#addBeanPostProcessor
org.springframework.beans.factory.support.AbstractBeanFactory#addBeanPostProcessor



### spring bean life cycle
1. instantiate
2. populate properties
3. BeanNameAware callback
4. BeanFactoryAware callback
5. ApplicationContextAware callback
6. pre initialization: BeanPostProcessor
7. afterPropertiesSet of InitializingBean
8. custom init method
9. post initialization: BeanPostProcessors
10. Bean Ready to use


1. instantiate
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(java.lang.String)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
instanceWrapper = createBeanInstance(beanName, mbd, args);
//原生对象

2. populate properties
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
	    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
		PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
//到这里应该是autowired的属性都已经被set了
// ibp = AutoWiredAnnotationBeanPostProcessor。
所以这个意思是@autowire注解也是用beanpostprocessor做的
但是他的postProcessAfterInitialization是return bean，也就是说什么都没做
这里用的是他的postProcessProperties方法，
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties
    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
    metadata.inject(bean, beanName, pvs);
org.springframework.beans.factory.annotation.InjectionMetadata.inject
org.springframework.beans.factory.annotation.InjectionMetadata.InjectedElement.inject
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement.inject
    value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency
org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
        addCandidateEntry(result, candidate, descriptor, requiredType);
            Object beanInstance = descriptor.resolveCandidate(candidateName, requiredType, this);
org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate
    return beanFactory.getBean(beanName);
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(java.lang.String)
//这里就是通过beanfactory再去拿bean的过程。
这里AbstractBeanFactory是DefaultListableBeanFactory类型

3. BeanNameAware callback
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods

4. BeanFactoryAware callback
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods

5. ApplicationContextAware callback

6. pre initialization: BeanPostProcessor
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization
对类中某些特殊方法的调用，比如 @PostConstruct，Aware接口
 
7. afterPropertiesSet of InitializingBean
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeInitMethods
InitializingBean接口，afterPropertiesSet，init-method属性调用

8. custom init method
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeInitMethods
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeCustomInitMethod

9. post initialization: BeanPostProcessors
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
//在这里，才真正有了代理对象（如果你配置了AOP的话）。
//在spring的初始化阶段，会有很多的BeanPostProcessor,但不是全部都有用来增强。大部分的postProcessAfterInitialization这个方法都是直接返回原bean

10. Bean Ready to use

### spring三级缓存解决循环依赖
** 主框架
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean
    Object sharedInstance = getSingleton(beanName); 
        DefaultSingletonBeanRegistry.getSingleton(beanName, true) //一二三级缓存都去找
        //注意这里是一个分水岭。我们以A,B互相依赖为例子。
        //第一次时候，这里拿A是会null，第二次的时候，就不会是null了。因为第一次的时候，就会把factory放进去，所以第二次再从factory拿，就有了。
        //并且从factory中拿后，会放进early缓存里。以便B Bean的创建过程结束后，返回到A接下去的创建过程，A能够再从里面拿出来代理过后的自己。
    if singleton != null 
        返回
    if singleton == null
        sharedInstance = getSingleton(beanName, () -> return createBean(beanName, mbd, args);
        //没有的话就去create：这时候整个工作流程就走到另一个类了
        org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean
    instanceWrapper = createBeanInstance(beanName, mbd, args); //无参构造器构造原生对象
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean)); //加进factory缓存
    populateBean(beanName, mbd, instanceWrapper); //属性注入
    exposedObject = initializeBean(beanName, exposedObject, mbd); //一些初始化，包括代理对象。
        但是这里代理对象应该是在AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsAfterInitialization
        它会调AbstractAutoProxyCreator.postProcessAfterInitialization
        如果它的earlyProxyReferences里没有这个bean，才会去wrap。
        这个earlyProxyReferences里添加这个bean是在用factory缓存里的factory拿bean的时候。
        它的内含逻辑是，如果这个bean已经被earlyrefered过，那refer过它的那个bean在需要这个bean作为属性的时候，
        已经用工厂把他生产出来，放进early缓存了。
        到我现在这一步我就可以不用去wrap，而是去early里拿。如下。
        如果没有循环依赖的话，它才会去wrap。生产出一个代理对象
    earlySingletonReference = getSingleton(beanName, false); //去一二级缓存中找。
    如果有循环依赖的话，会在第二级缓存中找到的，因为在populatebean的时候，已经通过factory把它的所属object生产出来并放在early缓存里了
    exposedObject = earlySingletonReference;
    这里就是代理后的对象，返回代理后的对象。

** 从第三级缓存factory中拿的过程
     
    
然后循环依赖的问题是，A的准备需要B，B的准备又需要A，这样永远没人能够走出去
所以这里用了一个缓存。他的前提是A的准备的两个步骤可以分开，一个是无参构造器的构造，一个是属性的注入。
所以通过无参构造器构造好A之后，就先把它放缓存里，然后去注入它的属性B。然后发现哪里都找不到B，所以，要从无到有，
就走到B的Bean生命周期。
也是一样的步骤，先去B的无参构造构造好，然后放缓存，再去注入属性A，因为之前A已经放缓存里了，这里就可以直接拿到了。
然后B就完成了自己的使命，return到之前A要注入属性B的那一段，然后A也就属性注入完成了。

这里有一个问题，就是缓存的设计。
1. 只有singletonObjects： 
    这里有一个很直接明显的问题，本成品bean可能就直接被用户拿去使用了。
2. singletonObjects + earlyBeanObjects
    半成品放earlyBeanObjects里。成品再放singletonObjects里。这个半成品是，只用了无参构造器构造的实例，其他的步骤都还没做。也就是只做了bean的构造的第一步
    这样子在B的周期里，就能够成功注入A（半成品）。
    问题来了，这时候A的属性注入（bean构造第二步）做完了，它接下去要做其他步骤。在倒数第几步做beanpostprocessor（AOP）的时候，产生了一个新的代理类。
    但是B这个时候不知道，它的field还是之前的原生A。这个就是要解决的问题。
3. 为了解决这个问题，引入了一个Factory缓存。
    参看底下（### singletonFactories）
    也就是这个factory缓存，是能生产出半成品的代理类的。
    但这里有一个问题，其他它可以在要放进factory缓存的时候，就把它生产出来放early缓存里就可以了。为什么还要延迟生产。
    ///////////////////////////
    
### singletonFactories
* 什么时候使用的
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(java.lang.String, boolean)    
    singletonFactory = this.singletonFactories.get(beanName);
	singletonObject = singletonFactory.getObject();
	//这个singletonFactory是下面的“什么时候放进去的”的时候放进去的
	    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getEarlyBeanReference
        org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.getEarlyBeanReference
        Object proxy = this.createProxy(bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        //这里就生成了代理对象
	earlySingletonObjects.put(beanName, singletonObject);
	singletonFactories.remove(beanName);
    
* 什么时候放进去的     
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
     
### earlySingletonObjects
* 什么时候被放进去的
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(java.lang.String, boolean)
在这个方法里面earlySingletonObjects拿不到，并且， allowEarlyReference是true
    singletonObject = singletonFactory.getObject();
    earlySingletonObjects.put(beanName, singletonObject);
    this.singletonFactories.remove(beanName);
//就从factory中新建出来，然后放到early里，并且从factory中移掉。
/////////////////////////////////////////////////////////////////////

上面是在这里被调用的
	public Object getSingleton(String beanName) : DefaultSingletonBeanRegistry
		return getSingleton(beanName, true);

中间插入一下，getsingleton false是在这里
		if (earlySingletonExposure) { ： AbstractAutowireCapableBeanFactory.doCreateBean
			Object earlySingletonReference = getSingleton(beanName, false);
然后true就是会去factory里拿，false就是不去factory里拿

大体就是，一开始A会新建出来，然后把自己放工厂，然后去populate自己的属性
populate 属性B的时候，缓存没找到B，就去新建B。
B会被新建出来，然后把自己放工厂，然后去populate自己的属性
populate属性A的时候，缓存就找到A了，就放进来。再去做其他的initialize，然后就完成了Bean B
return给Bean A，A继续去做initialize，就完成了Bean A。
 
### difference between @Bean and @Component
* @Bean可以导入第三方包的类，@Component不能（不然你就得去人家的源代码上加@Componet）
* @Bean可以加选择逻辑
```aidl
@Bean
@Scope("prototype")
public SomeService someService() {
    switch (state) {
    case 1:
        return new Impl1();
    case 2:
        return new Impl2();
    case 3:
        return new Impl3();
    default:
        return new Impl();
    }
}
```

### Spring bean 创建过程
主要过程是在 abstract autowire capable beanfactory里面
instantiate，实例化，用无参构造函数new一个对象。无参构造器只是其中一个例子，应该是推断构造函数，然后去new对象。
populate bean，属性注入。
initialize，初始化。
    invokeAwareMethods。
        调用bean有实现的一些aware接口，beanNameAware，beanFactoryAware
    applyBeanPostProcessorsBeforeInitialization: @PostConstruct
    invokeInitMethods: 
        InitializingBean的afterPropertiesSe，是直接调用这个方法 -> boolean isInitializingBean = (bean instanceof InitializingBean);
        用反射调用init-method的方法
    applyBeanPostProcessorsAfterInitialization： 像AOP什么的，如果没有循环依赖，不需要earlyexpose，这里就会做aop


### 循环依赖解决方法
主要的思路是，把半成品先放在缓存中暴露出来，这样循环依赖的一环可以被打破，也就是说一个bean在没有准备好的时候，已经可以被引用。
前提是无参构造器的构造和属性的注入，可以被拆分成两步。
一个类先去无参构造出一个对象，然后把bean defination和factory放进factory缓存里
然后去populatebean，就是属性注入，这时要拿的另一个对象，就去走刚才的过程，
先无参构造，然后又要去属性注入第一个对象。这时候它就去缓存里拿A，也就只能从facory缓存里拿，
并且构造出来，放early里。然后就接下去做自己的生命周期。做完后放singletonobject里，返回给A的populatebean
然后A继续往下走，去做自己的其他生命周期，aware接口，applybeanpostprocessbeforeintitialize来做@postconstruct，
然后invokeinit，接口调用afterproperties, 再反射init-method。然后apply bean post proess after initialize
这里就是做aop。但注意，这里如果前面有循环引用的话，这里就不作aop了，而是之前会从early里拿，early已经是aop过后的了。
这里只有不是循环引用的，才会做aop代理。
关于几级缓存，其实两级缓存肯定是需要的，三级就不一定了。可以直接用early缓存，把要放factory的时候，直接把他create出来然后放early就好了。
也就是说，bean要么在singletonobjects里，要么在earlyobjects里























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

























### JDK动态代理
* 动态代理和静态代理的区别
    * 静态代理是编译期改变。代理类的java文件在编译器就要已经准备好。编译器生成.class文件，然后classloader加载进内存。
    * 动态代理是运行期改变。没有代理类的java文件。classloader直接在内存生成proxy的类byte数组。proxy这个类没有任何编译时期的东西。
* 接口
    * java.lang.reflect.Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
* 生成方式
    * 通过ProxyGenerator，是可以动态直接生成代理类的字节码。
    * 我们通过这个生成再保存在class文件中，通过反编译器打开，可以看到它的内容。
    * 就是一个类，implement了你的接口，然后里面的成员变量是它通过反射造出来的被代理类的所有方法。
    * 它的方法就是把你传进来的invocationhandler作用在它通过反射生成的原始的方法上。
* CGLib
    * 也是一样的，只是它是对于类的。JDK动态代理是基于接口的。
    * 它就是proxy extends targetclass。也是就是继承你的target类，然后包裹你的方法。
    
    
    

### 拦截器Intercepter和过滤器filter
* Intercepter是前置增强后置增强都可以，并且它是一个java里面比较普遍的东西。
* filter是只在web编程里的概念。

### spring aop和aspectj的关系
* 是两个关于aop的不同的框架
* aspectj是专门做aop的，所以更强大。spring aop是因为他这个体系用到，所以有涉及到这个东西。
* weaving
    * aspectj: compile-time weaving and post-compile weaving
    * spring: load-time weaving
    * 一个是直接根据soucecode和他自己的东西（也是soucecode）生成class文件，或者是等你生成class文件后，改你的class文件
    * 一个是在compile time什么都没做。在load class文件进内存的时候，直接改你内存里的byte数组，生成他要的类。
* spring后面有集成aspectj的东西，主要可能是他的注解方式把，不大清楚。

### aop
* aspect
    * 你要的在很多地方都要重复的事情，比如加日志
* joinpoint
    * 某一个具体的方法
* pointcut
    * 一个表达式，可以匹配所以你想做aop的方法
* advice
    * 某个方法上具体要做的事情
* weaving
    * 织入方式。是你要额外做的事情（比如加日志）和你的目标业务类，怎么绑定起来的方式

### JdkDynamicAopProxy
```
    public static void main(String[] args) {
        UserService u = new UserServiceImpl();
        u.setName("helloworld");
        UserService p = (UserService)Proxy.newProxyInstance(UserServiceImpl.class.getClassLoader(), u.getClass().getInterfaces(), (proxy, method, argus)-> {
             System.out.println("in proxy and calling : " + method.getName());
             return method.invoke(u, argus);
         });
        p.setName("sf");
        System.out.println(p.getName());
    }
```
去看spring aop出来的类，其实就是一个Proxy，然后他的h field是这个JdkDynamicAopProxy
也就说JdkDynamicAopProxy相当于我们自己写的时候的InvocationHandler
他里面只有一个接口是invoke，在这里面对它代理的object进行一些增强
List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
拿到所有的加在上面的aspect，然后去invoke相应的东西



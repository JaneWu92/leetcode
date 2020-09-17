

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
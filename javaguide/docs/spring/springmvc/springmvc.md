### 原始MVC的比较
本来的MVC（就是跟spring没有关系）是：
View直接接受用户的请求，然后去调用相对应的controller，controller再去调用model（pojo和service）去做相对应的逻辑。
然后model直接把数据返回给view。
现在的spring是多加了一层front controller。
由这个front controller来接受用户的请求。把view和MC解耦开来
controller和model之间的关系还是一样。
但是现在返回的数据是由controller返回给front controller，然后front controller再去给View
* 这个front controller说的就是DispatcherSevlet

### 配置
* web.xml
    * 配置DispatcherSevlet。用来拦截所有的请求
        * by default的
        * 配置springmvc.xml。这个配置文件是spring core的那套。IOC，AOP
        * 配置url-pattern: / ==》 就是拦截所有的请求
* index.jsp
    * 这个是默认的你lauch一个app如果没写url它自己会去launch的页面。
    * 在这个index.jsp里，就写href="helloservice"
    * 是一个超链接，会去发送someservice的request
* HelloWorldController
    * 这个controller会有@Controller的标签
    * 它会有一些方法，方法上会有@RequestMapping的标签。就是指这些方法是对应要回应你什么样的“request”
    * 比如这里写一个@RequestMapping("helloservice")
    * 那么上述index.jsp的那个href就会由这个service来做。
    * 这个service就做一些事情，再返回一个String: /WEB-INF/pages/success.jsp
    * 就是做redirect到另外一个页面。一个完整的请求就结束了
* 视图解析器：InternalResourceViewResolver
    * 配在spring的core xml里，这样就可以不用每次返回/WEB-INF/pages/success.jsp
    * 可以把prefix和suffix配在里面
    * 这样controller在返回字符串作为redirect或者forward的时候，就可以省略掉前后缀，直接写“success”
    

### redirect vs forward
可能是redirect是到另外一个controller，而forward只是到另外一个页面而已
然后前者是还不返回给用户，后者是直接返回给用户了。
好像也不对，两者应该都是要被front controller，也就是DispatcherSevlet拦截处理把
===========
redirect是服务端先返回一个response给客户端，客户端拿到response再重新发一个request给服务端
所以在外部表现来看，浏览器的地址栏会有两个地址
forward是在服务端内部，直接把request给另外一个controller，另外一个controller处理完再返回response给客户端。
所以在外部表现看来，浏览器的地址栏只会有一个地址。


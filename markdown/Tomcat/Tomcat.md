## Tomcat 中为什么要使用自定义类加载器？
一个 Tomcat 中可以部署多个应用，而每个应用中都存在很多类，并且各个应用中的类是独立的，而全类名是可以相同的。
比如一个订单系统中可能存在 com.xxx.User 类，一个库存系统中可能也存在 com.xxx.User 类，
一个 Tomcat 不管内部部署了多少应用，Tomcat 启动之后就是一个 Java 进程，也就是一个 JVM，
所以如果 Tomcat 中只存在一个类加载器，比如默认的 AppClassLoader。
那么就只能加载一个 com.xxx.User 类，这是有问题的。  
而在 Tomcat 中，会为部署的每个应用都生成一个类加载器，名字叫做 WebAppClassLoader。
这样 Tomcat 中每个应用就可以使用自己的类加载器去加载自己的类，从而达到应用之间的类隔离，不出现冲突。  
另外 Tomcat 还利用自定义加载器实现了热加载功能。

（JVM 如何区分类？  
类的唯一标识：类的加载实例 + 类的全限定类名。
所以 Tomcat 可以达到为每个应用使用自己的类加载器去加载同全限定名的类而出现不冲突的效果：  
WebAppClassLoaderA 类加载实例 + 全限定类名  
WebAppClassLoaderB 类加载实例 + 全限定类名）

## Tomcat 如何进行优化？
对于 Tomcat 调优，可以从两个方面来进行调整：内存和线程。  
首先启动 Tomcat，实际上就是启动了一个 JVM，所以可以按 JVM 调优的方式来进行调整。从而达到 Tomcat 优化的目的。  
另外 Tomcat 中设计了一些缓存区，比如 appReadBufSize，bufferPoolSize 等缓存区来提高吞吐量。  
还可以调整 Tomcat 的线程，比如调整 minSpareThreads 参数来改变 Tomcat 空闲时的线程数。  
调整 maxThreads 参数来设置 Tomcat 处理连接的最大线程数。并且还可以调整 IO 模型，比如使用 NIO、APR 这种相比于 BIO 更加高效的 IO 模型。 

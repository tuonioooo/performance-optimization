# Tomcat调优篇实战

* ## 优化内存

/bin/catalina.sh添加JAVA\_OPTS参数  
** jdk1.7**

```
JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms512m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=1024M -XX:PermSize=1024m -XX:MaxPermSize=1024m -XX:+DisableExplicitGC"
```

> 1.8版本中已经没有PermSize、MaxPermSize

```
JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms512m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=1024M -XX:+DisableExplicitGC"
```

> 参数说明
>
> -Djava.awt.headless：没有设备、键盘或鼠标的模式。有关介绍：[What is Headless mode in Java](https://link.jianshu.com?t=https://blog.idrsolutions.com/2013/08/what-is-headless-mode-in-java/)
>
> -Dfile.encoding： 设置字符集
>
> -server：jvm的server工作模式，对应的有client工作模式。使用“java -version”可以查看当前工作模式
>
> -Xms512m：初始Heap大小，使用的最小内存
>
> -Xmx1024m：Java heap最大值，使用的最大内存
>
> -XX:NewSize=512m：表示新生代初始内存的大小，应该小于 -Xms的值
>
> -XX:MaxNewSize=1024M：表示新生代可被分配的内存的最大上限，应该小于 -Xmx的值
>
> -XX:PermSize=1024m：设定内存的永久保存区域\(注：jdk1.8 was removed\)
>
> -XX:MaxPermSize=1024m：设定最大内存的永久保存区域\(注：jdk1.8 was removed\)
>
> -XX:+DisableExplicitGC：自动将System.gc\(\)调用转换成一个空操作，即应用中调用System.gc\(\)会变成一个空操作

* ## 优化连接数

### 1）优化线程数

在conf/server.xml找到Connectorport="8080" protocol="HTTP/1.1"，增加maxThreads和acceptCount属性（使acceptCount大于等于maxThreads），如下：

```
<Connectorport="8080" protocol="HTTP/1.1"connectionTimeout="20000" redirectPort="8443"acceptCount="500" maxThreads="400" />
```

> 参数说明：
>
> maxThreads：tomcat可用于请求处理的最大线程数，默认是200
>
> minSpareThreads：tomcat初始线程数，即最小空闲线程数
>
> maxSpareThreads：tomcat最大空闲线程数，超过的会被关闭
>
> acceptCount：当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理.默认100

### 2）使用线程池

在conf/server.xml中增加executor节点，然后配置connector的executor属性，如下：

```
<Executorname="tomcatThreadPool" namePrefix="req-exec-"maxThreads="1000" minSpareThreads="50"maxIdleTime="60000"/>
<Connectorport="8080" protocol="HTTP/1.1"executor="tomcatThreadPool"/>
```

> 参数说明：
>
> namePrefix：线程池中线程的命名前缀
>
> maxThreads：线程池的最大线程数  默认150
>
> minSpareThreads：线程池的最小空闲线程数 默认4
>
> maxIdleTime：超过最小空闲线程数时，多的线程会等待这个时间长度，然后关闭
>
> threadPriority：线程优先级

### 3）优化运行模式（connector for protocol）

* **BIO**

一个线程处理一个请求。缺点：并发量高时，线程数较多，浪费资源。Tomcat7或以下在Linux系统中默认使用这种方式。

配置conf/server.xml

```
<Connector executor="tomcatThreadPool" port="8080" protocol="HTTP/1.1" connectionTimeout="20000" enableLookups="false" redirectPort="8443" URIEncoding="UTF-8" />
```

* **NIO**

利用Java的异步IO处理，可以通过少量的线程处理大量的请求。Tomcat8在Linux系统中默认使用这种方式。Tomcat7必须修改Connector配置来启动（conf/server.xml配置文件）。

配置conf/server.xml

```
<Connector executor="tomcatThreadPool" port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" connectionTimeout="20000" enableLookups="false" redirectPort="8443" URIEncoding="UTF-8" />
```

* ## tomcat中**如何禁止和允许列目录下的文档 **

在{tomcat\_home}/conf/web.xml中，把listings参数配置成false即可，如下：

```
<servlet> 
... 
<init-param> 
<param-name>listings</param-name> 
<param-value>false</param-value> 
</init-param> 
... 
</servlet>
```

* ## tomcat中**如何禁止和允许主机或IP地址访问**

```
<Host name="localhost" ...> 
  ... 
  <Valve className="org.apache.catalina.valves.RemoteHostValve" 
         allow="*.mycompany.com,www.yourcompany.com"/> 
  <Valve className="org.apache.catalina.valves.RemoteAddrValve" 
         deny="192.168.1.*"/> 
  ... 
</Host>
```

* ## 一个真实有效的高并发Tomcat优化配置示例：

```
<Executor name="tomcatThreadPool"        # 配置TOMCAT共享线程池，NAME为名称　
          namePrefix="HTTP-8088-exec-"    # 线程的名字前缀，用于标记线程名称
          prestartminSpareThreads="true"  # executor启动时，是否开启最小的线程数
          maxThreads="5000"               # 允许的最大线程池里的线程数量，默认是200，大的并发应该设置的高一些，这里设置可以支持到5000并发
          maxQueueSize="100"              # 任务队列上限
          minSpareThreads="50"            # 最小的保持活跃的线程数量，默认是25.这个要根据负载情况自行调整了。太小了就影响反应速度，太大了白白占用资源
          maxIdleTime="10000"             # 超过最小活跃线程数量的线程，如果空闲时间超过这个设置后，会被关别。默认是1分钟。
 />
```

```
<Connector port="8088" protocol="org.apache.coyote.http11.Http11NioProtocol"
   connectionTimeout="5000" redirectPort="443" proxyPort="443" executor="tomcatThreadPool"  
   URIEncoding="UTF-8"/> # 采用上面的共享线程池
```

参考文档：

[http://tomcat.apache.org/tomcat-8.5-doc/config](http://tomcat.apache.org/tomcat-8.5-doc/config/index.html)

[http://tomcat.apache.org/tomcat-8.0-doc/config](#)

[http://tomcat.apache.org/tomcat-7.0-doc/config](#)


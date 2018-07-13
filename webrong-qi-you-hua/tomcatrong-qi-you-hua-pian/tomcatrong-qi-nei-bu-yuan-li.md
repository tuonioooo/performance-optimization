# Tomcat概念及原理 {#tomcat概念及原理}

Servlet\(server applet 服务端小程序\)是一种国际组织的协议、约定.  
Tomcat只是针对Servlet协议的规范做了封装，其他这样的软件还有jetty等.

## Tomcat总体结构 {#tomcat总体结构}

Server –&gt; Service –&gt;  
Connector & Container\( Engine –&gt; Host –&gt; Context\( Wrapper\( Servlet \) \) \)

![](/assets/import-tomcat01.png)

Tomcat 的心脏是两个组件：Connector 和 Container，Connector 组件是可以被替换，这样可以提供给服务器设计者更多的选择，因为这个组件是如此重要，不仅跟服务器的设计的本身，而且和不同的应用场景也十分相关，所以一个 Container 可以选择对应多个 Connector。

多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了。所以整个 Tomcat 的生命周期由 Server 控制。

### Server {#server}

前面说一对情侣因为 Service 而成为一对夫妻，有了能够组成一个家庭的基本条件，但是它们还要有个实体的家，这是它们在社会上生存之本，有了家它们就可以安心的为人民服务了，一起为社会创造财富。

Server 要完成的任务很简单，就是要能够提供一个接口让其它程序能够访问到这个 Service 集合、同时要维护它所包含的所有 Service 的生命周期，包括如何初始化、如何结束服务、如何找到别人要访问的 Service。还有其它的一些次要的任务，如您住在这个地方要向当地政府去登记啊、可能还有要配合当地公安机关日常的安全检查什么的。

![](/assets/import-tomcat02.png)

![](/assets/import-tomcat03.png)

### Service {#service}

我们将 Tomcat 中 Connector、Container 作为一个整体比作一对情侣的话，Connector 主要负责对外交流，可以比作为 Boy，Container 主要处理 Connector 接受的请求，主要是处理内部事务，可以比作为 Girl。那么这个 Service 就是连接这对男女的结婚证了。是Service 将它们连接在一起，共同组成一个家庭。当然要组成一个家庭还要很多其它的元素

说白了，Service 只是在 Connector 和 Container 外面多包一层，把它们组装在一起，向外面提供服务，一个 Service 可以设置多个 Connector，但是只能有一个 Container 容器。这个 Service 接口的方法列表如下:

![](/assets/import-tomcat04.png)

### Container {#container}

Container是一个接口，定义了下属的各种容器，尤其是Wrapper、Host、Engine、Context

![](/assets/import-tomcat05.png)

### Engine {#engine}

负责处理来自相关联的service的所有请求，处理后，将结果返回给service，而connector是作为service与engine的中间媒介出现的。  
一个engine下可以配置一个默认主机，每个虚拟主机都有一个域名。当engine获得一个请求时，它把该请求匹配到虚拟主机\(host\)上，然后把请求交给该主机来处理。  
Engine有一个默认主机，当请求无法匹配到任何一个虚拟主机时，将交给默认host来处理。Engine以线程的方式启动Host。

![](/assets/import-tomcat06.png)

### Host {#host}

代表一个虚拟主机，每个虚拟主机和某个网络域名（Domain Name）相匹配。  
每个虚拟主机下都可以部署一个或多个web应用，每个web应用对应于一个context，有一个context path。  
当Host获得一个请求时，将把该请求匹配到某个Context上，然后把该请求交给该Context来处理匹配的方法是“最长匹配”，所以一个path==””的Context将成为该Host的默认Context所有无法和其它Context的路径名匹配的请求都将最终和该默认Context匹配。

### Context {#context}

一个Context对应于一个Web应用，一个Web应用由一个或者多个Servlet组成Context在创建的时候将根据配置文件$CATALINA\_HOME/conf/web.xml和$WEBAPP\_HOME/WEB-INF/web.xml载入Servlet类。当Context获得请求时，将在自己的映射表\(mapping table\)中寻找相匹配的Servlet类，如果找到，则执行该类，获得请求的回应，并返回。

### Wrapper {#wrapper}

Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。Wrapper 是最底层的容器，它没有子容器了，所以调用它的 addChild 将会报错。  
Wrapper 的实现类是 StandardWrapper，StandardWrapper 还实现了拥有一个 Servlet 初始化信息的 ServletConfig，由此看出 StandardWrapper 将直接和 Servlet 的各种信息打交道。

### Connector {#connector}

Connector将在某个指定的端口上来监听客户的请求，把从socket传递过来的数据，封装成Request，传递给Engine来处理，并从Engine处获得响应并返回给客户。

Tomcat通常会用到两种Connector：

1. Http Connector 在端口8080处侦听来自客户browser的http请求。 AJP Connector
2. 在端口8009处侦听来自其它WebServer\(Apache\)的servlet/jsp代理请求。

### Lifecycle {#lifecycle}

现实生活中大部分的事物都有生命周期，就像人的生老病死一样。

在编程中也有很多对象是具有生命周期的，从初始化、运行、回收等 会经历几个不同的阶段。 在tomcat中容器相关的好多组建都实现了Lifecycle接口，当tomcat启动时，其依赖的下层组件会全部进行初始化。 并且可以_**对每个组件生命周期中的事件添加监听器**_。

例如当服务器启动的时候，tomcat需要去调用servlet的init方法和初始化容器等一系列操作，而停止的时候，也需要调用servlet的destory方法。而这些都是通过org.apache.catalina.Lifecycle接口来实现的。由这个类来制定各个组件生命周期的规范。

![](/assets/import-tomcat07.png)

下图是生命周期的所有状态

![](/assets/import-tomcat08.png)

### LifecycleListener {#lifecyclelistener}

在Lifecycle的介绍中提到，Lifecycle会对每个组件生命周期中的事件添加监听器，也就是addLifecycleListener\(LifecycleListener listener\)方法，而LifecycleListener就是上面提到的监听器。

### LifecycleEvent {#lifecycleevent}

顾名思义，就是当有监听事件发生的时候，LifecycleEvent会存储时间类型和数据

```
/**
* Construct a new LifecycleEvent with the specified      parameters.
*
* @param lifecycle Component on which this event occurred
* @param type Event type (required)
* @param data Event data (if any)
*/
public LifecycleEvent(Lifecycle lifecycle, String type, Object data) {
   super(lifecycle);
   this.type = type;
   this.data = data;
}
```

## Tomcat启动过程 {#tomcat启动过程}

以嵌入式的tomcat-embed-core:8.5.6为例，以编程的方式启动tomcat，可以看到调用了server.start\(\)

Tomcat的start\(\)方法

```
/**
 * Start the server.
 *
 * @throws LifecycleException Start error
 */
public void start() throws LifecycleException {
    getServer();
    getConnector();
    server.start();
}
```

server的默认实现是StandardServer, start\(\)方法在其父类LifecycleBase中

![](/assets/import-tomcat09.png)

_**声明：**_除了StandardServer, 这个类也是StandardService, StandardEngine, StandardHost, StandardContext的父类，所以当调用这些类的start\(\)方法时其实都是调用此处的start\(\)方法，而最重要的是在start\(\)方法中会调用startInternal\(\)，startInternal\(\)在LifecycleBase中是抽象方法，具体实现由各实现类去做，在此后不再赘述。

### 启动server {#启动server}

启动server其实就是启动service容器

![](/assets/import-tomcat10.png)

### 启动service {#启动service}

启动service其实就是启动engine容器和connector容器

![](/assets/import-tomcat11.png)

### 启动Engine {#启动engine}

启动engine其实就是启动host容器\(多线程\)

![](/assets/import-tomcat12.png)

### 启动Host {#启动host}

启动Host的方式和上图一样，都是以线程的方式启动子Container，这里Host的children为Context

### 启动Context {#启动context}

启动Wrapper![](/assets/import-tomcat13.png)

下面看一下非常重要的一个方法 loadServlet:

![](/assets/import-tomcat14.png)

![](/assets/import-tomcat15.png)

![](/assets/import-tomcat16.png)

它基本上描述了对 Servlet 的操作，当装载了 Servlet 后就会调用 Servlet 的 init 方法，同时会传一个 StandardWrapperFacade 对象给 Servlet，这个对象包装了 StandardWrapper，ServletConfig 与它们的关系图如下：  
![](/assets/import-tomcat17.png)

### 启动Connector {#启动connector}

Tomcat的Connector是Coyote connector的一种实现，这是tomcat的官方解释：The Coyote HTTP/1.1 Connector element represents a Connector component that supports the HTTP/1.1 protocol. It enables Catalina to function as a stand-alone web server, in addition to its ability to execute servlets and JSP pages.  
Tomcat8之后默认使用nio作为接受请求策略，默认在Service启动的时候进行初始化，当然也可以单独启动，在默认的构造函数中会初始化ProtocolHandler

![](https://img-blog.csdn.net/20170302150526227?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

tomcat中支持两种协议的连接器：HTTP/1.1与AJP/1.3

HTTP/1.1协议负责建立HTTP连接，web应用通过浏览器访问tomcat服务器用的就是这个连接器，默认监听的是8080端口；

AJP/1.3协议负责和其他HTTP服务器建立连接，监听的是8009端口，比如tomcat和apache或者iis集成时需要用到这个连接器。

![](https://img-blog.csdn.net/20170302151012422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

Connector的启动其实就是ProtocolHandler的启动，如下图：

![](https://img-blog.csdn.net/20170302152520161?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

ProtocolHandler的类结构如下图：

![](https://img-blog.csdn.net/20170302153301260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

Connector的startInternal方法调用了ProtocolHandle的start方法，这个start方法就在AbstractProtocol中，如下图：

![](https://img-blog.csdn.net/20170302153904961?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

经历了这么多，终于快到终点了，这时候又出来了一个EndPoint，不过不要急，这个EndPoint已经算是终点了，EndPoint是咱们Tomcat启动的Socket管理者\(注意：通过类图可以看出AbstractEndpoint已经脱离了Lifecycle和LifecycleListener体系，所以它只是一个简简单单的Socket管理者\)，因为是由他直接启动默认的Nio，在启动的时候先看看类结构图：

![](https://img-blog.csdn.net/20170302154444437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

再看一下EndPoint能做什么，看方法就知道了

![](https://img-blog.csdn.net/20170302154549407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

createAcceptor、createExecutor等方法都是在初始化EndPoint很重要方法，因为在接收请求的时候，通过Acceptor的接收，经过重重模块，才能一路到达最后的Servlet，这个在后面会有讲到。

那么EndPoint最后的启动，看下图：

![](https://img-blog.csdn.net/20170302155014429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VueXVuamllMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

在这个地方会启动很多的线程，第一次读这段代码的同学可能会有点乱，因为有想过启动线程，现在还有些不知道是为什么启动，这就是tomcat的有意思之处了，因为下一篇我会说到tomcat接收请求的流程，所以在下一篇文章我会详细讲解。

好了，大家，tomcat 的启动流程大概就是这样，当然这里面还有一些更细节的知识，此文章适合第一次想要了解Tomcat的同学，当然以后我也会一直完善这篇文章，有看到有意思的部分我也会补充的。


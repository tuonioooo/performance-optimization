# WEB程序容器结构剖析



开发一个web容器涉及很多不同方面不同层面的技术，例如通信层的知识，程序语言层面的知识等等，且一个可用的web容器是一个比较庞大的系统，要说清楚需要很长的篇幅，本文旨在介绍如何设计一个web容器，只探讨实现的思路，并不涉及过多的具体实现。把它分解划分成若干模块和组件，每个组件模块负责不同的功能，下图列出一些基本的组件，并将对每个组件进行介绍。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152065.jpg "web容器,设计,JSP")



**连接接收器**

  
主要的职责就是监听是否有客户端套接字连接并接收socket，再将socket交由任务执行器（线程池）执行。不断从系统底层读取socket，做尽可能少的处理，再扔进线程池。为什么强调要做尽可能少的处理？这里关系到系统性能问题，过多的处理会严重影响吞吐量。因为一般只有一个接收器（一条线程负责套接字接收工作），所以它对每次接收处理的时间长短将很可能对整体性能产生影响。于是接收器所干的活都是非常少且简单的，仅仅维护了几个状态变量、流量控制闸门的累加操作、serverSocket的接收操作、设置接收到的socket的一些属性、将接收到的socket放入线程池以及一些异常处理。其他需要较长时间处理的逻辑就交给了线程池，例如对socket底层数据的读取，对http协议报文的解析及响应客户端的一些操作等等。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152170.jpg "web容器,设计,JSP")



  
**连接数控制器**

  
对于一台机器而言，访问请求的总流量有高峰期且服务器有物理极限，为了保证web服务器不被冲垮我们需要采取一些措施进行保护预防，需要稍微说明的此处的流量更多的是指套接字的连接数，通过控制套接字连接个数来控制流量。其中一种有效的方法就是采取流量控制，它就像在流量的入口增加了一道闸门，闸门的大小决定了流量的大小，一旦达到最大流量将关闭闸门停止接收直到有空闲通道。计数器可用JDK的AQS框架实现。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152171.jpg "web容器,设计,JSP")



  
**套接字工厂**

  
不同的使用场合可能需要不同的安全级别，例如在支付相关的交易就必须对信息加密后再发送，这其中还涉及到密钥协商的过程，而在另外一些普通场合则无需对报文加密。反应到应用层则是使用http与https的问题。  
简单讲TLSSSL协议给每次通信①提供认证服务，认证本次会话实体身份的合法性。②提供加密服务，强加密机制能保证通信过程中的消息不会被破译。③提供防篡改服务，利用Hash算法对消息进行签名，通过验证签名保证通信内容不被篡改。  
http协议对应Socket，而https则对应SSLSocket。如何生成Socket及SSLSocket则交由套接字工厂。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152172.jpg "web容器,设计,JSP")



  
**任务定义器——Task**

  
定义需要执行的任务，告诉线程池要执行什么样的任务。任务主要分为三点：处理socket并响应客户端、连接数计数器减一、关闭socket。其中对socket的处理是最重要也是最复杂的，它包括对底层socket字节流的读取、http协议请求报文的解析（请求行、请求头、请求体等信息的解析）、根据请求行解析得到路径去寻找相应主机上web项目的资源、根据处理的结果组装好http协议响应报文输出到客户端。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152173.jpg "web容器,设计,JSP")

  
**任务执行器**

  
一个拥有最大最小线程数限制的线程池，之所以称之为“任务执行器”是因为线程池可以看做是启动了若干线程不断检测某个任务队列，一旦发现有需要执行的任务则执行。最大最小线程数限制、多余线程回收时间限制、超出最大线程数时线程池做出的拒绝动作等等。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152174.jpg "web容器,设计,JSP")

  
**报文读取**

  
用于向操作系统底层读取来自客户端的报文并提供缓冲机制。报文复制到desBuf。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152175.jpg "web容器,设计,JSP")

  
**报文输出**

  
用于向操作系统底层写入由web容器处理后的报文并提供缓冲机制。将报文outputBuf通过缓冲区写入到操作系统。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152176.jpg "web容器,设计,JSP")

  
**输入过滤器**

  
在这个读取的过程中希望做一些额外的处理，并且这些额外处理可能是根据不同条件做不同的处理，考虑到程序解耦与扩展，于是引入过滤器。通过一层层的过滤器完成过滤操作后才能到desBuf，这个过程就像被加入了一道道处理关卡，经过关卡都会被执行相应操作，最终完成源数据到目的数据的操作。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152177.jpg "web容器,设计,JSP")

  
**输出过滤器**

  
与输入过滤器功能类似，用于在报文输出的时候。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152178.jpg "web容器,设计,JSP")

  
**报文解析器**

  
提供解析http协议各个部分的能力。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152179.jpg "web容器,设计,JSP")

  
**请求生成器**

  
按照面向对象的思想，把每个请求过程中与请求相关的属性及协议字段等抽象成一个Request对象。包括请求行、请求头、请求体三部分信息，在处理过程中需要什么值可直接从request对象中获取。为实现servlet标准提供方便。

  
![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152180.jpg "web容器,设计,JSP")

  
**响应生成器**

  
与请求相对应，需要一个响应对象生成器。包括响应行、响应头、响应体三部分信息，在处理结果相关值可直接设置到response对象中。为实现servlet标准提供方便。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152181.jpg "web容器,设计,JSP")

  
**地址映射器**

  
地址映射器是请求与各个web项目、各个资源的路由器。一个请求的访问根据路径被映射找到响应的资源输出给请求客户端。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152182.jpg "web容器,设计,JSP")

  
**生命周期**

  
为了进一步模块化，整个容器拥有很多组件，这些组件可能在不同的时刻需要做不同的事件，需要一个生命周期统一把所有组件管理起来。例如所有组件的启动、停止、关闭等操作都抽离由生命周期统一管理，就可以方便管理这些组件的生命周期。希望在某某状态事情发生之前之后做点什么？添加一个生命周期监听器即可优雅实现。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152183.jpg "web容器,设计,JSP")



**JMX管理器**

  
系统运行状态的监控及管理，服务器性能、服务器相关参数的收集、JVM负载、web连接数、线程池、[数据库](http://www.2cto.com/database/)连接池、缓存管理、配置文件重新加载等方面。可提供一些远程可视化管理，实时性高。同时也为分布式系统的管理提供了一个解决方案。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152184.jpg "web容器,设计,JSP")

  
**Web载入器**

  
WebLoader用于加载web应用项目，一个web容器可能包含了若干个web应用。为了达到lib及servlet的隔离，对于每个web应用要使用不同的类加载器ClassLoader，且这些类加载器不是父子关系，以此达到class隔离效果，即一个web应用的lib不会被其他web应用使用。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152185.jpg "web容器,设计,JSP")

  
**会话管理器**

  
会话管理器主要对session进行管理，包括：①生成sessionid，一般cookies或url未带jsessionid值则认为不存在会话，需要重新生成sessionid用作会话id。②很多客户端的会话都保存在服务器中，对于超时的会话要定期清理以确保服务器内存不会浪费。③对于一些重要的会话可以持久化到磁盘，需要时可重新加载到内存中使用。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152186.jpg "web容器,设计,JSP")

  
**运行日志**

  
对运行时一些警告、异常、错误进行记录。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152187.jpg "web容器,设计,JSP")



  
**访问日志**

  
访问日志一般会记录客户端的访问相关信息，包括客户端ip、请求时间、请求协议、请求方法、请求字节数、响应码、会话id、处理时间等等。访问日志可以统计访问用户的数量、访问时间分布等规律及个人爱好等等，这些数据可以帮助公司在运营策略上做出抉择。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152188.jpg "web容器,设计,JSP")



  
**安全管理器**

  
Web项目运行在web容器平台上，这就好比将一个应用嵌入到一个平台上面运行，要使嵌入的程序能正常运行，首先平台要能安全正常运行。并且要最大程度做到平台不受嵌入的应用程序影响，两者在一定程度上达到隔离的效果。启动时通过-Djava.security.manager -Djava.security.policy==web.policy指定policy文件，此文件定义了各种权限。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152189.jpg "web容器,设计,JSP")



  
**运行监控&远程管理**

  
提供一个可以实时监控web容器运行状态的平台，并且能进行远程管理。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152190.jpg "web容器,设计,JSP")



  
**集群**

  
集群一般有两种：①负载均衡集群，一般是通过一定的分发算法把访问流量均匀分布到集群里面的各个机器上进行处理。②高可用集群，集群通信把若干机器连接起来，这种集群更偏重的是当集群中某个机器发生故障后能通过自动切换或流量转移等措施来保证整个集群对外的可用性。  
web一般请求都是无状态，可以直接做集群，但涉及session则属于有状态，需要使用集群通信技术进行session拷贝。相关技术包括组播、单播。



![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152191.jpg "web容器,设计,JSP")



  
**Servlet引擎**

  
servlet引擎利用反射把web应用中的servlet及[jsp](http://www.2cto.com/kf/web/jsp/)生成对象并放入servlet对象池中，并根据实际调用相应的方法。web应用将业务逻辑处理都放在dopost或doget方法中，web容器处理请求时就会按照这里定义好的处理逻辑进行处理，处理完响应客户端。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152192.jpg "web容器,设计,JSP")



  
**JSP编译器**

  
按照规范JSP最终都是被编译成servlet执行，所以要按照规范对jsp文件进行编译。JSP编译器其实就是对jsp语法进行翻译，根据jsp语法处理。

![](http://www.2cto.com/uploadfile/Collfiles/20150915/2015091509152193.jpg "web容器,设计,JSP")

  
一个web容器基本包含以上介绍的[组件](http://www.2cto.com/kf/all/zujian/)的功能，根据各个组件模块进行实现即可搭建起一个可以让你的web运行起来的web容器。

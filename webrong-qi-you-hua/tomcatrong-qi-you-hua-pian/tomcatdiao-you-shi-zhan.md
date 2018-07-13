# Tomcat调优实战

* ## 优化内存

/bin/catalina.sh添加JAVA\_OPTS参数

**jdk1.7**

```
JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms512m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=1024M -XX:PermSize=1024m -XX:MaxPermSize=1024m -XX:+DisableExplicitGC"
```

**jdk1.8**

```
JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms512m -Xmx1024m -XX:NewSize=512m -XX:MaxNewSize=1024M -XX:+DisableExplicitGC"
```

> 1.8版本中已经没有PermSize、MaxPermSize

> 参数说明：
>
>   
>
>
>  -Djava.awt.headless：没有设备、键盘或鼠标的模式。有关介绍：
>
> [What is Headless mode in Java](https://link.jianshu.com?t=https://blog.idrsolutions.com/2013/08/what-is-headless-mode-in-java/)
>
>   
>
>
>  -Dfile.encoding： 设置字符集
>
>   
>
>
>  -server：jvm的server工作模式，对应的有client工作模式。使用“java -version”可以查看当前工作模式
>
>   
>
>
>  -Xms512m：初始Heap大小，使用的最小内存
>
>   
>
>
>  -Xmx1024m：Java heap最大值，使用的最大内存
>
>   
>
>
>  -XX:NewSize=512m：表示新生代初始内存的大小，应该小于 -Xms的值
>
>   
>
>
>  -XX:MaxNewSize=1024M：表示新生代可被分配的内存的最大上限，应该小于 -Xmx的值
>
>   
>
>
>  -XX:PermSize=1024m：设定内存的永久保存区域\(注：jdk1.8 was removed\)
>
>
>
>  -XX:MaxPermSize=1024m：设定最大内存的永久保存区域\(注：jdk1.8 was removed



> -XX:+DisableExplicitGC：自动将System.gc\(\)调用转换成一个空操作，即应用中调用System.gc\(\)会变成一个空操作




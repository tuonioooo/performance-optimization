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

  


  



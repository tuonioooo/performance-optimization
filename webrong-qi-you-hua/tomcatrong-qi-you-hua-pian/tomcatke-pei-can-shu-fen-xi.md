# Tomcat可配参数分析

* 配置tomcat调用的虚拟机内存大小

```
（1）直接设置tomcat

　　Linux

　　修改TOMCAT_HOME/bin/catalina.sh 
　　位置cygwin=false前。
　　JAVA_OPTS="-server -Xms256m -Xmx512m -XX:PermSize=64M -XX:MaxPermSize=128m" （仅做参考，具体数值根据自己的电脑内存配置）

　　windows

　　修改TOMCAT_HOME/bin/catalina.bat 
　　第一行加上
　　JAVA_OPTS="-server -Xms256m -Xmx512m -XX:PermSize=64M -XX:MaxPermSize=128m" 

（2）配置环境变量

　　环境变量中设     变量名：JAVA_OPTS     变量值：-Xms512m   -Xmx512m
```




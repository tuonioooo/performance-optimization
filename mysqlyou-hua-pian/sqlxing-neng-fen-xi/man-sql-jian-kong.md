# 慢SQL监控

**前言：** mysql可以记录用户执行的sql：记录到文件、表格 ，mysql可以定义执行多少时间以上得sql属于慢查询，也会根据配置，记录相关信息到文件、表格

**背景说明：**

公司想监控记录每天执行了哪些sql，哪些sql是慢查询，然后去优化sql

**技术说明：**

其实只要搞清楚了mysql怎样记录执行sql的

怎样记录慢查询的即可

**技术细节：**

* 修改my.cnf

```
general_log=1 #开启mysql执行sql的日志
slow_query_log=1 #开启mysql慢sql的日志
log_output=table,File #日志输出会写表，也会写日志文件，为了便于程序去统计，所以最好写表
```




# **数据库层面优化**

数据库优化这个课题较大，可分为四大类：

*       主机性能
*       内存使用性能
*       网络传输性能
*       SQL语句执行性能

当order by 和 group by无法使用索引时，增大max\_length\_for\_sort\_data参数设置和增大sort\_buffer\_size参数的设置


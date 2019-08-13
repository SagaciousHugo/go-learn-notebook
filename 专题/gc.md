# gc


##golang在各个版本中gc算法不同

- v1.1 STW

- v1.3 Mark STW, Sweep 并行

- v1.5 三色标记法

- v1.8 hybrid write barrier

参考资料：
https://studygolang.com/articles/18850?fr=sidebar 图解各种gc算法
https://blog.csdn.net/redenval/article/details/85775314 根据源码分析gc
https://www.cnblogs.com/luckcs/articles/4107647.html 垃圾回收分析例子
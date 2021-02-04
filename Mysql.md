# Mysql

### 大量数据如何存储？如何sharding和路由？如何进行对查找、更新等优化？

https://zhuanlan.zhihu.com/p/54594681

https://juejin.cn/post/6844904132910776328

#### 水平分片

* 几种分片规则：
  * 根据用户id求模，将数据分散到不同的数据库
  * 根据日期进行分片
  * 将摸个字段进行hash或者求模

水平拆分往往还涉及到路由到正确数据库的过程

* 可以在业务代码中注入对应的路由规则
  * 这种方法有侵入性

* 可以通过代理的方式进行转换

分片后的id不能够只使用mysql表的自增id了，需要引入 分布式id 生成技术

同时某些查询操作，如数据统计MAx、Min、Sum以及join等操作，需要像es集群处理多shrad的方式一样，将查询也进行分片、聚合



#### 垂直切分

根据功能将联系更低的列分出来到另外的表里面

如果映射关系需要额外记录的话，可以采用新的关系表的形式

可以更加内聚



### mysql数据监听

可以通过监听binlog的方式进行

* 几种log之间的关系

https://blog.csdn.net/lzhcoder/article/details/88814364

https://www.cnblogs.com/rjzheng/p/9721765.html

* binlog的删除方法

https://www.cnblogs.com/tv151579/p/8289204.html
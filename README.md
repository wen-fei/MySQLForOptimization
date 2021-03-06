references www.imooc.com
# 扛得住的MySQL笔记
## 一、什么撑起了双十一大促
1. web服务器很容易扩展，横向扩展即可，对外提供一样的呢日用
2. 数据库服务器不容易扩展，会破坏完整性和一致性
> 一些指标

1. QPS [每秒钟处理查询量]() 和 TPS[每秒处理的事物量]()
2. 并发量和CPU使用率
3. 磁盘IO
> 因素

1. sql查询速度
2. 服务器硬件
3. 网卡流量
4. 磁盘IO
> 带来的问题

1. 超大QPS和TPS是主要的问题。一般80%的问题都是数据库慢查询引起的
2. 大量的并发和超高的CPU使用率带来的风险
	* 大量的并发： 数据库连接数被占满（连接量和并发量不是同一个概念，并发量是指同一时刻所需要服务器处理的请求的数量，而连接量比并发量大得多），max_connection 默认为100。当连接满，前端就可能出现500错误的提示
	* 超高的CPU使用率：因CPU资源耗尽而出现宕机
3. 磁盘IO
	* 磁盘IO性能突然下降（使用更快的磁盘设备）
	* 其他大量消耗磁盘性能的计划任务（调整计划任务，做好磁盘维护），例如主库的备份调整到副库
4. 网卡流量
	* 减少从服务器的数量
	* 进行分级缓存
	* 避免使用“select * ” 进行查询
	* 分离业务网络和服务器网络
> 大表带来的问题


1. 什么是大表
	* 记录行数巨大，单表超过千万行
	* 表数据文件巨大，表数据文件超过10G
2. 大表对查询的影响
	* 慢查询：很难在一定的时间内过滤出所需要的数据、
3. 大表对DDL操作的影响
	* 建立索引需要很长的时间
		* MYSQL版本<5.5 建立索引会锁表
		* MySQL版本>=5.5 虽然不会锁表但会引起主从延迟
	* 修改表结构需要长时间锁表，
		* 会造成长时间的主从延迟
		* 影响正常的数据操作（会被阻塞）
4. 如何处理大表 
	1. 分库分表把一张大表分成多个小表（不适合大多数公司）
	难点：
		* 分表主键的选择
		* 分表后跨分区数据的查询和统计
	 
	2. 大表的历史数据归档减少对前后端业务的影响
	难点：
		* 归档时间点的选择
		* 如何进行归档操作 
> 大事务带来的问题

1. 事务的特性
	* 事务的原子性(ATOMICITY)：整个事务中的所有操作要么全部提交成功，要么全部回滚失败
	* 事务的一致性（CONSITSTENCY）：数据的完整性没有被破坏
	* 事务的隔离性（ISOLATION）： 一个事务对数据库中数据的修改在未提交完成前对于其他事务是不可见的 
![](C:\Users\user\Desktop\1.png)
	* 事务的持久性：一旦事务提交，则其提交的数据则永久保存到数据库当中
2. 什么是大事务
	* 定义：运行时间比较长，操作的数据比较多的事务
	* 风险：
		* 锁定太多数据，造成大量的阻塞和锁超时
		* 回滚所需时间比较长
		* 执行时间长，容易造成主从延迟
3. 如何处理大事务
 	* 避免一次处理太多的数据
 	* 移除不必要在事务中的SELEFCT操作 
## 二、影响数据库性能的方面
1. 服务器硬件
2. 服务器系统
3. 数据库存储引擎的选择
4. 数据库参数配置
5. 数据库结构设计和SQL语句

前3条差距不大，最主要的是第4和第5条
## 三、数据库索引优化
### 3.1. Btree索引和Hash索引
1. MySQL支持的索引类型
> ### B-tree索引


* B-tree索引的特点：
	* B-tree以B+树的结构存储数据
	* B-tree索引能够加快数据的查询速度
	* B-tree索引更适合进行范围查找
* 在上面情况下可以用到B树索引：
	* 全值匹配的查询，例如order_sn='6767654'查询指定订单号商品
	* 匹配最左前缀的查询，此指联合索引查询下最左边的条件符合情况下会使用此索引，如果是第二个或其他条件则不行
	* 匹配列前缀查询：用于模糊查询的匹配开头order_sn like '676%'
	* 匹配范围值的查询
	* 精确匹配坐前列并范围匹配另外一列
	* 只访问索引的查询：查询只需要访问索引而不需要访问数据行
* Btree索引的使用限制
	* 如果不是按照索引最左列开始查找，则无法使用索引 
	* 使用索引时不能跳过索引中的列
	* Not in 和<>操作无法使用索引
	* 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引
> ### Hash索引

* Hash索引的特点
	* Hash索引是基于Hash表实现的，只有查询条件精确匹配Hash索引中的所有列时(等值查询)，才能够使用到Hash索引
	* 对于Hash索引中的所有列，存储引擎都会为每一行计算一个Hash码，Hash索引中存储的就是Hash码
* Hash索引的限制
	* Hash索引必须进行二次查找（Hash索引中保存的只是索引和Hash码，必须先通过索引找到行，在通过行的记录找到值，但由于基本都是缓存在内存中的，速度非常快，所以大部分情况下对性能影响并不会很大）
	* Hash索引无法用于排序（通过Hash码而不是值，所以无法排序）
	* Hash索引不支持部分索引查找也不支持范围查找（只有全部匹配才能找到对应的Hash码，部分匹配无法找到）
	* Hash索引有可能产生Hash冲突，无法避免。所以选择性很差的列不能进行Hash索引，例如性别只包括男女，存在大量冲突，但如身份证号这样的就可以
> 为什么使用索引

* 索引大大减少了存储引擎需要扫描的数据量
* 索引可以帮助我们进行排序以避免使用临时表
* 索引可以把随机IO变为顺序IO
> 索引是不是越多越好

* 索引会增加写操作的成本
* 太多的索引会增加查询优化器的选择时间

## 四、索引优化策略
### 4.1 索引列上不能使用表达式或函数
![Markdown](http://i2.tiimg.com/602813/ce858349ab0872b4.png)
### 4.2 前缀索引和索引列的选择性
![Markdown](http://i1.ciimg.com/602813/26fd2bd7e2802cfd.png)
### 4.3 联合索引
如何选择索引列的顺序
* 经常会被使用到的列优先
* 状态性的索引不要放左边，即选择性高的列优先（性别选择性差）
* 宽度小的列优先（这样IO会小）
### 4.4 覆盖索引（包含了所有需要查询的字段值得索引）
> 优点

* 可以优化缓存，减少磁盘IO操作（因为索引远比数据小，如果只读取索引就可以找到所需要的数据，就可以...）
* 可以减少随机IO，变随机IO操作为顺序IO操作（B树索引可以把随机IO变为顺序IO，顺序IO处理速度更快）
* 可以避免对Innodb逐渐索引的二次查询（二级索引在叶子结点中保存的是行的主键值，查找到相应的主键后，还要通过主键进行二次查询，才能够获取行的数据）
* 可以避免MyISAM表进行系统调用

> 无法使用覆盖索引的情况

* 存储引擎不支持覆盖索引
* 查询中使用了太多的列
* 使用了双%号的like查询   
## 五、使用索引优化查询
> 使用索引扫描来优化排序

1. 通过排序操作
2. 按照索引顺序扫描数据
> 使用索引扫描来优化排序

1. 索引的列顺序和Order By子句的顺序完全一致
2. 索引中所有列的方向（升序、降序）和Order By子句完全一致
3. Order by中的字段全部在关联表中的第一张表 

> 模拟Hash索引优化查询（使用B树索引模拟）

1. 智能处理键值的全值匹配查找
2. 所使用的Hash函数决定这索引键的大小

> 利用索引优化锁

1. 索引可以减少锁定的行数
2. 索引可以加快处理速度，同时也加快了锁的释放 

> 索引的维护和优化

* 重复索引
![Markdown](http://i4.eiimg.com/602813/37f38160482d9792.png)
* 冗余索引
![Markdown](http://i1.ciimg.com/602813/019a0ae7d90c8037.png)
* 查找和删除重复索引
![Markdown](http://i2.tiimg.com/602813/49ab525b2313ac2e.png)
* 查找未被使用过的索引
TODO: update image url

下面的sql语句可以查找每个数据库上每个表的索引以及索引的使用次数
![Markdown](http://i2.tiimg.com/602813/b480163a1d538083.png)

* 更新索引统计信息以及减少索引碎片

	索引统计信息
		analyze table table_name
		
	维护索引碎片
		optimize table table_name

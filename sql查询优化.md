# SQL查询优化
## 一、 获取有性能问题的SQL
1. 通过用户反馈获取存在性能问题的SQL
2. 通过慢查询日志获取存在性能问题的SQL
3. 实时获取存在性能问题的SQL
## 二、 慢查询日志
> 缺点：

存取日志需要大量磁盘空间
> 对慢查询日志进行控制

![Markdown](http://i2.tiimg.com/602813/f72feca7c3e91ded.png)
默认值为10秒，但太大了，通常改为0.0001秒也就是1毫秒比较合适
<font color='red'>long_queries_not_using_indexes</font> 是否记录未使用索引的SQL

> 慢查询日志中记录的内容

![Markdown](http://i2.tiimg.com/602813/1e83926aea2ed050.png)

1. 执行查询的用户
2. 查询时间
3. 锁的时间
4. 行
5. 检查的函数
6. sql执行所用时间
7. sql语句

> 慢查询日志分析工具

![Markdown](http://i4.eiimg.com/602813/136ea273ed1a5c2d.png)
![Markdown](http://i1.ciimg.com/602813/d28aaf8f551c545d.png)
-t top 按结果top多少条的查询

![Markdown](http://i4.eiimg.com/602813/9bec1514eabb2e25.png)

## 三、实时获取有性能问题的SQL
![Markdown](http://i2.tiimg.com/602813/b988d398b3f86320.png)
SQL语句如下，查询当前服务器中执行时间超过60秒的sql
![Markdown](http://i2.tiimg.com/602813/d392e45e2f59c42f.png)
## 四、 SQL的解析预处理以及生产执行计划
![Markdown](http://i4.eiimg.com/602813/0eaee7dc7b138ebc.png)
> 查询缓存对SQL性能的影响：

![Markdown](http://i1.ciimg.com/602813/d055530c694d2200.png)
对读写比较频繁的查询中建议关闭查询缓存
> SQL的解析预处理以及生成执行计划

![Markdown](http://i2.tiimg.com/602813/13d6b2dcbdc8f3e6.png)
![Markdown](http://i4.eiimg.com/602813/0bb607d51dafa20e.png)

> MySQL优化器可优化的SQL类型

![Markdown](http://i2.tiimg.com/602813/e196e0da119186df.png)
![Markdown](http://i4.eiimg.com/602813/5edf7761e9c7f3bc.png)
![Markdown](http://i4.eiimg.com/602813/53caedc4e068ad34.png)

* 对in()条件进行优化 
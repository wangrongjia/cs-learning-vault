---
dg-publish: false
---
## 慢sql

**如果sql本身没什么问题，并发量导致   考虑使用缓存**

## 查询慢sql日志
```bash
（1）设置开启：SET GLOBAL slow_query_log = 1;　　　#默认未开启，开启会影响性能，mysql重启会失效
（2）查看是否开启：SHOW VARIABLES LIKE '%slow_query_log%';
（3）设置阈值：SET GLOBAL long_query_time=3;
（4）查看阈值：SHOW 【GLOBAL】 VARIABLES LIKE 'long_query_time%';　　#重连或新开一个会话才能看到修改值
（5）通过修改配置文件my.cnf永久生效，在[mysqld]下配置：
　　[mysqld]
　　slow_query_log = 1;　　#开启
　　slow_query_log_file=/var/lib/mysql/atguigu-slow.log　　　#慢日志地址，缺省文件名host_name-slow.log
　　long_query_time=3;　　  #运行时间超过该值的SQL会被记录，默认值>10
　　log_output=FILE
```

如果我们的慢SQL很多，人工分析肯定分析不过来，这时候我们就需要借助一些分析工具，MySQL自带了一个慢查询分析工具mysqldumpslow，以下是常见使用示例

```markup
mysqldumpslow ­s c ­t 10 /var/run/mysqld/mysqld­slow.log # 取出使用最多的10条慢查询
mysqldumpslow ­s t ­t 3 /var/run/mysqld/mysqld­slow.log # 取出查询时间最慢的3条慢查询
mysqldumpslow ­s t ­t 10 ­g “left join” /database/mysql/slow­log #得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow ­s r ­t 10 ­g 'left join' /var/run/mysqld/mysqldslow.log # 按照扫描行数最多的
```

### explain
https://www.cnblogs.com/zjxiang/p/9160564.html

type:
system > const > eq_ref > ref > range > index > all

sql优化： https://tech.meituan.com/2014/06/30/mysql-index.html

### 慢查询优化基本步骤

0.先运行看看是否真的很慢，注意设置SQL_NO_CACHE

1.**where条件单表查，锁定最小返回记录表。这句话的意思是把查询语句的where都应用到表中返回的记录数最小的表开始查起，单表每个字段分别查询，看哪个字段的区分度最高**
举个例子，假设有一个查询需要联结三个表，其中一张表的记录数非常大，而另外两个表的记录数较少。在这种情况下，就应该从记录数最少的两张表开始查询，然后再联结到记录数最多的表。而且在查询每个表时，应该尽可能利用索引，只查询需要的字段，减少数据的读取和匹配次数，以提高查询效率。同时，应该根据实际情况选择区分度最高的字段进行查询，避免无谓的匹配和比较。
表1: user_info，包含用户信息
表3: user_order，包含用户订单信息
查询用户名为Tom的用户最近一个月内的订单记录
应该先查用户表 然后去关联订单表  因为一般情况  订单表总记录数回比用户表大的多

2.explain查看执行计划，是否与1预期一致（从锁定记录较少的表开始查询）

3.order by limit 形式的sql语句让排序的表优先查

4.了解业务方使用场景

5.加索引时参照建索引的几大原则

6.观察结果，不符合预期继续从0分析

#### 建索引的几大原则

1.**最左前缀匹配原则**，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2.=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

3.尽量选择**区分度高的列作为索引**，区分度的公式是count(distinct col)/count(1)， 表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。

4.**索引列不能参与计算**，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’)。

5.**尽量的扩展索引，不要新建索引**。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

### 锁表
**行级锁  还是表级锁**

 **mysql 行锁模拟、锁查看及排除**：https://blog.csdn.net/chuangxin/article/details/86754156
**Lock wait timeout exceeded; try restarting transaction.** 这个报错一般就是锁表
MySQL的 InnoDB 存储引擎支持行级锁，InnoDB 的行锁是通过给索引项加锁实现的。

这句话说明了什么?

说明了一件事：**只有通过索引条件检索数据时，InnoDB 才使用行锁，否则使用表锁。**
 Innodb行锁的优化
-   加索引，让查询走索引
-   学会控制事务
-   隔离级别不要随便设置，根据不同情况不同选择就可以了


#### 锁表的处理
https://xie.infoq.cn/article/469900fb8757d181892384335

```sql
1.  #查看进程id，然后用kill id杀掉进程
2.  show processlist;
3.  SELECT * FROM information_schema.PROCESSLIST；
4.  #查询正在执行的进程
5.  SELECT * FROM information_schema.PROCESSLIST where length(info) >0 ;
6.  #查询是否锁表
7.  show OPEN TABLES where In_use > 0;
8.  #查看被锁住的
9.  SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
10.  #等待锁定
11.  SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
12.  #杀掉锁表进程
13.  kill 进程号
```

### SQL语句常见优化

https://heapdump.cn/article/3892198

只要简单了解过MySQL内部优化机制，就很容易写出高性能的SQL

  
**1.不使用子查询：**

```markup
SELECT * FROM t1 WHERE id (SELECT id FROM t2 WHERE name='hechunyang');
```

在MySQL5.5版本中，内部执行计划器是先查外表再匹配内表，如果外表数据量很大，查询速度会非常慢  
在MySQL5.6中，有对内查询做了优化，优化后SQL如下

```markup
SELECT t1.* FROM t1 JOIN t2 ON t1.id = t2.id;
```

但也仅针对select语句有效，update、delete子查询无效，所以生成环境不建议使用子查询

  
**2.避免函数索引**

```markup
SELECT * FROM t WHERE YEAR(d) >= 2016;
```

即使d字段有索引，也会全盘扫描，应该优化为：

```markup
SELECT * FROM t WHERE d >= '2016-01-01';
```

  
**3.使用IN替换OR**

```markup
SELECT * FROM t WHERE LOC_ID = 10 OR LOC_ID = 20 OR LOC_ID = 30;
```

非聚簇索引走了3次，使用IN之后只走一次：

```markup
SELECT * FROM t WHERE LOC_IN IN (10,20,30);
```

  
**4.LIKE双百分号无法使用到索引**

```markup
SELECT * FROM t WHERE name LIKE '%de%';
```

应优化为右模糊

```markup
SELECT * FROM t WHERE name LIKE 'de%';
```

**5.增加LIMIT M,N 限制读取的条数**

**6.避免数据类型不一致**

```markup
SELECT * FROM t WHERE id = '19';
```

应优化为

```markup
SELECT * FROM t WHERE id = 19;
```

  
**7.分组统计时可以禁止排序**

```markup
SELECT goods_id,count(*) FROM t GROUP BY goods_id;
```

默认情况下MySQL会对所有GROUP BY co1，col2 …的字段进行排序，我们可以对其使用ORDER BY NULL禁止排序，避免排序消耗资源

```markup
SELECT goods_id,count(*) FROM t GROUP BY goods_id ORDER BY NULL;
```

  
**8.去除不必要的ORDER BY语句**

